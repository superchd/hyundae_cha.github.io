---
title: 하모니 제어
sidebar:
  nav: docs-ko
aside:
  toc: true
key: 20251203
tags: [hamony, control]
lang: ko
math: true

---



간단히 말하면,

- **command_and_report.cpp = 로봇 쪽 게이트웨이(펌웨어/서버)**
- **elbow_pid_control.py = 내 노트북 쪽 브레인(클라이언트/컨트롤러 + 로거)**

둘이 **UDP 소켓으로 서로 계속 말 주고받으면서** 돌아가는 구조야.

------

## 1. 각 코드가 하는 역할

### 1) command_and_report.cpp (Harmony PC 안에서 동작) 

이 코드는 Harmony 연구용 인터페이스를 초기화하고,

- **오른팔 7개 조인트 각도(라디안)를 읽어서**
- **200 Hz로 `double[7]` 배열을 UDP로 계속 쏴 줘**
  - `sendData()` 스레드에서 `rightStates[i].position_rad`를 `data[7]`에 넣고
  - `192.168.2.2:12346`(하드코딩된 TARGET_IP/TARGET_PORT)로 송신 

또 한편으로는

- **로봇이 UDP 서버가 되어 포트 12345에서 문자열을 기다리고 있다가**,
- `"EF_0.785398"` 이런 식의 메시지를 받으면
  - `_` 앞: `"EF"` → prefix (elbow flexion 의미)
  - `_` 뒤: 숫자 부분을 `double value`로 파싱 
- 그 값을 이용해서 **오른팔 조인트 override 설정**을 만든 다음
  - 모든 조인트는 `{0, stiffness, 0}`
  - **i == 5 (elbow)**만 `{ -value, stiffness, 0 }`로 넣어서
  - `right->setJointsOverride(rightOverrides)` 호출해서 실제 Harmony 엘보우에 명령을 준다 

즉,
 👉 **“외부에서 오는 문자열 → elbow 명령으로 변환해서 로봇에 적용”**
 👉 **“현재 조인트 각도 → 외부 PC로 200 Hz 피드백 송신”**
 을 동시에 하는 **게이트웨이 + 간단한 로봇 컨트롤러**라고 보면 돼.

------

### 2) elbow_pid_control.py (내 컴퓨터에서 동작) 

이 파이썬 코드는 **외부 PC 쪽에서 로봇을 제어하고, 데이터도 로깅하는 상위 컨트롤러** 역할을 해.

1. **네트워크 세팅**

- `TARGET_IP = "192.168.2.1"`, `TARGET_PORT = 12345` → **이쪽으로 명령 전송**
- `LOCAL_PORT = 12346`에 `sock.bind(("0.0.0.0", LOCAL_PORT))` → **로봇이 보내는 7개 조인트 각도를 여기에 수신** 

즉,

- **보낼 때:** 파이썬 → `192.168.2.1:12345` → C++가 듣고 elbow override
- **받을 때:** C++ → `192.168.2.2:12346` → 파이썬이 각도 수신

1. **명령 포맷**

- `command_send()` 에서

  ```
  message = f"EF_{command_setpoint:.6f}".encode('utf-8')
  sock.sendto(message, (target_ip, target_port))
  ```

  이렇게 **`"EF_각도값"` 형식 문자열**을 C++ 쪽으로 보내고,

- 동시에 **마지막으로 받은 조인트 각도들과 함께 CSV 로그용 dict_list에 쌓음** 

1. **로봇 각도 수신 + 상태 업데이트**

- `udp_receiver()` 스레드에서
  - 로봇이 보내는 7개 `double`(r0~r6)을 `struct.unpack('7d', data[:56])`으로 풀어서
  - `time, r0_theta, …, r6_theta, r5_set` 형태로 dict_list에 계속 기록
  - `r5_state["theta"], "theta_prev", "t_recv", "t_recv_prev"`를 업데이트해서 **엘보우 각도/속도 추정에 사용** 

1. **PD 기반 저항 모드 (가상 임피던스)**

- `pd_resistive_controller()`에서 **엘보우 각도(r5)를 계속 읽어오고**,
  - 평형각 `THETA_EQ_DEG`(예: 70도) 기준으로
  - P: (현재 각도 - 평형각)
  - D: 속도(θ 변화율)
     를 써서 `theta_target`을 만들고 
- 이 `theta_target`을
  - LPF로 한 번 부드럽게 만들고(`CMD_LP_ALPHA`)
  - 변화율 제한(`MAX_CMD_RATE`) 걸고
  - 안전 각도 범위(`THETA_MIN`, `THETA_MAX`) 안으로 자른 다음
  - 최종 `theta_cmd_current`을 다시 `"EF_{값}"`으로 C++에게 전송 

즉,
 👉 **“로봇에서 받은 실제 elbow 각도 → PD 제어로 새 setpoint 계산 → 다시 로봇에 전송”**
 👉 동시에 **로그 + 플롯 + CSV 저장**까지 해주는 상위 레벨 제어 코드야.

------

## 2. 둘 사이의 관계를 한 줄로 정리하면

- **C++ (command_and_report.cpp)**
  - 로봇 안에서 돌아가는 **하드웨어 가까운 쪽 코드**
  - **UDP 서버**로서 외부 명령(`EF_값`)을 받아 elbow override로 바꾸고,
  - **UDP 클라이언트**로서 자기 조인트 각도를 외부로 스트리밍
- **Python (elbow_pid_control.py)**
  - 내 노트북에서 돌아가는 **상위 제어 + 데이터 로깅 코드**
  - **로봇 각도 피드백을 받아**(r5_theta)
  - PD/임피던스 로직으로 **새 elbow 목표각을 계산해서**
  - `"EF_값"` 형태로 **C++에게 보내는 브레인**

비유하자면,

> **command_and_report.cpp = “로봇 몸 안에 있는 통역사 + 팔 근육에 직접 명령하는 신경”**
>  **elbow_pid_control.py = “바깥에서 상황 보면서 계속 명령 내리는 뇌”**

그래서 둘은 **UDP 통신으로 서로 연결된 “몸 ↔ 뇌” 관계**라고 이해하면 제일 편해.









### Command and report.cpp



```c++
#include "research_interface.h"
#include <array>
#include <iostream>
#include <string>
#include <cstring>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <thread>

#define PORT 12345  // The port the server will listen on
#define TARGET_IP "192.168.2.2"  // The hardcoded target IP
#define TARGET_PORT 12346    // The hardcoded target port

#define s400Stiffness_Nm_p_rad 25.0 // desired joint stiffness for series 400 [shoulder] motors(max is 50)
#define s600Stiffness_Nm_p_rad 10.0 // desired joint stiffness for series 600 [elbow] motors (max is 30)
#define s700Stiffness_Nm_p_rad  1.0 // desired joint stiffness for series 700 [wrist] motors(max is 3)

// ★ 조인트 인덱스 정의 (필요하면 여기 숫자만 바꿔주면 됨)
constexpr int ELBOW_FLEX_IDX     = 5; // elbow flexion (이미 코드/주석에서 5번이라고 되어 있음)
constexpr int SHOULDER_ABD_IDX   = 1; // shoulder abduction (실제 인덱스랑 다르면 이 숫자만 바꿔줘)

// prefix -> 조인트 인덱스 매핑 함수
int jointIndexFromPrefix(const std::string& prefix)
{
    if (prefix == "EF") return ELBOW_FLEX_IDX;   // Elbow Flexion
    if (prefix == "SA") return SHOULDER_ABD_IDX; // Shoulder Abduction

    // 필요하면 나중에 더 추가
    // if (prefix == "SF") return 0; // Shoulder Flexion (예시)

    return -1; // 알 수 없는 prefix
}

/**
 * @brief Thread function to stop while loop on usr command [Ctrl-D]
 *
 * @param spin is the while loop flag from main
 */
void loopSpin(bool* spin) {
    std::string str;

    while (*spin) {
        *spin = bool(std::cin >> str);
    }
}

/**
 * @brief returns the desired joint stiffness based on the actuator type
 * Joints <5 are all series 400, joint 5 is the elbow series 500, and
 * joint 6 is the wrist series 700 joint.
 * @param joint_idx current index for joint
 * @param scaling is used to indirectly control the stiffnesses
 * @return double the desired joint stiffness
 */
double jointStiffness(int joint_idx, double scaling) {
    double stiffness;
    harmony::ArmJoint joint = harmony::ArmJoint(joint_idx);

    switch (joint) {
        case harmony::ArmJoint::elbowFlexion:
            stiffness = s600Stiffness_Nm_p_rad * scaling;
            break;
        case harmony::ArmJoint::wristPronation:
            stiffness = s700Stiffness_Nm_p_rad * scaling;
            break;
        default:
            stiffness = s400Stiffness_Nm_p_rad;
            break;
    }

    return stiffness;
}

void sendData(bool* spin, const int sockfd, const struct sockaddr_in target_addr,
              const harmony::ResearchInterface * info, double T_s)
{
    while(*spin){

        auto rightStates = info->joints().rightArm.getOrderedStates();

        double data[7] = {0};
        for(int i = 0; i < rightStates.size(); i++){
            data[i] = rightStates[i].position_rad;
        }

        // Send the message to the hardcoded address
        if (sendto(sockfd, data, sizeof(data), 0, 
                   (const struct sockaddr *)&target_addr, sizeof(target_addr)) < 0) {
            // std::cerr << "Failed to send message to hardcoded address" << std::endl;
        }

        std::this_thread::sleep_for(std::chrono::milliseconds(uint(1000 * T_s)));
    }
}

int main() {

    double fs = 200; // Frame rate in hz

    // init research interface
    harmony::ResearchInterface info;
    if (!info.init()) {
        std::cerr << "ERROR: Research Interface failed to initialize!" << std::endl;
        return -1;
    }

    auto left = info.makeLeftArmController(); // left arm controller
    auto right = info.makeRightArmController(); // right arm controller
    if (!left->init() || !right->init()) {
        std::cerr << "Failed to initialize Arm Controllers" << std::endl;
        return -1;
    }

    // setup loop vars
    double T_s = 1 / fs; // Time_step in seconds
    char logIndicator[4]{'-', '\\', '|', '/'}; // indicator for recording... adds some flair
    int i = 0; // loop iteration count
    bool spin = true; // while loop flag

    int sockfd;
    struct sockaddr_in server_addr, client_addr, target_addr;
    char buffer[1024];
    socklen_t client_addr_len = sizeof(client_addr);

    // Create UDP socket
    if ((sockfd = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
        std::cerr << "Failed to create socket" << std::endl;
        return -1;
    }

    // Zero out the server address structure
    memset(&server_addr, 0, sizeof(server_addr));
    memset(&client_addr, 0, sizeof(client_addr));

    // Set up the server address structure
    server_addr.sin_family = AF_INET; // IPv4
    server_addr.sin_addr.s_addr = INADDR_ANY; // Listen on all available interfaces
    server_addr.sin_port = htons(PORT); // Port number

    // Bind the socket to the server address
    if (bind(sockfd, (const struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        std::cerr << "Bind failed" << std::endl;
        close(sockfd);
        return -1;
    }

    std::cout << "UDP server listening on port " << PORT << std::endl;
    std::cout.flush();

    // Set up the hardcoded target address
    memset(&target_addr, 0, sizeof(target_addr));
    target_addr.sin_family = AF_INET;
    target_addr.sin_port = htons(TARGET_PORT); // Hardcoded target port

    if (inet_pton(AF_INET, TARGET_IP, &target_addr.sin_addr) <= 0) {
        std::cerr << "Invalid address/ Address not supported" << std::endl;
        return -1;
    }

    // run until [Ctrl-D] is pressed
    std::thread spinThread(loopSpin, &spin); // thread to stop the while loop
    std::thread sendThread(sendData, &spin, sockfd, target_addr, &info, T_s);

    std::cout << "Enter [Ctrl-D] to stop recording.\n";
    std::cout.flush();

    // ★ 각 조인트별 현재 offset [rad] 저장용 (항상 유지)
    std::array<double, harmony::armJointCount> jointOffsets{};
    jointOffsets.fill(0.0);

    while (spin) {
        // Receive message from the client
        int len = recvfrom(sockfd, buffer, sizeof(buffer) - 1, 0, 
                           (struct sockaddr *)&client_addr, &client_addr_len);
        if (len < 0) {
            std::cerr << "Failed to receive message" << std::endl;
            continue;
        }
        buffer[len] = '\0';  // Null-terminate the received data

        std::string received_message(buffer);
        std::cout << "Received: " << received_message << std::endl;
        std::cout.flush();

        // "PREFIX_value" 형식 파싱
        size_t underscore_pos = received_message.find('_');
        if (underscore_pos == std::string::npos) {
            std::cerr << "Invalid message format: " << received_message << std::endl;
            continue;
        }

        std::string prefix = received_message.substr(0, underscore_pos);
        double value = 0.0;
        try {
            value = std::stod(received_message.substr(underscore_pos + 1));
        } catch (const std::exception& e) {
            std::cerr << "Failed to parse value from message: " << received_message
                      << " (" << e.what() << ")" << std::endl;
            continue;
        }

        // prefix -> 조인트 인덱스 찾기
        int targetIdx = jointIndexFromPrefix(prefix);
        if (targetIdx < 0 || targetIdx >= harmony::armJointCount) {
            std::cerr << "Unknown joint prefix: " << prefix << std::endl;
            continue;
        }

        // ★ 이 메시지에서 해당 조인트의 offset만 업데이트 (나머지는 유지)
        jointOffsets[targetIdx] = value;

        // Set overrides for ALL joints using jointOffsets
        std::array<harmony::JointOverride, harmony::armJointCount> rightOverridesArr;
        for (int j = 0; j < harmony::armJointCount; j++) {
            double offset = -jointOffsets[j];  // Python에서 보낸 값을 그대로 쓰고, 부호는 여기서 통일
            rightOverridesArr[j] = {offset, jointStiffness(j, 1), 0.0};
        }

        harmony::ArmJointsOverride rightOverrides = harmony::ArmJointsOverride(rightOverridesArr);
        right->setJointsOverride(rightOverrides);
    }

    right->removeOverride();

    spinThread.join();
    sendThread.join();

    // Close the socket
    close(sockfd);
    return 0;
}

```





command_and_report.cpp 249번쨰 줄



12/4 문제는 하나의 조인트씩 제어하려고 하는데 

만약 3번 조인트를 제어하려고 하면 , 1 , 2 번 조인트는 free 상태있어야 하는데 free 상태가 아니라 계속 제어가 되고 있음

그렇기 때문에 구조적으로 코드를 바꾸어야지

딱 하나씩만 제어를 하고 , 또 하나씩만 제어를 해야 전체 제어에 용이할듯

지금 내가 원하는건 

원하는 위치로 로봇의 조인트 앵글 다 맞추었을때(저항 주기시작) -> 여기까지는 오케이야 

그 다음에 로봇의 조인트에 자유를 줘야하는데 즉 저항을 안주어야하는데 계속 과거의 명령어를 들고 있는 느낌이라서 이 부분을 수정해야함 ... 어떻게 하면 될까....

이걸 좀 고쳐야함 

 

```c++
        for (int j = 0; j < harmony::armJointCount; j++) {
            double offset  = 0.0;
            double stiff   = 0.0;
            double damping = 0.0;

            // 이 조인트에 대해 Python에서 실제로 offset 명령이 들어온 경우에만
            // stiffness를 켜고 offset을 적용
            if (std::fabs(jointOffsets[j]) > EPS) {
                offset = -jointOffsets[j];       // 부호 통일
                stiff  = jointStiffness(j, 1.0); // 해당 조인트 stiffness 적용
            } else {
                // 한 번도 명령 안 받은 조인트 or 거의 0인 경우:
                // offset=0, stiffness=0 → 거의 제어 안 받는 느낌  -> 여기가 문제인것 같음 , offset = 0, stiffness = 0으로 만드니까 계속 저항이 걸려있는것 같은데
                // 내가 원하는 이상적인거는 free 한 힘이 들어야함 
                offset = 0.0;
                stiff  = 0.0;
            }
```









## 하모니 로봇 내 파일 구조 

## 1. 전체 큰 그림

이 레포는 **Harmony 로봇용 리서치 라이브러리** 하나를 만들고,
 그 위에 여러 **툴(binary)** 을 얹어 놓은 구조야. 

계층으로 보면 대충 이렇게 생각하면 됨:

1. **Application / Tools 계층**
   - `tools/` 안의 `harmony_logger`, `harmony_exerciser`, `sendPositionsUDP`, `setController` 같은 실행파일들
2. **Research Library 계층**
   - `libharmony_research.a` (빌드 결과)
   - 헤더: `include/arm_controller.h`, `research_interface.h`, `joint_states.h`, `shared_memory*.h` 등
   - 구현: `src/arm_controller.cpp`, `src/research_interface.cpp`
3. **Third-party & Build System**
   - `subprojects/eigen-3.4.0` : 수학/선형대수 라이브러리 Eigen 전체 소스
   - `subprojects/googletest-release-1.11.0` : 단위 테스트용 GoogleTest
   - `meson.build`, `builddir/` : Meson + Ninja 빌드 시스템 관련 파일들

실제 **로봇 내부 “제어기/펌웨어”**는 여기 없는, 다른 프로세스/장비에 있고,
 이 레포는 그 로봇과 **shared memory + C++ API** 로 통신하는 “연구자용 인터페이스”라고 보면 될 것 같아.

------

## 2. 핵심 라이브러리 구조 (로봇 내부 시스템 관점)

### (1) `include/` – 로봇 데이터 구조 & API 정의

- `arm_controller.h`
   → Harmony 팔(어깨–팔꿈치) 제어를 위한 상위 레벨 컨트롤러 클래스.
   → “이 조인트를 이런 모드/값으로 움직여라” 같은 명령을 내리는 인터페이스일 가능성이 높음.
- `research_interface.h`
   → **로봇 메인 컨트롤 프로세스와 통신하는 핵심 인터페이스**.
   → shared memory attach, 데이터 읽고 쓰기, 초기화/종료 같은 걸 담당.
- `joint_states.h`
   → 각 조인트의 **각도, 속도, 토크, 상태 플래그** 등을 담는 struct 정의일 가능성이 큼
   (ROS의 sensor_msgs/JointState 같은 느낌).
- `pose.h`
   → 엔드이펙터나 세그먼트의 **자세(orientation + position)** 표현을 위한 구조체/클래스.
- `shared_memory.h`, `shared_memory_manager.h`
   → OS-level shared memory를 열고 닫고, 구조체를 매핑하는 부분.
   → 로봇 내부 다른 프로세스(실시간 제어 loop)와 데이터를 주고받는 통로라고 보면 됨.
- `sizes.h`
   → 조인트 개수, 버퍼 크기, shared memory 안에서 struct 크기/오프셋 같은 상수 정의.
- `overrides.h`
   → 빌드 플랫폼에 따라 바뀌는 설정(예: POSIX vs Windows)이나 매크로 override용.

이 헤더들이 **“로봇 내부 상태를 외부에서 어떻게 보냐 / 어떻게 명령을 넣냐”**를 정의하는 최상단 API라고 보면 돼. 

------

### (2) `src/` – 실제 구현

- `src/arm_controller.cpp`
   → `arm_controller.h`에 선언된 함수들의 구현.
   → 위치 제어, 속도 제어, 임피던스/토크 모드 전환 같은 로직이 들어 있을 가능성이 큼.
   → 내부적으로 `research_interface`와 `shared_memory`를 써서 실제 로봇에 명령 전달.
- `src/research_interface.cpp`
   → shared memory attach, 데이터 구조 매핑,
   “현재 joint_states를 읽어와라”, “command 버퍼에 이 값 써라” 같은 실제 I/O 코드.

빌드하면 이 둘이 `builddir/src/libharmony_research.a` static 라이브러리로 묶임. 

즉,

- **로봇 메인 제어 시스템**: (다른 프로세스/기기, 여기엔 없음)
- **이 라이브러리**: 그 시스템이 내놓은 shared memory를 C++에서 쉽게 쓰도록 wrapping한 계층

이라고 생각하면 로봇 내부 시스템 구조가 좀 그려질 거야.

------

## 3. Tools 계층 – 연구자/개발자가 쓰는 유틸들

`builddir/tools/`를 보면 여러 실행파일이 나와 있는데, 각각 이 라이브러리를 링크해서 특정 기능만 하는 작은 앱들이야. 

대표적인 것들:

- `harmony_logger` / `data_logger.cpp.o`
   → **로봇 상태(Joint state 등)를 시간에 따라 로그 파일로 저장**하는 툴.
- `harmony_exerciser` / `data_exerciser.cpp.o`
   → 로봇을 일정 패턴으로 움직여 보는 **운동/테스트 스크립트 실행기** 느낌.
   (예: 반복적인 shoulder abduction, ROM 테스트 같은 거)
- `commandAndReport` / `command_and_report.cpp.o`
   → 특정 명령을 보내고, 그 결과를 콘솔에 출력하는 CLI.
   (로봇 모드 변경, 특정 joint만 움직이기 등)
- `setController` / `set_controller.cpp.o`
   → 각 조인트/팔의 **컨트롤 모드(IMPEDANCE, POSITION, TORQUE 등)** 를 설정하는 툴로 추정.
- `printControllers` / `controller_printer.cpp.o`
   → 현재 각 조인트가 어떤 컨트롤 모드, 게인 값 등을 쓰고 있는지 출력.
- `printValues` / `value_printer.cpp.o`
   → joint state (각도/속도/토크 등) 혹은 기타 상태 변수를 한 번에 찍어보는 툴.
- `sendPositionsUDP` / `udp_sender.cpp.o`
   → **UDP로 조인트 포지션 명령을 송신**하는 툴.
   → 네트워크를 통해 외부 프로그램(예: Python GUI, 다른 PC)에서 명령을 보낼 때 쓸 수 있음.
- `udpEchoServer` / `udp_echo_server.cpp.o`
   → UDP 통신 테스트용 echo server. 네트워크 latency, 패킷 로스 확인용.
- `stubHarmony` / `stub_harmony.cpp.o`
   → 진짜 로봇 없이 **가짜 Harmony 프로세스**를 띄워서 shared memory 등 인터페이스만 흉내내는 시뮬레이터.
   → 연구 코드 개발/디버깅을 PC에서만 할 때 유용.

이 툴들 구조만 봐도 “실제 로봇은 항상 별도의 제어 프로세스로 돌고 있고,
 우리는 shared memory / UDP / 라이브러리 API를 통해 간접적으로 제어한다”는 구조가 보이지?

------

## 4. Tests – 내부 모듈 검증

- `builddir/tests/` 아래에
  - `arm_controller_tests`
  - `joint_states_tests`
  - `research_interface_tests`
  - `shared_memory_manager_tests`
  - `size_tests`
     등 여러 테스트 실행파일이 있어. 

이건 `include/` + `src/`에 있는 각 모듈들이 제대로 동작하는지
 GoogleTest (`subprojects/googletest-release-1.11.0`)로 검증하는 용도.

------

## 5. 빌드/의존성 계층

- `meson.build`, `src/meson.build`
   → Meson 빌드 스크립트. 어떤 소스가 어떤 라이브러리를 만들고 어떤 툴을 만드는지 정의.
- `builddir/meson-info`, `builddir/meson-logs`, `builddir/meson-private`
   → Meson이 내부 상태/로그를 저장하는 폴더들.
- `subprojects/eigen-3.4.0`
   → 로봇 제어/선형대수 연산에 쓰는 Eigen 라이브러리 전체.
   → 여기 안에 bench, test, doc 등 엄청 많지만, **실제 우리가 쓰는 건 헤더들(Eigen/Core 등)** 이라고 보면 됨.

------

## 6. “로봇 내부 시스템”을 소프트웨어 관점에서 한 줄로 정리하면

> **로봇 실시간 제어기(다른 프로세스)가 shared memory에 joint state와 command 버퍼를 열어주고,
>  이 레포의 `research_interface`와 `arm_controller`가 그 버퍼를 C++에서 다루기 쉽게 감싸고,
>  `harmony_logger`, `harmony_exerciser` 같은 여러 툴과 네트워크(UDP)를 통해
>  연구자가 로봇을 제어하고 데이터를 읽는 구조**

라고 보면 될 것 같아.
