---
title: PID 컨트롤
sidebar:
  nav: docs-ko
aside:
  toc: true
key: 20251112
tags: PID 컨트롤
lang: ko
math: true
---





## 1. PID 컨트롤이란?

PID 제어는 **오차(error)** 를 보고 모터 출력을 자동으로 결정해주는 가장 기본적인 제어 방식이다.

- **목표값 (setpoint)**: 우리가 원하는 값 (예: 관절각도 60°)

- **현재값 (measurement)**: 센서가 측정한 실제 값 (예: 50°)

- **오차 (error)**:  
  $$
  e(t) = \text{목표값} - \text{현재값}
  $$

PID 제어기는 이 오차를 보고, 모터에 줄 출력(힘/토크)을 계산한다:

$$
u(t) = K_P e(t) + K_I \int e(t)\,dt + K_D \frac{de(t)}{dt}
$$

여기서  

- \(K_P\) : 비례 (Proportional) 게인  
- \(K_I\) : 적분 (Integral) 게인  
- \(K_D\) : 미분 (Derivative) 게인  

을 뜻한다.

---

## 2. P, I, D 각각 직관적으로 이해하기

### 2.1 P (Proportional) — "오차만큼 당기는 스프링"

가장 기본적인 제어:

$$
u_P(t) = K_P e(t)
$$

- 오차가 클수록 → 출력 크게  
- 오차가 작을수록 → 출력 작게  

예:  
목표 각도 60°, 현재 50° → 오차 10°  
→ \(u_P = K_P \times 10\)

그래서 드. 김이 말한

> 토크 ≈ (각도 변화) × P-gain

은 **P 제어만** 쓴 아주 단순한 형태로 이해할 수 있다.

---

### 2.2 I (Integral) — "오랫동안 쌓이는 불만"

P 제어만 쓰면, 마찰/중력 때문에 **항상 약간 모자란 상태**로 멈출 수 있다.

I 제어는

$$
u_I(t) = K_I \int e(t)\,dt
$$

- 오차가 **오랫동안** 계속되면  
  → 그 오차를 “시간에 대해 누적(적분)”해서  
  → 출력에 점점 더 많이 반영한다.

즉,

- 계속 2° 부족한 상태가 오래 가면  
  → I 항이 점점 커지면서  
  → 결국 목표각도에 더 가깝게 밀어 넣는다.

---

### 2.3 D (Derivative) — "속도를 잡아주는 댐퍼"

D 제어는 오차의 **변화 속도**를 본다.

$$
u_D(t) = K_D \frac{de(t)}{dt}
$$

- 목표를 향해 **너무 빠르게 접근**하면  
  → 브레이크를 걸어 overshoot(휙 지나갔다 되돌아오는 현상)을 줄인다.
- 직관적으로는 **댐퍼(damper)** 나 **쇼크업소버** 같은 역할.

정리하면:

- P = 스프링 (각도 차이를 당기는 힘)  
- D = 댐퍼 (속도에 비례해서 저항)  

그래서 **PD 제어 = 가상의 스프링 + 댐퍼**,  
이는 곧 **임피던스 제어(impedance control)** 와도 연결된다.

---

## 3. PID, PD 제어와 로봇/엑소(Harmony)의 관계

실제 로봇/엑소에서는 대략 이런 형태의 제어가 돌아간다:

```text
목표 관절각도 (θ_des)
          ↓
      [ PID / PD 제어기 ]
          ↓
      모터 토크/전류
          ↓
     실제 관절각도 (θ)
          ↓
      센서 피드백
          └───────(오차 계산으로 다시 위로)
```

Harmony 같은 재활 로봇은 내부적으로:

- **중력보상(gravity compensation)**  
- **PD 기반 임피던스 제어 (position + velocity)**  
- 추가적인 **saturation torque, baseline torque**  

등이 이미 설정되어 있다.

그래서 우리가 바깥에서 보는 토크는:

> **센서 토크 = 로봇이 알아서 넣는 토크  
>
> + 사람 토크  
> + 관성/속도 성분**  

이 섞인 값이다.

이 때문에:

- 단순히 “각도 오차 × P-gain = 사람의 joint torque”라고 보기 어렵고  
- **진짜 생체 토크만 깔끔하게 분리하는 건 현실적으로 거의 불가능에 가깝다**는 게  
  Keya가 강조한 포인트다.

---

## 4. PID 제어 한 줄 요약

- **P (비례)**: 오차 크기에 비례해서 힘  
- **I (적분)**: 오차가 오래 지속될수록 보상 크게  
- **D (미분)**: 변화가 너무 빠를 때 속도를 늦추는 브레이크  

로봇/엑소에서는 이 조합(PID/PD)으로

- 원하는 자세를 유지하고,  
- 부드럽게 움직이고,  
- 안전하게 사람과 상호작용하도록 만든다.

하지만 이런 내부 제어 때문에,  
**측정된 토크 값은 이미 “제어기+중력보상+사람”이 섞인 상태**이고,  
바깥에서 “사람 토크만” 꺼내 쓰려면 많은 가정과 모델링이 필요하다.





## 도입부 

Harmony SHR의 로봇 팔꿈치 조인트에 저항을 주려고한다.

우선, 일반적인 아이디어를 통해 제어하려고 했다.

**하모니 로봇의 EF(elbow) 조인트 각도를 계속 받아서,
팔이 움직일 때 그 반대 방향으로 살짝 setpoint를 밀어줘서 ‘저항감’을 만들어 주고,
그 모든 걸 CSV로 저장 + 플롯까지 하는 코드**.



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
# 설정(필수 import 아래에 둘 것)
###############################################################################
# --- 기록/플롯 공통 ---
cols = ['time', 'r0_theta', 'r1_theta', 'r2_theta', 'r3_theta', 'r4_theta', 'r5_theta', 'r6_theta', 'r5_set']
dict_list = []
now = datetime.now()

# --- 네트워크 ---
TARGET_IP   = "192.168.2.1"  # Harmony PC IP
TARGET_PORT = 12345          # Harmony 수신 포트
LOCAL_PORT  = 12346          # 이 스크립트 수신 포트

# --- 과제(step) 모드 파라미터 ---
delta_theta_deg = 25.0
start_setpoint  = np.deg2rad(90.0)

# --- 저항 모드(가상 임피던스) 파라미터 ---
RESISTIVE_MODE  = True     # True면 저항 모드 실행, False면 기존 step 과제 실행
CTRL_HZ         = 100      # 저항 제어 업데이트 주파수(50~200 권장)
THETA_EQ_DEG    = 90.0     # 평형 각도(라디안으로 변환해 사용)
KV              = 0.08     # 점성 계수 (지령 오프셋[rad]/[rad/s])
KS              = 0.00     # 스프링 계수 (지령 오프셋[rad]/[rad])
VEL_LP_ALPHA    = 0.2      # 각속도 저역통과 계수(0~1, 클수록 빠름)
DEADBAND_VEL    = 0.02     # 미세 속도 무시 데드밴드 [rad/s]
MAX_CMD_RATE    = np.deg2rad(30)   # 지령 변화율 제한 [rad/s]
THETA_MIN       = np.deg2rad(45)   # 안전 하한
THETA_MAX       = np.deg2rad(135)  # 안전 상한

# --- 실행 시간 ---
RESISTIVE_RUN_SEC = 60     # 저항 모드 실행 시간(초)

# --- 내부 상태 ---
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
# 유틸 / 통신 함수
###############################################################################
def command_send(sock, target_ip, target_port, command_setpoint):
    """EF_{theta} 포맷으로 위치 지령 전송 + 최신 상태 로깅"""
    global dict_list

    message = f"EF_{command_setpoint:.6f}".encode('utf-8')
    send_time = time.time()
    sock.sendto(message, (target_ip, target_port))

    # 로그용: 마지막 수신 각도를 붙여 기록(없으면 NaN)
    if dict_list:
        angles = list(map(dict_list[-1].get, cols[1:-1]))  # r0..r6
    else:
        angles = [np.nan] * 7
    values = [send_time] + angles + [command_setpoint]
    dict_data = {c: v for c, v in zip(cols, values)}
    dict_list.append(dict_data)

def command_loop(sock, target_ip, target_port, start, end, delay):
    """±delta step 과제"""
    for i in range(start, end + 1):
        if i % 2 == 0:
            angle_setpoint = np.deg2rad(90.0 + delta_theta_deg)
        else:
            angle_setpoint = np.deg2rad(90.0 - delta_theta_deg)

        command_send(sock, target_ip, target_port, angle_setpoint)
        print(f"[STEP] loop {i}, cmd={angle_setpoint:.3f} rad")
        time.sleep(delay)

def udp_receiver(sock, start, end, delay, timeout=5):
    """Harmony에서 온 7개 double(r0..r6)을 수신하고 로깅 + r5 상태 업데이트"""
    global dict_list, r5_state

    sock.settimeout(timeout)
    duration = (end - start) * delay + 15 if not RESISTIVE_MODE else (RESISTIVE_RUN_SEC + 5)
    start_time = time.time()

    while (time.time() - start_time) < duration and not stop_event.is_set():
        try:
            data, addr = sock.recvfrom(1024)
            recv_time = time.time()

            # 최소 7*8=56 bytes 필요
            if len(data) < 56:
                continue

            try:
                angle_values = list(struct.unpack('7d', data[:56]))
            except struct.error:
                continue

            # 현재 setpoint 추적(마지막 dict_list의 r5_set 사용)
            angle_setpoint = dict_list[-1][cols[-1]] if dict_list else start_setpoint

            # 로깅
            values = [recv_time] + angle_values + [angle_setpoint]
            dict_data = {c: v for c, v in zip(cols, values)}
            dict_list.append(dict_data)

            # r5 상태 업데이트
            r5_state["theta_prev"]   = r5_state["theta"]
            r5_state["t_recv_prev"]  = r5_state["t_recv"]
            r5_state["theta"]        = angle_values[5]  # r5_theta
            r5_state["t_recv"]       = recv_time

        except socket.timeout:
            print("[RX] timeout, exit receiver")
            break

def resistive_controller(sock, target_ip, target_port):
    """가상 점성/스프링으로 '저항감' 부여: 움직임 반대 방향으로 setpoint 미세 조정"""
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
            dt = max(1e-3, t_recv - t_recv_prev)  # 실제 수신 간격 기반
            vel_raw = (th - th_prev) / dt

            # 데드밴드 + 저역통과
            if abs(vel_raw) < DEADBAND_VEL:
                vel_raw = 0.0
            r5_state["vel_lp"] = (1 - VEL_LP_ALPHA) * r5_state["vel_lp"] + VEL_LP_ALPHA * vel_raw
            vel = r5_state["vel_lp"]

            # 가상 임피던스 오프셋
            offset = -KV * vel - KS * (th - theta_eq)
            theta_target = theta_eq + offset

            # 안전 범위
            theta_target = float(np.clip(theta_target, THETA_MIN, THETA_MAX))

            # 지령 변화율 제한
            max_step = MAX_CMD_RATE * period
            step = float(np.clip(theta_target - theta_cmd_current, -max_step, max_step))
            theta_cmd_current += step

            command_send(sock, target_ip, target_port, theta_cmd_current)

        # 주기 유지
        dt_sleep = period - (time.time() - t0)
        if dt_sleep > 0:
            time.sleep(dt_sleep)

###############################################################################
# 플로팅/후처리
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
    # 주의: 기존 코드에서 setpoint에 -1 곱했는데, 그게 꼭 필요한 이유가 없다면 제거
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
# 메인
###############################################################################
if __name__ == "__main__":
    # 과거 CSV만 그림 그리고 싶으면 True
    ONLY_PLOT = False
    file_only_plot = ""  # 예: 'command-angle-11-11-2025_15-10-00.csv'

    if ONLY_PLOT:
        plot_data(file_only_plot, plot_vlines=False)
        raise SystemExit(0)

    # 소켓 준비
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.bind(("0.0.0.0", LOCAL_PORT))

    # 수신 스레드 시작
    start = 0
    end   = 10
    delay = 5

    receiver_thread = threading.Thread(target=udp_receiver, args=(sock, start, end, delay))
    receiver_thread.daemon = True
    receiver_thread.start()

    # 초기 평형각으로 세팅
    time.sleep(0.5)
    theta_cmd_current = np.deg2rad(THETA_EQ_DEG)
    command_send(sock, TARGET_IP, TARGET_PORT, theta_cmd_current)
    time.sleep(0.5)

    if RESISTIVE_MODE:
        # 저항 모드: 고주기 제어 스레드
        ctrl_thread = threading.Thread(target=resistive_controller, args=(sock, TARGET_IP, TARGET_PORT))
        ctrl_thread.daemon = True
        ctrl_thread.start()

        # 설정 시간 동안 실행
        time.sleep(RESISTIVE_RUN_SEC)
        stop_event.set()
        ctrl_thread.join()
    else:
        # 기존 step 과제
        command_loop(sock, TARGET_IP, TARGET_PORT, start, end, delay)
        command_send(sock, TARGET_IP, TARGET_PORT, start_setpoint)

    # 수신 스레드 종료 대기
    receiver_thread.join()

    # 소켓 닫기
    sock.close()
    print("Done")

    # CSV 저장 + 플롯
    df_final = pd.DataFrame.from_dict(dict_list)
    date_time = now.strftime("%m-%d-%Y_%H-%M-%S")
    filename = 'command-angle-' + date_time + '.csv'
    df_final.to_csv(filename, index=False)
    print(f"Saved: {filename}")
    plot_data(filename)

```





## 중요 아이디어

**가상 임피던스(스프링+댐퍼)** 아이디어로 **목표각(setpoint)을 움직임의 반대쪽으로 살짝 밀어** 

사용자가 느끼는 저항감을 만들어 준다. 

즉, 로봇이 바로 잡아당기기보다는 **지령각을 계속 조정**해서 “버티는 느낌”을 주는 방식이다.



## 구체적인 방식

1. 센서(UDP)에서 r5 각도/시간을 계속 받음

- `udp_receiver()`가 `r5_state["theta"]`와 이전 값(`theta_prev`), 수신 시각(`t_recv`, `t_recv_prev`)을 갱신해요.
- 실제 수신 간격 `dt = t_recv - t_recv_prev`로 각속도를 계산할 준비를 해요.

1. 각속도 추정 + 노이즈 억제

- `resistive_controller()`에서
  - 원시 속도 `vel_raw = (th - th_prev) / dt`
  - 속도 데드밴드 적용: `abs(vel_raw) < DEADBAND_VEL`이면 0으로
  - 1차 저역통과: `vel_lp = (1-α)*vel_lp + α*vel_raw` (여기서 α = `VEL_LP_ALPHA`)
     이렇게 해서 튀는 속도값을 억제

1. 가상 임피던스(댐퍼 + 스프링)로 **지령 오프셋** 계산

- 평형각: `theta_eq = deg2rad(THETA_EQ_DEG)` (기본 90°)

- 오프셋:

  ```
  offset = -KV * vel  -  KS * (th - theta_eq)
  ```

  - `-KV * vel`  → **점성(댐핑)**: 사용자가 +방향으로 빠르게 돌리면 setpoint는 -방향으로 살짝 이동해서 **속도에 비례한 저항**을 느낌
  - `-KS * (th - theta_eq)` → **스프링(복원력)**: 평형각에서 벗어나면 setpoint가 반대쪽으로 당겨져 **위치 오차에 비례한 복원력**을 느낌

- 목표 지령각: `theta_target = theta_eq + offset`

1. 안전/부드러움 처리

- **안전 각도 제한**: `THETA_MIN`~`THETA_MAX`로 `theta_target`를 `np.clip`
- **지령 변화율 제한**:
  - 제어주기 `period = 1/CTRL_HZ`
  - 한 주기 최대 변화량: `max_step = MAX_CMD_RATE * period`
  - 실제 스텝: `step = clip(theta_target - theta_cmd_current, ±max_step)`
  - 누적: `theta_cmd_current += step`
     → 갑작스런 지령 점프를 막아 **부드럽고 안전**하게 저항.



## 실제 영상





<iframe
  src="https://drive.google.com/file/d/1J90x3zi9IZ-JZ9ufgY4KaMqVCr6Ab4me/view?usp=drive_link"
  width="100%"
  height="480"
  allow="autoplay"
  frameborder="0"
  allowfullscreen>
</iframe>







## 피드백

osciliate 가 심하다

이 osciliate를 줄이기 위해서 PID 컨트롤 중 I를 제외하고 PD 컨트롤을 사용하려고 한다.





![스크린샷 2025-11-12 172437]({{ "\assets\images\2025-11-12-pid-control\스크린샷 2025-11-12 172437.png" | relative_url }})







## PID 컨트롤 사용



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
# 설정(필수 import 아래에 둘 것)
###############################################################################
# --- 기록/플롯 공통 ---
cols = ['time', 'r0_theta', 'r1_theta', 'r2_theta', 'r3_theta', 'r4_theta', 'r5_theta', 'r6_theta', 'r5_set']
dict_list = []
now = datetime.now()

# --- 네트워크 ---
TARGET_IP   = "192.168.2.1"  # Harmony PC IP
TARGET_PORT = 12345          # Harmony 수신 포트
LOCAL_PORT  = 12346          # 이 스크립트 수신 포트

# --- 과제(step) 모드 파라미터 ---
delta_theta_deg = 25.0
start_setpoint  = np.deg2rad(90.0)

# --- 제어 모드 ---
RESISTIVE_MODE  = True      # True면 제어 스레드 실행(아래 PD), False면 기존 step 과제 실행
CTRL_HZ         = 100       # 제어 주파수(50~200 권장)

# --- PD 제어(가상 임피던스) 파라미터 ---
# 목표: 평형각(theta_eq)에서 벗어나는 움직임에 "부드러운 저항감" 제공
THETA_EQ_DEG    = 70      # 평형각(deg)
KP              = 0.15      # P 이득(스프링 강성에 해당) [rad/rad] -> setpoint 오프셋 배율
KD              = 0.06      # D 이득(점성에 해당) [rad/(rad/s)] -> 속도에 비례한 오프셋
DERIV_LP_ALPHA  = 0.2       # 속도 추정 저역통과 계수(0~1, 클수록 빠름)
DEADBAND_VEL    = 0.02      # 작은 속도 무시 [rad/s]

# --- 지령(출력) 스무딩/제한 ---
CMD_LP_ALPHA    = 0.2       # 지령 저역통과 계수(0~1) — setpoint 자체도 부드럽게
MAX_CMD_RATE    = np.deg2rad(30)   # 지령 변화율 제한 [rad/s]
THETA_MIN       = np.deg2rad(45)   # 안전 하한
THETA_MAX       = np.deg2rad(135)  # 안전 상한

# --- 실행 시간 ---
RESISTIVE_RUN_SEC = 60      # 제어 실행 시간(초)

# --- 내부 상태 ---
stop_event = threading.Event()
r5_state = {
    "theta": None,
    "theta_prev": None,
    "vel_lp": 0.0,
    "t_recv": None,
    "t_recv_prev": None
}
theta_cmd_current = np.deg2rad(THETA_EQ_DEG)   # 실제 보낸 최신 지령
theta_cmd_filt    = np.deg2rad(THETA_EQ_DEG)   # 저역통과를 거친 지령(출력용)

###############################################################################
# 유틸 / 통신 함수
###############################################################################
def command_send(sock, target_ip, target_port, command_setpoint):
    """EF_{theta} 포맷으로 위치 지령 전송 + 최신 상태 로깅"""
    global dict_list

    message = f"EF_{command_setpoint:.6f}".encode('utf-8')
    send_time = time.time()
    sock.sendto(message, (target_ip, target_port))

    # 로그용: 마지막 수신 각도를 붙여 기록(없으면 NaN)
    if dict_list:
        angles = list(map(dict_list[-1].get, cols[1:-1]))  # r0..r6
    else:
        angles = [np.nan] * 7
    values = [send_time] + angles + [command_setpoint]
    dict_data = {c: v for c, v in zip(cols, values)}
    dict_list.append(dict_data)

def command_loop(sock, target_ip, target_port, start, end, delay):
    """±delta step 과제"""
    for i in range(start, end + 1):
        if i % 2 == 0:
            angle_setpoint = np.deg2rad(90.0 + delta_theta_deg)
        else:
            angle_setpoint = np.deg2rad(90.0 - delta_theta_deg)

        command_send(sock, target_ip, target_port, angle_setpoint)
        print(f"[STEP] loop {i}, cmd={angle_setpoint:.3f} rad")
        time.sleep(delay)

def udp_receiver(sock, start, end, delay, timeout=5):
    """Harmony에서 온 7개 double(r0..r6)을 수신하고 로깅 + r5 상태 업데이트"""
    global dict_list, r5_state

    sock.settimeout(timeout)
    duration = (end - start) * delay + 15 if not RESISTIVE_MODE else (RESISTIVE_RUN_SEC + 5)
    start_time = time.time()

    while (time.time() - start_time) < duration and not stop_event.is_set():
        try:
            data, addr = sock.recvfrom(1024)
            recv_time = time.time()

            # 최소 7*8=56 bytes 필요
            if len(data) < 56:
                continue

            try:
                angle_values = list(struct.unpack('7d', data[:56]))
            except struct.error:
                continue

            # 현재 setpoint 추적(마지막 dict_list의 r5_set 사용)
            angle_setpoint = dict_list[-1][cols[-1]] if dict_list else start_setpoint

            # 로깅
            values = [recv_time] + angle_values + [angle_setpoint]
            dict_data = {c: v for c, v in zip(cols, values)}
            dict_list.append(dict_data)

            # r5 상태 업데이트
            r5_state["theta_prev"]   = r5_state["theta"]
            r5_state["t_recv_prev"]  = r5_state["t_recv"]
            r5_state["theta"]        = angle_values[5]  # r5_theta
            r5_state["t_recv"]       = recv_time

        except socket.timeout:
            print("[RX] timeout, exit receiver")
            break

###############################################################################
# PD 컨트롤러 (I 없음)
###############################################################################
def pd_resistive_controller(sock, target_ip, target_port):
    """
    '평형각(theta_eq)' 기준의 PD(스프링+댐퍼)로 setpoint를 미세 조정하여 저항감 제공.
    - D는 측정치 기반(vel)으로 계산 => step 변화 시 D-kick 방지
    - setpoint 자체도 1차 LPF + 변화율 제한 + 안전 클램프
    """
    global theta_cmd_current, theta_cmd_filt

    Ts = 1.0 / CTRL_HZ
    theta_eq = np.deg2rad(THETA_EQ_DEG)

    next_t = time.monotonic()

    while not stop_event.is_set():
        # 주기 정렬 (드리프트 최소화)
        now_t = time.monotonic()
        if now_t < next_t:
            time.sleep(next_t - now_t)
        next_t += Ts
        t0 = time.time()

        th          = r5_state["theta"]
        th_prev     = r5_state["theta_prev"]
        t_recv      = r5_state["t_recv"]
        t_recv_prev = r5_state["t_recv_prev"]

        if th is None or th_prev is None or t_recv is None or t_recv_prev is None:
            continue

        # 실제 수신 간격 기반 속도 추정
        dt_meas = max(1e-3, t_recv - t_recv_prev)
        vel_raw = (th - th_prev) / dt_meas

        # 소속도 무시(미세 떨림 제거) + 1차 LPF
        if abs(vel_raw) < DEADBAND_VEL:
            vel_raw = 0.0
        r5_state["vel_lp"] = (1 - DERIV_LP_ALPHA) * r5_state["vel_lp"] + DERIV_LP_ALPHA * vel_raw
        vel = r5_state["vel_lp"]  # 미분(속도) 추정치

        # PD 제어: setpoint 오프셋 = -Kp*(th - theta_eq) - Kd*vel
        #  - 위치가 평형각보다 +방향이면 setpoint를 -쪽으로 당겨 '복원력' 생성
        #  - +방향으로 빠르게 움직이면 setpoint를 더 -쪽으로 당겨 '점성 저항' 생성
        pos_err   = (th - theta_eq)          # 측정치 기준 오차(참조 = theta_eq)
        offset_pd = -KP * pos_err - KD * vel

        theta_target = theta_eq + offset_pd

        # 안전 각도 제한
        theta_target = float(np.clip(theta_target, THETA_MIN, THETA_MAX))

        # 지령 저역통과(출력 스무딩) -> setpoint 자체를 더 매끈하게
        theta_cmd_filt = (1 - CMD_LP_ALPHA) * theta_cmd_filt + CMD_LP_ALPHA * theta_target

        # 지령 변화율 제한(사용자/기계 안전)
        max_step = MAX_CMD_RATE * Ts
        step = float(np.clip(theta_cmd_filt - theta_cmd_current, -max_step, max_step))
        theta_cmd_current += step

        # 전송
        command_send(sock, target_ip, target_port, theta_cmd_current)

        # (옵션) 남은 시간 슬립으로 주기 유지 (위의 주기 정렬로 대체 가능)
        dt_sleep = Ts - (time.time() - t0)
        if dt_sleep > 0:
            time.sleep(dt_sleep)

###############################################################################
# 플로팅/후처리
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
# 메인
###############################################################################
if __name__ == "__main__":
    # 과거 CSV만 그림 그리고 싶으면 True
    ONLY_PLOT = False
    file_only_plot = ""  # 예: 'command-angle-11-11-2025_15-10-00.csv'

    if ONLY_PLOT:
        plot_data(file_only_plot, plot_vlines=False)
        raise SystemExit(0)

    # 소켓 준비
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.bind(("0.0.0.0", LOCAL_PORT))

    # 수신 스레드 시작
    start = 0
    end   = 10
    delay = 5

    receiver_thread = threading.Thread(target=udp_receiver, args=(sock, start, end, delay))
    receiver_thread.daemon = True
    receiver_thread.start()

    # 초기 평형각으로 세팅
    time.sleep(0.5)
    theta_cmd_current = np.deg2rad(THETA_EQ_DEG)
    theta_cmd_filt    = theta_cmd_current
    command_send(sock, TARGET_IP, TARGET_PORT, theta_cmd_current)
    time.sleep(0.5)

    if RESISTIVE_MODE:
        # PD 저항 모드: 고주기 제어 스레드
        ctrl_thread = threading.Thread(target=pd_resistive_controller, args=(sock, TARGET_IP, TARGET_PORT))
        ctrl_thread.daemon = True
        ctrl_thread.start()

        # 설정 시간 동안 실행
        time.sleep(RESISTIVE_RUN_SEC)
        stop_event.set()
        ctrl_thread.join()
    else:
        # 기존 step 과제
        command_loop(sock, TARGET_IP, TARGET_PORT, start, end, delay)
        command_send(sock, TARGET_IP, TARGET_PORT, start_setpoint)

    # 수신 스레드 종료 대기
    receiver_thread.join()

    # 소켓 닫기
    sock.close()
    print("Done")

    # CSV 저장 + 플롯
    df_final = pd.DataFrame.from_dict(dict_list)
    date_time = now.strftime("%m-%d-%Y_%H-%M-%S")
    filename = 'command-angle-' + date_time + '.csv'
    df_final.to_csv(filename, index=False)
    print(f"Saved: {filename}")
    plot_data(filename)

```





이 pid 컨트롤 코드로는 여전히 osciliate가 심한 상태



<iframe 
  src="https://drive.google.com/file/d/1_1tzgoD6d8xFI90pd0-90V8G5qre-GJg/preview" 
  width="640" 
  height="360" 
  allow="autoplay">
</iframe>



어떻게 개선할 것 인가??







12/9/2025



현재 한 관절 잡아서 kp, ki, kd 값 파라미터 튜닝중 

어떤걸 기준으로 파라미터 튜닝해야하는지 모르겠음



https://uofh-my.sharepoint.com/:x:/r/personal/hcha2_cougarnet_uh_edu/_layouts/15/Doc.aspx?sourcedoc=%7BEA728D6B-0A5D-4F3D-B7E0-7928D1B9F647%7D&file=PID_Tuning_Template_Harmony.xlsx&action=default&mobileredirect=true&wdOrigin=APPHOME-WEB.DIRECT%2CAPPHOME-WEB.BANNER.UPLOAD&wdPreviousSession=ada5ebb7-da55-447e-a1de-21adf03d8f00&wdPreviousSessionSrc=AppHomeWeb&ct=1765320400067

관련 링크임


여기서 파라미터값 수정중이다

