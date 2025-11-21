---
title: 임피던스 컨트롤
sidebar:
  nav: docs-ko
aside:
  toc: true
key: 20251120
tags: [impedance, control]
lang: en
math: true

---

## 임피던스 컨트롤 

![스크린샷 2025-11-20 112033]({{ "/assets/images/2025-11-20-impedance-control/스크린샷 2025-11-20 112033.png" | relative_url }})

## 1. 개요

로봇 팔이나 재활 로봇에서 **사람이 느끼는 저항감(무게감, 스프링 느낌)** 을 설계하고 싶을 때  
가장 많이 쓰는 방법이 **임피던스 컨트롤(impedance control)** 이다.

임피던스 컨트롤의 아이디어는 단순하다.

> _“사람이 조인트를 밀었을 때, 조인트가 **가상의 질량–댐퍼–스프링 시스템** 처럼 반응하게 만들자.”_

그래서 실제 로봇의 관성, 마찰, 중력과 상관없이  
사람 입장에서는 우리가 정한 $I_d, b_d, k_d$ 값대로 동작하는 것처럼 느끼게 된다.

위 그림처럼 모터가 달린 조인트에 사람이 토크를 가한다고 생각해 보자.

---

## 2. 조인트 동역학 모델

먼저 조인트의 실제 동역학은 다음과 같이 쓴다.

$$
I\ddot\theta + g(\theta) = \tau_m + \tau_I
$$

(1)

- $I$: 조인트의 실제 관성(moment of inertia)  
- $\theta$: 조인트 각도  
- $g(\theta)$: 중력에 의한 토크(중력 보상 항)  
- $\tau_m$: **모터(컨트롤러)가 조인트에 가하는 제어 토크**  
- $\tau_I$: **사람(또는 환경)이 조인트에 가하는 외란/인터랙션 토크**

즉, 모터 토크와 사람 토크의 합이 관성 + 중력 항을 만들어 내는 것이 (1)식이다.

---

## 3. 임피던스 컨트롤러의 형태

임피던스 컨트롤에서는 **사람이 느낄 “가상의” 질량–댐퍼–스프링**을 정해 준다.

- 원하는 질량: $I_d$  
- 원하는 점성(댐핑): $b_d$  
- 원하는 스프링 상수: $k_d$  
- 원하는 궤적(각도, 속도, 가속도): $\theta_d, \dot\theta_d, \ddot\theta_d$

컨트롤러는 모터 토크를 다음과 같이 만든다고 가정한다.

$$
\tau_m =
I\ddot\theta + g(\theta)
+ I_d(\ddot\theta_d - \ddot\theta)
+ b_d(\dot\theta_d - \dot\theta)
+ k_d(\theta_d - \theta)
$$

(2)

앞의 $I\ddot\theta + g(\theta)$ 항은 **실제 관성과 중력을 보상**해 주는 부분이고,  
뒤의 세 항이 **가상의 mass–damper–spring** 에 해당하는 부분이다.

---

## 4. 외력–변위 관계 유도

이제 (2)를 조인트 동역학 (1)에 대입해 보자.

(1)식에 (2)를 넣으면

$$
I\ddot\theta + g(\theta)
=
\Big[
I\ddot\theta + g(\theta)
+ I_d(\ddot\theta_d - \ddot\theta)
+ b_d(\dot\theta_d - \dot\theta)
+ k_d(\theta_d - \theta)
\Big]
+ \tau_I
$$

좌우에서 $I\ddot\theta + g(\theta)$를 지워주면

$$
I_d(\ddot\theta_d - \ddot\theta)
+ b_d(\dot\theta_d - \dot\theta)
+ k_d(\theta_d - \theta)
=
-\,\tau_I
$$

부호 정의를 바꾸어(또는 오른쪽으로 넘겨서) 정리하면 보통 다음과 같이 쓴다.

$$
I_d(\ddot\theta_d - \ddot\theta)
+ b_d(\dot\theta_d - \dot\theta)
+ k_d(\theta_d - \theta)
= \tau_I
$$

(3)

이 식이 **임피던스 컨트롤의 핵심**이다.

> 사람(외부)이 가하는 토크 $\tau_I$ 가  
> “가상의 질량–댐퍼–스프링” $(I_d, b_d, k_d)$ 의 **오차에 비례하는 값**이 되도록  
> 컨트롤러가 $\tau_m$ 을 만들어 준다.

즉, 사람 입장에서는 조인트가  
$\theta_d$ 주변에서 질량 $I_d$, 댐핑 $b_d$, 스프링 $k_d$ 를 가진 시스템처럼 느껴진다.

---

## 5. 요약

- $\tau_m$: 모터가 조인트에 가하는 제어 토크  
- $\tau_I$: 사람이(또는 환경이) 조인트에 가하는 외란 토크  
- 임피던스 컨트롤은 **실제 로봇의 관성·중력을 보상**하고  
  **사람이 느끼는 가상의 mass–damper–spring 파라미터 $I_d, b_d, k_d$** 를 설계하는 컨트롤이다.
- 최종적으로 사람 토크 $\tau_I$ 와 조인트의 운동 오차 $(\theta_d - \theta)$ 사이의 관계가  
  (3)식과 같은 **선형 임피던스 관계**를 만족하도록 만드는 것이 목표다.

이렇게 정리해 두면, 나중에 재활 로봇 실험에서  
“어깨 조인트에 어떤 임피던스 값을 넣었는지”,  
“사람이 느끼는 stiffness/damping을 어떻게 설계했는지” 를 설명할 때 그대로 가져다 쓸 수 있다.
