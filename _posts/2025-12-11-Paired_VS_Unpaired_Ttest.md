---
title: Paired vs Unpaired t-test 
sidebar:
  nav: docs-ko
aside:
  toc: true
key: 20251211
tags: [t-test, paired, unpaired, nonparametric, biostat]
lang: ko
math: false

---

# Paired vs Unpaired t-test 정리  

이 글에서는

1. **paired vs unpaired** 개념을 쉽게 정리하고  
2. 교재 extra credit 문제(조산 방지 약물 + 아기 체중)를  
   - 대응 표본 t-검정(paired t-test)  
   - 독립 표본 t-검정(unpaired t-test)  
   - 비모수 검정(Wilcoxon rank-sum, Mann–Whitney U)  

으로 모두 풀어본다.

---

## 1. Paired vs Unpaired: 직관적으로 이해하기

### 1.1 Paired data (대응 표본, 쌍체 자료)

키워드: **같은 대상, 짝이 있는 두 값**

예시

- 같은 사람의 Before vs After  
  - 다이어트 약 복용 전/후 체중  
  - 금연 전/후 폐 기능
- 같은 환자의 왼팔 vs 오른팔 근력
- 비슷한 특징(나이, 체중 등)으로 **두 명씩 짝을 만든 후**  
  - 한 명은 treatment, 한 명은 control 로 배정한 경우

엑셀 테이블 모양:

| Pair | Treatment | Control |
| ---- | --------- | ------- |
| 1    | T1        | C1      |
| 2    | T2        | C2      |
| ...  | ...       | ...     |

각 행(row) 안에서 **T와 C가 1:1로 의미 있는 짝**을 이룬다.  

이럴 때는 “각 쌍마다 차이 d = T − C 를 구해서, 그 차이들의 평균이 0이냐?”를 보는 검정이 자연스럽다.  

→ 이것이 **대응 표본 t-검정(paired t-test)** 이고,  
→ 사실상 “차이값 d에 대한 일표본 t-검정”이라고 생각해도 된다.

장점

- 같은 pair 안에서 공통인 요인(체질, 생활습관 등) 때문에 생기는 변동이 **서로 상쇄**된다.
- 그래서 **처치(treatment) 때문에 생긴 차이만 더 선명하게** 보인다.
- 결과적으로 **변동 감소 → 검정력 증가 → p-value가 더 작아지는 경향**이 있다.

---

### 1.2 Unpaired data (독립 표본)

키워드: **서로 다른 사람들, 짝이 없음**

예시

- 남자 20명 vs 여자 20명 키 비교
- 병원 A 환자 30명 vs 병원 B 환자 30명
- treatment 그룹 15명, control 그룹 15명인데  
  누가 누구랑 짝이라는 정보가 없거나, 짝 정보를 쓰지 않기로 한 경우

엑셀 테이블 모양:

| ID   | Group | Weight |
| ---- | ----- | ------ |
| 1    | T     | ...    |
| 2    | T     | ...    |
| …    | C     | ...    |

→ 이럴 때는 **두 집단의 평균**을 비교하는  
**독립 이표본 t-검정(unpaired two-sample t-test)** 을 쓴다.

검정 구조

- H0: treatment 평균 = control 평균  
- H1: treatment 평균 ≠ control 평균

pair 정보를 활용하지 않기 때문에,  
같은 데이터라면 보통 **paired 분석보다 검정력이 조금 약해질 수 있다.**

---

### 1.3 한 줄 요약

- 엑셀에서 **같은 행에 “T값, C값”이 같이 있으면 → paired**  
- 그룹이 그냥 따로따로만 있으면 → unpaired  

이번 아기 체중 문제는 “임산부를 두 명씩 묶어서, 한 명은 treatment, 한 명은 control” 이라서  
**쌍(pair)을 만들 수 있고, 따라서 paired 분석이 가능**하다.  
하지만, 쌍 정보를 무시하고 그냥 두 집단으로만 보면서 **unpaired 분석**을 할 수도 있다.

---

## 2. 대응 표본 t-검정(paired t-test) 개념 요약

다음과 같이 **같은 사람의 Before / After** 데이터를 생각해 보자.

- Before: placebo (플라시보 복용 시)
- After: drug (약물 복용 시)

각 사람 i에 대해

- Before 값 = B_i  
- After 값  = A_i  
- 차이 d_i = After − Before (또는 Before − After, 방향은 연구자가 정함)

그럼 대응 표본 t-검정은 다음 순서로 생각할 수 있다.

1. 모든 i에 대해 **차이 d_i** 를 계산한다.
2. d 값들의 평균: d_bar
3. d 값들의 표준편차: sd_d
4. 차이 평균의 표준오차: SE_d = sd_d / sqrt(n)
5. t 값 계산:

   - t = d_bar / SE_d

6. 자유도 df = n − 1 에 대해 t-분포를 사용하여 p-value 계산

해석

- t 값이 0에서 멀리 떨어질수록,  
  “차이 평균이 진짜 0일 가능성은 적다” → 약효가 있다고 볼 근거가 강해진다.

핵심 아이디어

- **독립 t-test vs paired t-test 모두 “차이 / 불확실성” 구조**를 가진다.
- paired t-test는 “사람별 차이”를 직접 다루기 때문에  
  처치 효과를 좀 더 민감하게 포착할 수 있다.

---

## 3. Extra credit: Baby weight 예제 정리

### 3.1 문제 요약

임상시험: 조산 방지 약물이 **출생 체중에 영향을 주는지** 보고 싶다.

- 총 30명 임산부
  - 15명 → treatment (약물)
  - 15명 → control (placebo)
- 24–28주 사이에 한 번 약 or placebo를 복용
- 두 명씩 짝을 만들어
  - 한 명을 treatment, 한 명을 control 로 배정 (무작위)

출생 체중(파운드)이 다음과 같이 주어졌다고 하자.

#### Treatment 그룹 (15명, lb)

6.9, 7.6, 7.3, 7.6, 6.8, 7.2, 8.0, 5.5, 5.8, 7.3, 8.2, 6.9, 6.8, 5.7, 8.6

#### Control 그룹 (15명, lb)

6.4, 6.7, 5.4, 8.2, 5.3, 6.6, 5.8, 5.7, 6.2, 7.1, 7.0, 6.9, 5.6, 4.2, 6.8

요약 통계량(계산 결과)

- n1 = n2 = 15
- treatment 평균 ≈ 7.08 lb  
- control 평균  ≈ 6.26 lb  
- 평균 차이 (T − C) ≈ 0.82 lb

---

## 4. (a) Paired vs Unpaired 분석

### 4.1 Paired t-test

각 pair별로 “같은 줄”의 두 값을 사용해 **차이 d = T − C** 를 만든다.

차이 d 값들

- 0.5, 0.9, 1.9, -0.6, 1.5, 0.6, 2.2, -0.2, -0.4, 0.2, 1.2, 0.0, 1.2, 1.5, 1.8

요약

- d_bar (평균 차이) ≈ 0.82 lb  
- sd_d (차이의 표준편차) ≈ 0.887  
- SE_d = sd_d / sqrt(15) ≈ 0.229  

t 통계량

- t = d_bar / SE_d ≈ 0.82 / 0.229 ≈ 3.58  
- 자유도 df = 15 − 1 = 14  

p-value (양측 기준)

- t = 3.58, df = 14 → p 값은 대략 0.003 근처 (0.01보다 작음)

대략적인 95% 신뢰구간

- 약 0.33 lb ~ 1.31 lb

해석 (paired 분석)

- 약물을 투여한 그룹이 **평균적으로 약 0.8 lb 더 무겁게 태어났다.**
- 이 차이는 **통계적으로 유의**하다 (p ≈ 0.003).  
- 따라서 “약물이 출생 체중을 증가시킨다”는 주장에 대한 근거가 꽤 강하다고 볼 수 있다.

---

### 4.2 Unpaired two-sample t-test (equal variance 가정)

이번에는 **쌍 정보를 무시**하고, 단순히 두 집단 평균만 비교해 본다.

각 그룹 표준편차(계산 결과)

- treatment sd ≈ 0.90  
- control sd ≈ 0.96  

pooled 표준편차

- s_p ≈ 0.93  

평균 차이의 표준오차

- SE = s_p × sqrt(1/n1 + 1/n2) ≈ 0.340  

t 통계량

- t = (7.08 − 6.26) / 0.340 ≈ 2.41  
- df = 15 + 15 − 2 = 28  

p-value (양측 기준)

- t = 2.41, df = 28 → p ≈ 0.02

대략적인 95% 신뢰구간

- 약 0.12 lb ~ 1.52 lb

해석 (unpaired 분석)

- 여기서도 treatment 그룹이 control 그룹보다 **유의하게 더 무겁다** (p ≈ 0.02).
- 다만 paired 분석에 비하면 p 값이 조금 더 크다 → 검정력이 약간 낮다.

---

### 4.3 (a) 질문에 대한 답: 분석 방식이 결론을 바꾸는가?

- **두 방법 모두**
  - treatment 그룹 아기 체중이 control 그룹보다 크다.
  - 즉, “약물이 효과가 있다”는 **같은 방향의 결론**을 준다.
- 차이점
  - paired t-test: p ≈ 0.003 (증거가 매우 강함)
  - unpaired t-test: p ≈ 0.02 (여전히 유의하지만 상대적으로 덜 강함)

정리

> 결론 자체는 같지만,  
> **paired 분석이 더 강한 통계적 증거를 제공**한다.  
> (쌍 정보를 활용하면 노이즈가 줄어 검정력이 증가하기 때문)

---

## 5. (b) 비모수 방법 – unpaired 가정

이번에는 “쌍을 무시하고” 두 집단이 **독립**이라고 가정한 뒤,  
**Wilcoxon rank-sum test (Mann–Whitney U test)** 로 다시 분석해 보자.

### 5.1 절차 개념

1. treatment + control 30명 아기 체중을 전부 한 리스트로 모은다.
2. 작은 값부터 오름차순 정렬하고, 1, 2, 3, … 순위를 부여한다.  
   (동점이 있으면 평균 순위 사용)
3. treatment 그룹이 차지한 순위들의 합 R_T 를 구한다.
4. Wilcoxon U 통계량으로 변환해서 p-value 를 구한다.

계산 결과 (정렬 후 순위 합산)

- treatment 순위합 R_T ≈ 290.5  
- control 순위합 R_C ≈ 174.5  

Mann–Whitney U 통계량

- U1 = R_T − n1(n1 + 1)/2 = 290.5 − 120 = 170.5  
- U2 = n1*n2 − U1 = 225 − 170.5 = 54.5 (두 값 중 하나만 써도 됨)

정규 근사

- E(U) = n1*n2 / 2 = 112.5  
- Var(U) = n1*n2*(n1 + n2 + 1) / 12 ≈ 581.25  
- sd_U = sqrt(Var(U)) ≈ 24.11  
- z = (U1 − E(U)) / sd_U ≈ (170.5 − 112.5) / 24.11 ≈ 2.41  

p-value (양측 기준)

- z ≈ 2.41 → p ≈ 0.016

해석 (비모수, unpaired)

- p ≈ 0.016 < 0.05  
- 따라서 비모수 방법으로도  
  treatment 그룹 아기 체중이 control 그룹보다 **유의하게 더 크다**는 결론이 나온다.

---

## 6. (c) 비모수 방법이 더 나은가?

비교 대상

- 모수적(parametric) 방법
  - paired t-test
  - unpaired two-sample t-test
- 비모수(nonparametric) 방법
  - Wilcoxon rank-sum (Mann–Whitney U)

### 6.1 자료의 특징

- 변수: 출생 체중 (lb) → 생체 데이터 중에서도 비교적 **정규에 가까운 연속형 변수**
- 극단적인 이상값(outlier)이 거의 없음
- 샘플 크기: n1 = n2 = 15 → 너무 작지도, 아주 크지도 않은 수준

이런 상황에서는

- t-test의 정규성 가정이 크게 위배되었다고 보기 어렵고,

- 실제로

  - paired t-test
  - unpaired t-test
  - Wilcoxon rank-sum

  세 가지 모두 **같은 방향의 결론**을 준다.

### 6.2 결론 (c에 대한 답)

- 이 데이터에서는 출생 체중 분포가 크게 일그러져 있지 않고,
  이상값도 없어 보인다.
- 따라서 **모수적 t-test, 특히 paired t-test가 더 자연스럽고 효율적**이다.
- 비모수 검정은 “분포가 심하게 비대칭이거나 이상값이 많을 때” 특히 유용하지만,  
  이 예제에서는 그 정도로 비정상적인 분포는 아니다.

요약

> 이 예제에서는  
> 비모수 방법(b)보다는 **모수적 paired t-test를 주 방법(Main analysis)** 으로 사용하는 것이 적절하다.  
> 비모수 검정은 “보조 확인용”으로 쓰는 정도가 좋다.

---

## 7. 전체 요약

1. **paired vs unpaired**
   - 같은 행에 “T값, C값”이 함께 있으면 → paired 설계  
   - 그룹이 따로따로만 있으면 → unpaired 설계  

2. 이 아기 체중 예제에서
   - paired t-test: p ≈ 0.003 → 약물이 출생 체중을 증가시킨다는 증거가 매우 강함
   - unpaired t-test: p ≈ 0.02 → 여전히 유의하지만, paired 보다는 덜 강함
   - Wilcoxon rank-sum: p ≈ 0.016 → 비모수로도 일관된 결론

3. 분포가 크게 이상하지 않고, 짝 정보가 있는 설계라면
   - **paired t-test가 가장 강력하고 자연스러운 선택**이다.