---
title: 하모니 저항
sidebar:
  nav: docs-ko
aside:
  toc: true
key: 20251203
tags: [hamony, control]
lang: ko
math: true
---





### PID 적용 전 코드

시작 자세를 제어 기법 없이 돌리다보니 처음 자세 유지가 안되고 막 로봇 조인트가 흔들리는 상태이다.... -> 수정 반드시 필요

```c++
/**
 * @file hold_start_pose.cpp
 * @brief 현재 자세에서 시작자세로 부드럽게 이동 후, 그 자세에 임피던스 걸고 고정하는 코드
 */

#include "research_interface.h"

#include <array>
#include <chrono>
#include <csignal>
#include <cstdlib>
#include <iostream>
#include <thread>

#define PI 3.141592653
#define DEG_2_RAD (PI / 180.0)

// 시리즈별 최대 임피던스 (SDK 예제 기준)
#define s400Stiffness_Nm_p_rad 50.0 // shoulder 계열 (series 400)
#define s600Stiffness_Nm_p_rad 30.0 // elbow (series 600)
#define s700Stiffness_Nm_p_rad  3.0 // wrist (series 700)

// 로그 없이도 쓰기 위해 동일한 포맷 사용: [time, L7, R7] → 총 1 + 7 + 7 = 15
#define nCols (2 * harmony::armJointCount + 1)

// ===== Ctrl+C 처리 위한 전역 플래그 =====
volatile std::sig_atomic_t g_stop = 0;
void handleSigint(int) { g_stop = 1; }

/******************************************************************************************
 * 보조 함수들
 *****************************************************************************************/

// 조인트 타입에 따라 기본 stiffness 리턴
double jointStiffness(int joint_idx) {
    double stiffness;
    harmony::ArmJoint joint = harmony::ArmJoint(joint_idx);

    switch (joint) {
        case harmony::ArmJoint::elbowFlexion:
            stiffness = s600Stiffness_Nm_p_rad;
            break;
        case harmony::ArmJoint::wristPronation:
            stiffness = s700Stiffness_Nm_p_rad;
            break;
        default:
            stiffness = s400Stiffness_Nm_p_rad;
            break;
    }
    return stiffness;
}

// scaling(0~1)으로 stiffness를 점점 키우고 싶을 때 사용하는 버전
double jointStiffness(int joint_idx, double scaling) {
    double base = jointStiffness(joint_idx);
    return base * scaling;
}

// data 배열(시간 + L7 + R7)을 왼팔/오른팔 override로 변환
struct AllArmsOverrides {
    harmony::ArmJointsOverride leftOverrides;
    harmony::ArmJointsOverride rightOverrides;
};

AllArmsOverrides data2override(const std::array<double, nCols>& data) {
    std::array<harmony::JointOverride, harmony::armJointCount> leftOverrides;
    std::array<harmony::JointOverride, harmony::armJointCount> rightOverrides;

    for (int i = 0; i < harmony::armJointCount; i++) {
        // stiffness는 full 값
        leftOverrides[i]  = { data[i + 1],                         jointStiffness(i), 0.0 };
        rightOverrides[i] = { data[i + harmony::armJointCount + 1], jointStiffness(i), 0.0 };
    }

    return {
        harmony::ArmJointsOverride(leftOverrides),
        harmony::ArmJointsOverride(rightOverrides)
    };
}

// data 배열을 사용하되, stiffness에 scaling(0~1) 적용 (임피던스 ramp-up에 사용)
AllArmsOverrides data2override(const std::array<double, nCols>& data, double scaling) {
    std::array<harmony::JointOverride, harmony::armJointCount> leftOverrides;
    std::array<harmony::JointOverride, harmony::armJointCount> rightOverrides;

    for (int i = 0; i < harmony::armJointCount; i++) {
        leftOverrides[i]  = { data[i + 1],                         jointStiffness(i, scaling), 0.0 };
        rightOverrides[i] = { data[i + harmony::armJointCount + 1], jointStiffness(i, scaling), 0.0 };
    }

    return {
        harmony::ArmJointsOverride(leftOverrides),
        harmony::ArmJointsOverride(rightOverrides)
    };
}

// 현재 Harmony에서 읽은 양팔 조인트 각도를 [time, L7, R7] 포맷으로 반환
std::array<double, nCols> getCurrentArmPositionsAsDataLine(harmony::ResearchInterface* info) {
    auto leftStates  = info->joints().leftArm.getOrderedStates();
    auto rightStates = info->joints().rightArm.getOrderedStates();

    std::array<double, nCols> data{};
    data[0] = 0.0; // timestamp 자리, 여기서는 0으로 고정

    for (int i = 0; i < harmony::armJointCount; i++) {
        data[i + 1] = leftStates[i].position_rad;
    }
    for (int i = 0; i < harmony::armJointCount; i++) {
        data[i + harmony::armJointCount + 1] = rightStates[i].position_rad;
    }

    return data;
}

// start → finish 로 nSteps 동안 선형 보간
std::array<double, nCols> step2targetPosition(const std::array<double, nCols>& start,
                                              const std::array<double, nCols>& finish,
                                              int iter, int nSteps)
{
    std::array<double, nCols> step{};
    step[0] = 0.0;

    for (int i = 1; i < nCols; i++) {
        step[i] = start[i] + (finish[i] - start[i]) * (static_cast<double>(iter) / nSteps);
    }
    return step;
}

/******************************************************************************************
 * Main
 *****************************************************************************************/

int main(int, const char**) {
    // 샘플링 주파수 및 타임스텝
    const double fs   = 200.0;                 // 200 Hz
    const uint32_t T_ms = static_cast<uint32_t>(1000.0 / fs); // 5 ms

    // Ctrl+C 핸들러 등록
    std::signal(SIGINT, handleSigint);

    // Harmony Research Interface 초기화
    harmony::ResearchInterface info;
    if (!info.init()) {
        std::cerr << "Failed to initialize Research Interface" << std::endl;
        return -1;
    }

    auto left  = info.makeLeftArmController();
    auto right = info.makeRightArmController();
    if (!left->init() || !right->init()) {
        std::cerr << "Failed to initialize Arm Controllers" << std::endl;
        return -1;
    }

    // (1) 현재 로봇 자세 읽기
    auto robotStart = getCurrentArmPositionsAsDataLine(&info);

    // (2) 목표 시작자세 구성
    //  - 왼팔은 현재 자세를 유지
    //  - 오른팔만 우리가 원하는 "시작자세"로 이동
    //
    //  아래 desiredRightDeg 값을 원하는 시작자세(도 단위)로 수정해서 사용하면 됨.
    //
    //  예시: modes_parameters.json 의 "home" 과 비슷한 값:
    //  "setpoints": [0, 0, 0, -30, -45, 90, -40] (degree 기준)
    std::array<double, harmony::armJointCount> desiredRightDeg = {
        0.0,   // J1
        0.0,   // J2
        0.0,   // J3
        0.0, // J4
        0.0, // J5
        90.0,  // J6
        0.0  // J7
    };

    // robotStart 기반으로 target 배열을 만들고, 오른팔 부분만 원하는 각도로 교체
    std::array<double, nCols> target = robotStart;
    for (int j = 0; j < harmony::armJointCount; ++j) {
        double rad = desiredRightDeg[j] * DEG_2_RAD;
        // 오른팔은 인덱스 1 + armJointCount 부터 시작
        target[1 + harmony::armJointCount + j] = rad;
    }

    std::cout << "[Step 1] Scaling up impedance at current pose (4 s)\n";
    std::cout.flush();

    // (3) Step 1: 현재 자세에서 4초 동안 임피던스 0 → full 로 서서히 증가
    {
        const int bufferTime_s = 4;
        const int nSteps       = static_cast<int>(bufferTime_s * fs);

        for (int i = 0; i <= nSteps && !g_stop; ++i) {
            // 매 스텝마다 현재 자세를 다시 읽어서 그 자세를 유지하도록 설정
            robotStart = getCurrentArmPositionsAsDataLine(&info);
            double scale = static_cast<double>(i) / nSteps;

            auto overrides = data2override(robotStart, scale);
            left->setJointsOverride(overrides.leftOverrides);
            right->setJointsOverride(overrides.rightOverrides);

            if ((i * T_ms) % 1000 == 0) {
                std::cout << ".";
                std::cout.flush();
            }
            std::this_thread::sleep_for(std::chrono::milliseconds(T_ms));
        }
        std::cout << "\n[Step 1] Done\n";
    }

    if (g_stop) {
        std::cout << "Interrupted during impedance ramp-up. Cleaning up...\n";
        left->removeOverride();
        right->removeOverride();
        return 0;
    }

    std::cout << "[Step 2] Moving to start pose (10 s)\n";
    std::cout.flush();

    // (4) Step 2: 현재자세(robotStart) → target(시작자세) 10초 동안 선형 보간으로 이동
    {
        const int bufferTime_s = 10;
        const int nSteps       = static_cast<int>(bufferTime_s * fs);

        std::array<double, nCols> data{};
        // robotStart는 방금 전 Step 1 마지막의 posture. 다시 한번 읽어서 사용해도 됨.
        robotStart = getCurrentArmPositionsAsDataLine(&info);

        for (int i = 0; i <= nSteps && !g_stop; ++i) {
            data = step2targetPosition(robotStart, target, i, nSteps);

            // 이동 단계에서는 full stiffness 사용
            auto overrides = data2override(data);
            left->setJointsOverride(overrides.leftOverrides);
            right->setJointsOverride(overrides.rightOverrides);

            if ((i * T_ms) % 1000 == 0) {
                std::cout << ".";
                std::cout.flush();
            }
            std::this_thread::sleep_for(std::chrono::milliseconds(T_ms));
        }
        std::cout << "\n[Step 2] Done\n";
    }

    if (g_stop) {
        std::cout << "Interrupted during move-to-start. Cleaning up...\n";
        left->removeOverride();
        right->removeOverride();
        return 0;
    }

    std::cout << "[Step 3] Holding start pose. Press Ctrl+C to stop.\n";
    std::cout.flush();

    // (5) Step 3: 시작자세(target)를 계속 유지 (override로 고정)
    {
        auto overrides = data2override(target); // full stiffness + target angles

        while (!g_stop) {
            left->setJointsOverride(overrides.leftOverrides);
            right->setJointsOverride(overrides.rightOverrides);
            std::this_thread::sleep_for(std::chrono::milliseconds(T_ms));
        }
    }

    std::cout << "\nCtrl+C detected. Removing overrides and exiting.\n";

    // (6) 종료 시 override 해제 (필수)
    left->removeOverride();
    right->removeOverride();

    return 0;
}
```









