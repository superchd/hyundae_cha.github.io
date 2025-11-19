---
title: PID Control
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251112
tags: [PID control, Harmony, robotics]
lang: en
math: true
---

## Introduction

The goal is to apply **resistance at the elbow joint (EF joint)** of the Harmony SHR robot.

The initial idea was:

> Continuously read the Harmony EF (elbow flexion) joint angle,  
> slightly push the setpoint in the *opposite direction* of the motion to create a “resistive feeling,”  
> and log everything to CSV + generate plots.

Below is the first implementation of this **virtual impedance (resistive mode)** controller.

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import socket
import time
import threading
import struct
import numpy as np
import pandas as pd
from datetime import datetime
import matplotlib.pyplot as plt

###############################################################################
# Configuration (place right after imports)
###############################################################################
# --- Common for logging/plotting ---
cols = ['time', 'r0_theta', 'r1_theta', 'r2_theta', 'r3_theta', 'r4_theta', 'r5_theta', 'r6_theta', 'r5_set']
dict_list = []
now = datetime.now()

# --- Network ---
TARGET_IP   = "192.168.2.1"  # Harmony PC IP
TARGET_PORT = 12345          # Harmony receive port
LOCAL_PORT  = 12346          # This script receive port

# --- Step-task parameters ---
delta_theta_deg = 25.0
start_setpoint  = np.deg2rad(90.0)

# --- Resistive-mode (virtual impedance) parameters ---
RESISTIVE_MODE  = True     # True: run resistive mode, False: run original step task
CTRL_HZ         = 100      # Control update frequency (50–200 recommended)
THETA_EQ_DEG    = 90.0     # Equilibrium angle (converted to rad later)
KV              = 0.08     # Viscous coefficient (command offset [rad] / [rad/s])
KS              = 0.00     # Spring coefficient (command offset [rad] / [rad])
VEL_LP_ALPHA    = 0.2      # Low-pass coefficient for angular velocity (0–1, larger = faster)
DEADBAND_VEL    = 0.02     # Velocity deadband [rad/s]
MAX_CMD_RATE    = np.deg2rad(30)   # Command rate limit [rad/s]
THETA_MIN       = np.deg2rad(45)   # Safety lower bound
THETA_MAX       = np.deg2rad(135)  # Safety upper bound

# --- Run time ---
RESISTIVE_RUN_SEC = 60     # Duration of resistive mode [s]

# --- Internal state ---
stop_event = threading.Event()
r5_state = {
    "theta": None,
    "theta_prev": None,
    "vel_lp": 0.0,
    "t_recv": None,
    "t_recv_prev": None
}
theta_cmd_current = np.deg2rad(THETA_EQ_DEG)

###############################################################################
# Utility / communication functions
###############################################################################
def command_send(sock, target_ip, target_port, command_setpoint):
    """Send position command in 'EF_{theta}' format + log latest state."""
    global dict_list

    message = f"EF_{command_setpoint:.6f}".encode('utf-8')
    send_time = time.time()
    sock.sendto(message, (target_ip, target_port))

    # Logging: attach last received angles (if any), otherwise NaN
    if dict_list:
        angles = list(map(dict_list[-1].get, cols[1:-1]))  # r0..r6
    else:
        angles = [np.nan] * 7
    values = [send_time] + angles + [command_setpoint]
    dict_data = {c: v for c, v in zip(cols, values)}
    dict_list.append(dict_data)

def command_loop(sock, target_ip, target_port, start, end, delay):
    """±delta step task."""
    for i in range(start, end + 1):
        if i % 2 == 0:
            angle_setpoint = np.deg2rad(90.0 + delta_theta_deg)
        else:
            angle_setpoint = np.deg2rad(90.0 - delta_theta_deg)

        command_send(sock, target_ip, target_port, angle_setpoint)
        print(f"[STEP] loop {i}, cmd={angle_setpoint:.3f} rad")
        time.sleep(delay)

def udp_receiver(sock, start, end, delay, timeout=5):
    """Receive 7 doubles (r0..r6) from Harmony, log them, and update r5 state."""
    global dict_list, r5_state

    sock.settimeout(timeout)
    duration = (end - start) * delay + 15 if not RESISTIVE_MODE else (RESISTIVE_RUN_SEC + 5)
    start_time = time.time()

    while (time.time() - start_time) < duration and not stop_event.is_set():
        try:
            data, addr = sock.recvfrom(1024)
            recv_time = time.time()

            # Need at least 7*8=56 bytes
            if len(data) < 56:
                continue

            try:
                angle_values = list(struct.unpack('7d', data[:56]))
            except struct.error:
                continue

            # Current setpoint (last r5_set in dict_list)
            angle_setpoint = dict_list[-1][cols[-1]] if dict_list else start_setpoint

            # Logging
            values = [recv_time] + angle_values + [angle_setpoint]
            dict_data = {c: v for c, v in zip(cols, values)}
            dict_list.append(dict_data)

            # Update r5 state
            r5_state["theta_prev"]   = r5_state["theta"]
            r5_state["t_recv_prev"]  = r5_state["t_recv"]
            r5_state["theta"]        = angle_values[5]  # r5_theta
            r5_state["t_recv"]       = recv_time

        except socket.timeout:
            print("[RX] timeout, exit receiver")
            break

def resistive_controller(sock, target_ip, target_port):
    """Virtual spring + damper: move setpoint slightly opposite to motion to create resistance."""
    global theta_cmd_current

    period = 1.0 / CTRL_HZ
    theta_eq = np.deg2rad(THETA_EQ_DEG)

    while not stop_event.is_set():
        t0 = time.time()

        th          = r5_state["theta"]
        th_prev     = r5_state["theta_prev"]
        t_recv      = r5_state["t_recv"]
        t_recv_prev = r5_state["t_recv_prev"]

        if th is not None and th_prev is not None and t_recv is not None and t_recv_prev is not None:
            dt = max(1e-3, t_recv - t_recv_prev)  # based on actual receive interval
            vel_raw = (th - th_prev) / dt

            # Deadband + low-pass filter
            if abs(vel_raw) < DEADBAND_VEL:
                vel_raw = 0.0
            r5_state["vel_lp"] = (1 - VEL_LP_ALPHA) * r5_state["vel_lp"] + VEL_LP_ALPHA * vel_raw
            vel = r5_state["vel_lp"]

            # Virtual impedance offset
            offset = -KV * vel - KS * (th - theta_eq)
            theta_target = theta_eq + offset

            # Safety range
            theta_target = float(np.clip(theta_target, THETA_MIN, THETA_MAX))

            # Rate limiting
            max_step = MAX_CMD_RATE * period
            step = float(np.clip(theta_target - theta_cmd_current, -max_step, max_step))
            theta_cmd_current += step

            command_send(sock, target_ip, target_port, theta_cmd_current)

        # Maintain loop period
        dt_sleep = period - (time.time() - t0)
        if dt_sleep > 0:
            time.sleep(dt_sleep)

###############################################################################
# Plotting / post-processing
###############################################################################
def find_edges(setpoints):
    edges_idx = []
    for i in range(1, len(setpoints)):
        if setpoints[i - 1] != setpoints[i]:
            edges_idx.append(i)
    return edges_idx

def plot_data(filename, plot_vlines=True):
    df = pd.read_csv(filename)
    edges_idx = find_edges(df.r5_set)

    plt.plot(df.time, df.r5_theta, label='measured angle')
    # Note: original code multiplied setpoint by -1; if not needed, keep as is
    plt.plot(df.time, df.r5_set, label='setpoint')

    deltat = 0.04
    if plot_vlines and len(edges_idx) > 0:
        for idx, edge_idx in enumerate(edges_idx):
            if idx == 0:
                plt.axvline(df.iloc[edge_idx]['time'] + deltat, color='r', label="0.04 s after command")
                plt.axvline(df.iloc[edge_idx]['time'] + 0.165, color='k', label="0.165 s after command")
            else:
                plt.axvline(df.iloc[edge_idx]['time'] + deltat, color='r')
                plt.axvline(df.iloc[edge_idx]['time'] + 0.165, color='k')

    plt.legend()
    plt.xlabel('Time (s)')
    plt.ylabel('Angle (rad)')
    plt.title(filename)
    plt.show()

###############################################################################
# Main
###############################################################################
if __name__ == "__main__":
    # If you only want to plot an existing CSV, set True
    ONLY_PLOT = False
    file_only_plot = ""  # e.g., 'command-angle-11-11-2025_15-10-00.csv'

    if ONLY_PLOT:
        plot_data(file_only_plot, plot_vlines=False)
        raise SystemExit(0)

    # Prepare socket
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.bind(("0.0.0.0", LOCAL_PORT))

    # Start receiver thread
    start = 0
    end   = 10
    delay = 5

    receiver_thread = threading.Thread(target=udp_receiver, args=(sock, start, end, delay))
    receiver_thread.daemon = True
    receiver_thread.start()

    # Initialize to equilibrium angle
    time.sleep(0.5)
    theta_cmd_current = np.deg2rad(THETA_EQ_DEG)
    command_send(sock, TARGET_IP, TARGET_PORT, theta_cmd_current)
    time.sleep(0.5)

    if RESISTIVE_MODE:
        # Resistive mode: high-rate control thread
        ctrl_thread = threading.Thread(target=resistive_controller, args=(sock, TARGET_IP, TARGET_PORT))
        ctrl_thread.daemon = True
        ctrl_thread.start()

        # Run for the specified duration
        time.sleep(RESISTIVE_RUN_SEC)
        stop_event.set()
        ctrl_thread.join()
    else:
        # Original step task
        command_loop(sock, TARGET_IP, TARGET_PORT, start, end, delay)
        command_send(sock, TARGET_IP, TARGET_PORT, start_setpoint)

    # Wait for receiver thread to finish
    receiver_thread.join()

    # Close socket
    sock.close()
    print("Done")

    # Save CSV + plot
    df_final = pd.DataFrame.from_dict(dict_list)
    date_time = now.strftime("%m-%d-%Y_%H-%M-%S")
    filename = 'command-angle-' + date_time + '.csv'
    df_final.to_csv(filename, index=False)
    print(f"Saved: {filename}")
    plot_data(filename)
