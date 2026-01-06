---
title: "로봇에게 쉐도우 복싱 가르치기 (1): 데이터 수집 파이프라인 구축"
sidebar:
  nav: docs-ko
aside:
  toc: true
key: 20260106
tags: [Robotics, C++, Python, Imitation Learning]
lang: ko
math: true



## 1. 프로젝트 개요: 로봇도 '쉐도우 복싱'이 가능할까?

이 프로젝트의 목표는 **Harmony SHR 로봇**에게 사람의 권투 동작(잽, 훅)을 가르치는 것입니다. 
최신 로봇 제어 트렌드인 **모방 학습(Imitation Learning)**을 적용하기 위해, 우선 실제 사람의 움직임 데이터를 고품질로 수집하는 것이 첫 번째 과제였습니다.

초기에는 로봇 팔을 잡고 동작할 때마다 키보드의 `Start`/`Stop` 버튼을 누르는 방식을 시도했으나, 다음과 같은 문제가 있었습니다.
* **흐름 끊김**: 펀치를 날리고 멈추고 버튼 누르는 과정이 부자연스러움.
* **데이터 파편화**: 수십 번 반복할 때마다 파일이 너무 많이 생성됨.

따라서 전략을 수정하여 **"한 번에 길게 녹화하고(Long-term Logging), 나중에 알고리즘으로 자르는(Auto-Slicing)"** 자동화 파이프라인을 구축했습니다.

---

## 2. 데이터 수집: C++ Continuous Logger

기존 제어 코드에서 복잡한 로직(Ramp-down, PID Control)을 제거하고, 순수하게 센서 데이터만 기록하는 **패시브 로거(Passive Logger)**를 C++로 구현했습니다.

### 주요 기능
* **200Hz 고속 로깅**: 5ms 주기로 양팔 14축(7-DOF x 2)의 위치와 토크 데이터를 기록.
* **인터럽트 처리**: `Ctrl+C` 입력 시 데이터를 안전하게 저장하고 종료.
* **단순화된 워크플로우**: 프로그램 실행 -> "원-투" 10세트 수행 -> 종료.

```cpp
/* record_full_session.cpp 핵심 로직 */

#include "research_interface.h"
#include <fstream>
#include <csignal>
#include <thread>
#include <iostream>

// 시그널 핸들러 (Ctrl+C 안전 종료)
static volatile std::sig_atomic_t g_stop = 0;
void handleSigint(int) { g_stop = 1; }

int main() {
    // ... 로봇 연결 및 초기화 ...
    harmony::ResearchInterface info;
    info.init();
    auto left = info.makeLeftArmController();
    auto right = info.makeRightArmController();
    left->init(); right->init();

    // 안전을 위해 오버라이드 제거 (완전 수동 모드)
    left->removeOverride();
    right->removeOverride();

    std::ofstream ofs("data/full_session_latest.csv");
    // CSV 헤더 작성 (time, qL_0...6, qR_0...6)
    ofs << "time,qL_0,qL_1,qL_2,qL_3,qL_4,qL_5,qL_6,qR_0,qR_1,qR_2,qR_3,qR_4,qR_5,qR_6\n";

    std::cout << "=== [통녹화 모드] 시작: 펀치 후 3초 휴식하세요 ===" << std::endl;
    auto t0 = std::chrono::steady_clock::now();

    while (!g_stop) {
        auto now = std::chrono::steady_clock::now();
        double t = std::chrono::duration<double>(now - t0).count();

        // 200Hz 주기로 양팔 데이터 획득 및 기록
        auto lState = info.joints().leftArm.getOrderedStates();
        auto rState = info.joints().rightArm.getOrderedStates();
        
        ofs << t;
        for(int i=0; i<7; ++i) ofs << "," << lState[i].position_rad;
        for(int i=0; i<7; ++i) ofs << "," << rState[i].position_rad;
        ofs << "\n";

        std::this_thread::sleep_for(std::chrono::milliseconds(5));
    }
    
    std::cout << "저장 완료." << std::endl;
    return 0;
}
3. 데이터 후처리: Python Auto-Cutter
수집된 거대한 CSV 파일(약 15,000 라인)에서 유의미한 펀치 동작만 추출하기 위해 파이썬 스크립트를 작성했습니다.

알고리즘: '정적 구간'을 찾아라
사람이 펀치를 날릴 때는 팔의 속도가 급격히 빨라지고, 쉴 때는 0에 가까워집니다. 이 특성을 이용해 데이터를 자동으로 분할했습니다.

속도 계산: 각 관절의 각속도(Angular Velocity)를 계산.

임계값(Threshold) 적용: 전체 관절 속도 합이 1.5 rad/s 이상이면 "동작 중"으로 판단.

세그멘테이션: 0.5초 이상 정적 구간이 지속되면 동작이 끝난 것으로 간주하고 파일 저장.

Python

# auto_cutter.py 핵심 로직
import pandas as pd
import numpy as np

# 1. 속도(Velocity) 계산
df = pd.read_csv('data/full_session_latest.csv')
dt = df['time'].diff().mean()
# 차분(diff)을 이용해 속도 추정
velocities = df.iloc[:, 1:].diff().abs().sum(axis=1) / dt

# 2. 움직임 감지 (Thresholding)
is_moving = velocities > 1.5 

# 3. 구간 자르기 및 저장 (Simplified)
combo_id = 1
start_idx = -1
min_pause = int(0.5 / dt)
gap = 0

for i in range(len(is_moving)):
    if is_moving[i]: 
        if start_idx == -1: start_idx = i
        gap = 0
    else:
        if start_idx != -1:
            gap += 1
            if gap > min_pause:
                # 파일 저장
                sub_df = df.iloc[start_idx : i - gap]
                if len(sub_df) > 20: # 노이즈 제거
                    sub_df.to_csv(f"data/cut_results/combo_{combo_id}.csv", index=False)
                    combo_id += 1
                start_idx = -1
4. 데이터 분석 및 품질 검증
수집된 데이터를 분석하여 학습에 적합한지 확인했습니다.

4.1 데이터 스펙
샘플링 레이트: 약 196.6 Hz (목표 200Hz 충족)

최대 속도: 11.73 rad/s (왼팔 기준)

일반적인 로봇 움직임보다 훨씬 빠른, 실제 펀치의 역동성이 그대로 담겼습니다.

4.2 시각화 (Visualization)
아래 그래프는 전체 세션의 관절 각도와 속도 변화를 보여줍니다. 파란색/빨간색 선(Speed)이 치솟는 구간이 바로 "원-투" 펀치를 날린 순간이며, 그 사이의 평평한 구간이 휴식 구간입니다.

(Python Matplotlib으로 생성한 분석 결과)

그래프를 통해 "동작(Active)"과 "휴식(Idle)"이 명확하게 구분됨을 확인할 수 있었으며, 이는 자동 자르기 알고리즘이 완벽하게 작동할 수 있음을 의미합니다.

5. 결론 및 향후 계획
이번 작업을 통해 **"데이터 수집 → 자동 전처리 → 품질 검증"**으로 이어지는 파이프라인을 완성했습니다. 이제 로봇은 사람의 펀치 동작을 학습할 준비가 되었습니다.

Next Steps:

분할된 데이터(combo_*.csv)를 로봇 실행 포맷(exercise.json)으로 변환.

데이터 스무딩(Smoothing)을 통해 미세한 떨림 보정.

실제 하드웨어에서 쉐도우 복싱 재현 테스트.

Note: 이 프로젝트는 Harmony SHR 재활 로봇 플랫폼을 기반으로 진행됩니다.