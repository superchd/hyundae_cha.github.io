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



아래는 Harmony Research Interface 기반으로 만든 **두 개의 모듈**을 정리한 글이다.

- (1) **10초 동안 양팔 모든 조인트 각도를 측정**해서 평균 자세(Initial Posture)를 저장  
- (2) 저장된 자세로 **안전하게 천천히(ramp) 이동**한 뒤 **계속 유지(hold)**

> 구현 영상은 YouTube에 업로드한 뒤, 아래 링크에 붙이면 된다.  
> ✅ Demo Video: **<YOUTUBE_URL_HERE>**

---

## 목표

실험에서 자주 필요한 동작은 다음 2가지다.

1. **Initial posture를 “안전하게 정의”**  
   - 10초 정도 측정해서 평균으로 잡으면 순간값보다 안정적이다.
2. 정의된 initial posture로 **천천히 복귀**하고, 이후 **계속 유지(LOCK/HOLD)**  
   - “토크 ramp-up만”으로는 각도 자체가 천천히 움직인다는 보장이 약할 수 있으니  
     **목표 각도를 시간에 따라 천천히 바꾸는 trajectory(보간) 방식**을 사용한다.

---

## 파일 구성

### 1) `capture_average_posture.cpp`

- 200 Hz로 10초 측정 (초반 warmup 0.5초 버림)
- 결과를 `posture_avg.csv`에 저장 (한 줄)

### 2) `move_and_hold_saved_posture.cpp`

- `posture_avg.csv`에서 목표 posture를 로드
- 현재 자세 → 목표 자세로 **천천히 이동(MOVE phase)**  
  - 목표각 **smoothstep 보간**
  - reference의 변화율을 **qdotMax (rad/s)로 제한**
  - 초반에는 토크도 **ramp_sec 동안 alpha로 점진적 증가**
- 이동이 끝나면 **무한 HOLD phase**로 들어가 목표 자세 유지  
  - Ctrl+C로 종료

> 참고: 마지막 조인트(J7, index 6)는 **torque=0** (제어하지 않음)

---

## CSV 포맷

출력 파일: `posture_avg.csv`

- 한 줄 CSV
- 컬럼 순서:

```
left_q0,left_q1,left_q2,left_q3,left_q4,left_q5,left_q6,
right_q0,right_q1,right_q2,right_q3,right_q4,right_q5,right_q6
```

단위는 **rad** (ResearchInterface의 `position_rad` 그대로)

---

## 코드

### (1) 10초 평균 자세 저장: `capture_average_posture.cpp`

```cpp
/**
 * @file capture_average_posture.cpp
 * @brief 10초 동안 양팔 joint angle 평균 posture를 측정해서 CSV로 저장
 *
 * Output: posture_avg.csv
 * Format (1 line):
 *   left_q0,...,left_q6,right_q0,...,right_q6
 */

#include "research_interface.h"
#include <array>
#include <chrono>
#include <csignal>
#include <fstream>
#include <iostream>
#include <thread>
#include <cmath>

volatile std::sig_atomic_t g_stop = 0;
void handleSigint(int) { g_stop = 1; }

constexpr int NJOINTS = harmony::armJointCount;

void readCurrentPositions(harmony::ResearchInterface& info,
                          std::array<double, NJOINTS>& leftPos,
                          std::array<double, NJOINTS>& rightPos)
{
    auto leftStates  = info.joints().leftArm.getOrderedStates();
    auto rightStates = info.joints().rightArm.getOrderedStates();
    for (int i = 0; i < NJOINTS; ++i) {
        leftPos[i]  = leftStates[i].position_rad;
        rightPos[i] = rightStates[i].position_rad;
    }
}

int main(int, const char**)
{
    std::signal(SIGINT, handleSigint);

    const double fs = 200.0;
    const double dt = 1.0 / fs;
    const int capture_sec = 10;
    const double warmup_sec = 0.5;

    const int total_samples  = static_cast<int>(std::round(capture_sec * fs));
    const int warmup_samples = static_cast<int>(std::round(warmup_sec * fs));

    harmony::ResearchInterface info;
    if (!info.init()) {
        std::cerr << "Failed to initialize Research Interface\n";
        return -1;
    }

    std::array<double, NJOINTS> sumL{}; sumL.fill(0.0);
    std::array<double, NJOINTS> sumR{}; sumR.fill(0.0);
    int used = 0;

    std::cout << "[Capture] 10s average posture @ " << fs << " Hz\n";
    std::cout << "Warmup discarded: " << warmup_sec << " s\n";
    std::cout << "Press Ctrl+C to stop.\n";

    for (int k = 0; k < total_samples && !g_stop; ++k) {
        std::array<double, NJOINTS> qL{}, qR{};
        readCurrentPositions(info, qL, qR);

        if (k >= warmup_samples) {
            for (int j = 0; j < NJOINTS; ++j) {
                sumL[j] += qL[j];
                sumR[j] += qR[j];
            }
            used++;
        }

        std::this_thread::sleep_for(std::chrono::milliseconds(
            static_cast<int>(std::round(dt * 1000.0))
        ));
    }

    if (used <= 0) {
        std::cerr << "[Capture] No valid samples collected.\n";
        return -1;
    }

    std::array<double, NJOINTS> avgL{}, avgR{};
    for (int j = 0; j < NJOINTS; ++j) {
        avgL[j] = sumL[j] / static_cast<double>(used);
        avgR[j] = sumR[j] / static_cast<double>(used);
    }

    // CSV 저장
    const char* outFile = "posture_avg.csv";
    std::ofstream ofs(outFile);
    if (!ofs) {
        std::cerr << "Failed to open output file: " << outFile << "\n";
        return -1;
    }

    // 한 줄로 저장
    for (int j = 0; j < NJOINTS; ++j) {
        ofs << avgL[j] << ",";
    }
    for (int j = 0; j < NJOINTS; ++j) {
        ofs << avgR[j];
        if (j != NJOINTS - 1) ofs << ",";
    }
    ofs << "\n";
    ofs.close();

    std::cout << "[Capture] Saved: " << outFile << "\n";
    std::cout << "Used samples: " << used << "\n";
    return 0;
}
```

---

### (2) 천천히 이동 + 계속 유지: `move_and_hold_saved_posture.cpp`

핵심 안전 포인트:

- **목표 각도 자체를 천천히 변화**:  
  `q_ref(t) = q_start + smoothstep(t/T) * (q_target - q_start)`
- **reference 속도 제한**:  
  `|q_ref[k] - q_ref[k-1]| <= qdotMax * dt`
- **토크도 초반 ramp_sec 동안 점진적 증가**: `u *= alpha`

```cpp
/**
 * @file move_and_hold_saved_posture.cpp
 * @brief posture_avg.csv 목표 posture로 "천천히" 이동 후, 계속 유지(HOLD)하는 토크 PID 제어
 *
 * Input: posture_avg.csv (1 line)
 *   left_q0..left_q6,right_q0..right_q6
 *
 * - MOVE phase: 현재 자세 -> 목표 자세로 천천히 이동 (보간 + 속도 제한 + 토크 ramp)
 * - HOLD phase: 목표 자세를 계속 유지 (Ctrl+C까지)
 * - Last joint (J7, idx 6) torque = 0
 */

#include "research_interface.h"
#include "overrides.h"

#include <array>
#include <chrono>
#include <csignal>
#include <fstream>
#include <iostream>
#include <thread>
#include <cmath>
#include <algorithm>
#include <sstream>
#include <string>

volatile std::sig_atomic_t g_stop = 0;
void handleSigint(int) { g_stop = 1; }

constexpr int NJOINTS = harmony::armJointCount;

static inline double clamp(double x, double lo, double hi) {
    return std::max(lo, std::min(x, hi));
}

static inline double smoothstep(double s) {
    s = clamp(s, 0.0, 1.0);
    return s * s * (3.0 - 2.0 * s);
}

struct AllArmsOverrides {
    harmony::ArmJointsOverride leftOverrides;
    harmony::ArmJointsOverride rightOverrides;
};

AllArmsOverrides torque2override(const std::array<double, NJOINTS>& leftTau,
                                 const std::array<double, NJOINTS>& rightTau)
{
    std::array<harmony::JointOverride, NJOINTS> leftOverrides{};
    std::array<harmony::JointOverride, NJOINTS> rightOverrides{};
    for (int i = 0; i < NJOINTS; ++i) {
        leftOverrides[i]  = {0.0, 0.0, leftTau[i]};
        rightOverrides[i] = {0.0, 0.0, rightTau[i]};
    }
    return {
        harmony::ArmJointsOverride(leftOverrides),
        harmony::ArmJointsOverride(rightOverrides)
    };
}

void readCurrentPositions(harmony::ResearchInterface& info,
                          std::array<double, NJOINTS>& leftPos,
                          std::array<double, NJOINTS>& rightPos)
{
    auto leftStates  = info.joints().leftArm.getOrderedStates();
    auto rightStates = info.joints().rightArm.getOrderedStates();
    for (int i = 0; i < NJOINTS; ++i) {
        leftPos[i]  = leftStates[i].position_rad;
        rightPos[i] = rightStates[i].position_rad;
    }
}

bool loadPostureCSV(const std::string& path,
                    std::array<double, NJOINTS>& leftTarget,
                    std::array<double, NJOINTS>& rightTarget)
{
    std::ifstream ifs(path);
    if (!ifs) return false;

    std::string line;
    if (!std::getline(ifs, line)) return false;

    std::stringstream ss(line);
    std::string token;
    std::array<double, 2 * NJOINTS> vals{};

    int idx = 0;
    while (std::getline(ss, token, ',')) {
        if (idx >= 2 * NJOINTS) break;
        try {
            vals[idx] = std::stod(token);
        } catch (...) {
            return false;
        }
        idx++;
    }
    if (idx != 2 * NJOINTS) return false;

    for (int j = 0; j < NJOINTS; ++j) {
        leftTarget[j]  = vals[j];
        rightTarget[j] = vals[NJOINTS + j];
    }
    return true;
}

int main(int, const char**)
{
    std::signal(SIGINT, handleSigint);

    // ===== timing =====
    const double fs = 200.0;
    const double dt = 1.0 / fs;
    const uint32_t T_ms = static_cast<uint32_t>(std::round(1000.0 / fs));

    // ===== MOVE 설정 =====
    const double move_sec = 10.0;
    const double ramp_sec = 4.0;
    const int total_steps = std::max(1, static_cast<int>(std::round(move_sec * fs)));
    const int ramp_steps  = std::max(1, static_cast<int>(std::round(ramp_sec * fs)));

    // ===== 목표 posture 로드 =====
    std::array<double, NJOINTS> leftTarget{}, rightTarget{};
    const std::string inFile = "posture_avg.csv";
    if (!loadPostureCSV(inFile, leftTarget, rightTarget)) {
        std::cerr << "Failed to load posture file: " << inFile << "\n";
        return -1;
    }

    // ===== init RI / controllers =====
    harmony::ResearchInterface info;
    if (!info.init()) {
        std::cerr << "Failed to initialize Research Interface\n";
        return -1;
    }

    auto left  = info.makeLeftArmController();
    auto right = info.makeRightArmController();
    if (!left->init() || !right->init()) {
        std::cerr << "Failed to initialize Arm Controllers\n";
        return -1;
    }

    // ===== PID gains / torque limits =====
    std::array<double, NJOINTS> Kp{}, Ki{}, Kd{};
    std::array<double, NJOINTS> tauLimit{};

    for (int i = 0; i <= 4; ++i) { // Shoulder
        Kp[i] = 30.0; Ki[i] = 5.0; Kd[i] = 1.0;
        tauLimit[i] = 8.0;
    }
    Kp[5] = 20.0; Ki[5] = 3.0; Kd[5] = 0.8; // Elbow
    tauLimit[5] = 6.0;

    // Last joint (J7, idx 6) off
    Kp[6] = Ki[6] = Kd[6] = 0.0;
    tauLimit[6] = 2.0;

    // ===== reference 속도 제한 (rad/s) =====
    std::array<double, NJOINTS> qdotMax{};
    qdotMax.fill(0.25);
    qdotMax[5] = 0.30;
    qdotMax[6] = 0.0;

    // ===== 시작 자세 =====
    std::array<double, NJOINTS> leftStart{}, rightStart{};
    readCurrentPositions(info, leftStart, rightStart);

    // ===== PID state =====
    std::array<double, NJOINTS> eIntL{};  eIntL.fill(0.0);
    std::array<double, NJOINTS> ePrevL{}; ePrevL.fill(0.0);
    std::array<double, NJOINTS> eIntR{};  eIntR.fill(0.0);
    std::array<double, NJOINTS> ePrevR{}; ePrevR.fill(0.0);

    // ===== ref prev (for qdot limit) =====
    std::array<double, NJOINTS> leftRefPrev  = leftStart;
    std::array<double, NJOINTS> rightRefPrev = rightStart;

    std::cout << "[Move+Hold] Loaded target posture from: " << inFile << "\n";
    std::cout << "[Move] move_sec=" << move_sec << "s, ramp_sec=" << ramp_sec << "s\n";
    std::cout << "[Hold] After move, keep holding until Ctrl+C.\n";
    std::cout << "Last joint (J7, idx 6) torque = 0\n";
    std::cout << "Press Ctrl+C to stop.\n";

    // -------------------------
    // MOVE phase
    // -------------------------
    for (int k = 0; k < total_steps && !g_stop; ++k) {
        double s = (total_steps == 1) ? 1.0
                                      : (static_cast<double>(k) / static_cast<double>(total_steps - 1));
        double s_smooth = smoothstep(s);

        double alpha = clamp(static_cast<double>(k) / static_cast<double>(ramp_steps), 0.0, 1.0);

        std::array<double, NJOINTS> qL{}, qR{};
        readCurrentPositions(info, qL, qR);

        std::array<double, NJOINTS> leftRef{}, rightRef{};
        for (int j = 0; j < NJOINTS; ++j) {
            if (j == NJOINTS - 1) {
                leftRef[j]  = leftRefPrev[j];
                rightRef[j] = rightRefPrev[j];
                continue;
            }

            double qL_des = leftStart[j]  + s_smooth * (leftTarget[j]  - leftStart[j]);
            double qR_des = rightStart[j] + s_smooth * (rightTarget[j] - rightStart[j]);

            double maxStep = qdotMax[j] * dt;
            leftRef[j]  = clamp(qL_des, leftRefPrev[j]  - maxStep, leftRefPrev[j]  + maxStep);
            rightRef[j] = clamp(qR_des, rightRefPrev[j] - maxStep, rightRefPrev[j] + maxStep);
        }
        leftRefPrev  = leftRef;
        rightRefPrev = rightRef;

        std::array<double, NJOINTS> tauL{}; tauL.fill(0.0);
        std::array<double, NJOINTS> tauR{}; tauR.fill(0.0);

        for (int j = 0; j < NJOINTS; ++j) {
            if (j == NJOINTS - 1) {
                tauL[j] = 0.0; tauR[j] = 0.0;
                eIntL[j] = ePrevL[j] = 0.0;
                eIntR[j] = ePrevR[j] = 0.0;
                continue;
            }

            // Left PID -> leftRef
            const double eL = leftRef[j] - qL[j];
            eIntL[j] += eL * dt;

            if (Ki[j] > 1e-9) {
                const double iMax = tauLimit[j] / Ki[j];
                eIntL[j] = clamp(eIntL[j], -iMax, iMax);
            }

            const double dEL = (eL - ePrevL[j]) / dt;

            double uL = (Kp[j] * eL + Ki[j] * eIntL[j] + Kd[j] * dEL);
            uL *= alpha;
            uL = clamp(uL, -tauLimit[j], tauLimit[j]);

            tauL[j] = uL;
            ePrevL[j] = eL;

            // Right PID -> rightRef
            const double eR = rightRef[j] - qR[j];
            eIntR[j] += eR * dt;

            if (Ki[j] > 1e-9) {
                const double iMax = tauLimit[j] / Ki[j];
                eIntR[j] = clamp(eIntR[j], -iMax, iMax);
            }

            const double dER = (eR - ePrevR[j]) / dt;

            double uR = (Kp[j] * eR + Ki[j] * eIntR[j] + Kd[j] * dER);
            uR *= alpha;
            uR = clamp(uR, -tauLimit[j], tauLimit[j]);

            tauR[j] = uR;
            ePrevR[j] = eR;
        }

        auto overrides = torque2override(tauL, tauR);
        left->setJointsOverride(overrides.leftOverrides);
        right->setJointsOverride(overrides.rightOverrides);

        std::this_thread::sleep_for(std::chrono::milliseconds(T_ms));
    }

    if (g_stop) {
        std::cout << "\nStopped during MOVE. Removing overrides.\n";
        left->removeOverride();
        right->removeOverride();
        return 0;
    }

    std::cout << "[Move] Completed. Switching to HOLD (infinite) ...\n";

    // -------------------------
    // HOLD phase (infinite)
    // -------------------------
    while (!g_stop) {
        std::array<double, NJOINTS> qL{}, qR{};
        readCurrentPositions(info, qL, qR);

        std::array<double, NJOINTS> tauL{}; tauL.fill(0.0);
        std::array<double, NJOINTS> tauR{}; tauR.fill(0.0);

        for (int j = 0; j < NJOINTS; ++j) {
            if (j == NJOINTS - 1) {
                tauL[j] = 0.0; tauR[j] = 0.0;
                eIntL[j] = ePrevL[j] = 0.0;
                eIntR[j] = ePrevR[j] = 0.0;
                continue;
            }

            const double eL = leftTarget[j] - qL[j];
            eIntL[j] += eL * dt;
            if (Ki[j] > 1e-9) {
                const double iMax = tauLimit[j] / Ki[j];
                eIntL[j] = clamp(eIntL[j], -iMax, iMax);
            }
            const double dEL = (eL - ePrevL[j]) / dt;

            double uL = (Kp[j] * eL + Ki[j] * eIntL[j] + Kd[j] * dEL);
            uL = clamp(uL, -tauLimit[j], tauLimit[j]);
            tauL[j] = uL;
            ePrevL[j] = eL;

            const double eR = rightTarget[j] - qR[j];
            eIntR[j] += eR * dt;
            if (Ki[j] > 1e-9) {
                const double iMax = tauLimit[j] / Ki[j];
                eIntR[j] = clamp(eIntR[j], -iMax, iMax);
            }
            const double dER = (eR - ePrevR[j]) / dt;

            double uR = (Kp[j] * eR + Ki[j] * eIntR[j] + Kd[j] * dER);
            uR = clamp(uR, -tauLimit[j], tauLimit[j]);
            tauR[j] = uR;
            ePrevR[j] = eR;
        }

        auto overrides = torque2override(tauL, tauR);
        left->setJointsOverride(overrides.leftOverrides);
        right->setJointsOverride(overrides.rightOverrides);

        std::this_thread::sleep_for(std::chrono::milliseconds(T_ms));
    }

    std::cout << "\nCtrl+C detected. Removing overrides and exiting.\n";
    left->removeOverride();
    right->removeOverride();
    return 0;
}
```

---

## 실행 방법 (예시)

1) 평균 자세 캡처

```bash
./capture_average_posture
# => posture_avg.csv 생성
```

2) 천천히 이동 후 계속 유지

```bash
./move_and_hold_saved_posture
# Ctrl+C 누를 때까지 HOLD
```

---

## 파라미터 튜닝 포인트

### `capture_average_posture.cpp`

- `capture_sec = 10`
- `warmup_sec = 0.5`
- `fs = 200`

### `move_and_hold_saved_posture.cpp`

- `move_sec`: 돌아오는 시간(길수록 천천히 움직임)
- `ramp_sec`: 토크 ramp 시간(길수록 초반 더 부드러움)
- `qdotMax`: **가장 중요** (reference 속도 제한)
- `Kp/Ki/Kd`, `tauLimit`: 토크 PID 안정성 및 “락 강도” 결정

---

## 안전 메모

- 처음에는 반드시:
  - `move_sec`를 길게 (예: 12~20초)
  - `qdotMax`를 낮게 (예: 0.15~0.25 rad/s)
  - `tauLimit`를 낮게 시작
- 사람 착용 상태에서는 작은 변화가 크게 느껴질 수 있으니, **천천히** 올리는 게 정답.

---

## Demo Video

- YouTube: **<https://www.youtube.com/shorts/w6soTq1V0k8>**