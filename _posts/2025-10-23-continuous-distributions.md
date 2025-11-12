---
title: 연속확률변수
sidebar:
  nav: docs-ko
aside:
  toc: true
key: 20251023
tags: 통계
lang: ko
math: true
---

## 이산 → 연속: 개념 전환
- **연속형 확률변수**: $\mathbb{R}$ 전체(또는 구간)에서 **비가산** 많은 값.
- 이산형의 **PMF** 대신 연속형은 **PDF** $f_X(x)$ 사용.

### PDF 정의/성질
- 구간 확률: $$P(a\le X\le b)=\int_a^b f_X(x)\,dx$$
- **비음성**: $f_X(x)\ge0$
- **정규화**: $$\int_{-\infty}^{\infty} f_X(x)\,dx=1$$
- 점확률: $P(X=a)=\int_a^a f_X(x)\,dx=0$

### 다변량 PDF
- **결합밀도** $f_{X,Y}(x,y)$:
  $$P(a\le X\le b,\;c\le Y\le d)=\int_c^d\int_a^b f_{X,Y}(x,y)\,dx\,dy$$
- **독립**이면 $f_{X,Y}(x,y)=f_X(x)f_Y(y)$

### 주변/조건부 밀도
- 주변: $$f_X(x)=\int_{-\infty}^{\infty}f_{X,Y}(x,y)\,dy$$
- 조건부: $$f_{X|Y}(x|y)=\frac{f_{X,Y}(x,y)}{f_Y(y)}$$

---

## 기대값/분산(연속형)
- $$\mu_X=E[X]=\int_{-\infty}^{\infty} x\,f_X(x)\,dx$$
- $$\sigma_X^2=\mathrm{Var}(X)=\int (x-\mu_X)^2 f_X(x)\,dx=E[X^2]-\mu_X^2$$

---

## 예시: **균등분포(유한 구간)**
- 중심 $a$, 폭 $b$인 구간 $[a-\tfrac b2,\;a+\tfrac b2]$에서
  $$
  f_X(x)=\frac{1}{b}\,\mathbf{1}\!\left\{\,a-\tfrac b2\le x\le a+\tfrac b2\,\right\}
  $$
- 평균/분산: $\mu=a$, $\ \sigma^2=\dfrac{b^2}{12}$

---

## 선형결합, 공분산, 상관

### 선형결합
- $L=\sum_{i=1}^n c_i X_i$
- $E[L]=\sum c_i E[X_i]$ (**선형성**, 독립 불필요)
- $$\mathrm{Var}(L)=\sum c_i^2\mathrm{Var}(X_i)+2\sum_{i<j} c_i c_j\,\mathrm{Cov}(X_i,X_j)$$
  - **서로 독립**이면 교차항 0 → $\sum c_i^2\mathrm{Var}(X_i)$

### 공분산/상관계수
- $\mathrm{Cov}(X,Y)=E[XY]-E[X]E[Y]$
- $$\rho_{XY}=\frac{\mathrm{Cov}(X,Y)}{\sigma_X\sigma_Y}\in[-1,1] \quad\text{(무차원)}$$
- **주의**: $\mathrm{Cov}(X,Y)=0$ (또는 $\rho=0$) ⇒ **독립 아님**(일반적으로)

#### 반례(강의 예시)
- 표준화 $X$ (평균 0, 분산 1), $Y=X^2$ (비선형 종속)  
  대칭 PDF에서 $E[X^3]=0$ → $\mathrm{Cov}(X,Y)=0$ 이지만 **종속**.

---

## 정규분포 소개
- 표기: $X\sim\mathcal{N}(\mu,\sigma^2)$
- **PDF**
  $$
  f(x)=\frac{1}{\sqrt{2\pi}\,\sigma}\exp\!\left(-\frac{(x-\mu)^2}{2\sigma^2}\right)
  $$
- 파라미터 영향
  - $\mu$ 변화: **중심 이동**(모양 동일)
  - $\sigma$ 증가: **분산↑/폭 넓어짐**

---

## 실전 메모
- **PDF는 면적=확률**: 적분으로만 확률 계산, 점확률 0
- **독립성 체크**: 이론에선 분산 합 공식의 교차항; 데이터에선 $\rho$만으로 독립 단정 금지
- **표준화**: $Z=(X-\mu)/\sigma$는 $\mathcal{N}(0,1)$ → 표준정규표/함수 사용

---

## 정규분포 $\mathcal{N}(\mu,\sigma^2)$

**PDF**
$$
f(x)=\frac{1}{\sqrt{2\pi}\,\sigma}\exp\!\left(-\frac{(x-\mu)^2}{2\sigma^2}\right)
$$

- $\mu$ 증가 → **수평 이동**, $\sigma$ 증가 → **더 넓고 낮아짐**  
- 표준정규: $Z\sim\mathcal{N}(0,1)$, CDF $\Phi(z)=P(Z\le z)$  
- **대칭성**: $\Phi(-z)=1-\Phi(z)$  
- **구간확률**: $P(a\le Z\le b)=\Phi(b)-\Phi(a)$

**68–95–99% 규칙(근사)**  
- $P(|X-\mu|\le 1\sigma)\approx 0.68$  
- $P(|X-\mu|\le 2\sigma)\approx 0.95$  
- $P(|X-\mu|\le 3\sigma)\approx 0.99$

**형상 포인트**  
- **변곡점**: $x=\mu\pm\sigma$  
- **FWHM**: $\text{FWHM}\approx 2.35\,\sigma$ (정규에서만 유효한 근사)

---

## 퍼센타일 & Z-점수
- 퍼센타일: $z_p$는 $P(Z\le z_p)=p$를 만족  
  - 예: $z_{0.75}\approx +0.675,\; z_{0.25}\approx -0.675$

**표준화**
$$
Z=\frac{X-\mu}{\sigma}
\quad\Rightarrow\quad
P(a\le X\le b)=\Phi\!\left(\frac{b-\mu}{\sigma}\right)-\Phi\!\left(\frac{a-\mu}{\sigma}\right).
$$

**예시 — 상한 초과 확률**  
$X\sim \mathcal{N}(8,2^2)$에서 $P(X>12)=1-\Phi(2)\approx 0.023$ (약 2.3%).

---

## 연속성 보정(Continuity Correction)
- 연속(정규)로 **이산** 범위를 근사할 때 **±0.5** 경계 보정.
- **반올림 측정**이 있는 연속변수에도 경계 해석이 바뀜.

**예시 — 안압(IOP)**  
$X\sim \mathcal{N}(16,3^2)$, 정상: 12–20 mmHg, **정수 반올림**.  
보정 경계: **11.5 ≤ X ≤ 20.5**
$$
P(11.5\le X\le 20.5)=\Phi(1.5)-\Phi(-1.5)\approx 0.866\;(86\!\sim\!87\%)
$$
보정 없으면(12–20 그대로) ≈ **82%** → 보정 중요.

---

## 정규로의 근사 (Binomial/Poisson)

### 이항분포 $\mathrm{Bin}(n,p)$
- **조건**: $np(1-p)\gtrsim 5$ & $p$가 0/1에 치우치지 않을 때.
- **근사**: $X\approx \mathcal{N}(np,\;np(1-p))$
- **연속성 보정**
  - $P(X\le k)\approx P(Y\le k+0.5)$
  - $P(X\ge k)\approx P(Y\ge k-0.5)$
  - $P(X=k)\approx P(k-0.5\le Y\le k+0.5)$
  - 경계 0, $n$에서는 $(-\infty,0.5],\,[n-0.5,\infty)$ 사용

### 포아송 $\mathrm{Pois}(\mu)$
- **조건**: $\mu\gtrsim 10$이면 대칭성 좋아져 근사 양호.
- **근사**: $X\approx \mathcal{N}(\mu,\mu)$ (보정 동일)

> *주의*: 스큐가 큰 경우(작은 $n$, 극단 $p$, 작은 $\mu$)엔 정규 근사 **부적절**.

---

## 실무 치트시트
1. **표준화부터**: 임의 $(\mu,\sigma)$ → $Z$, $\Phi$ 활용.
2. **대칭성**: $\Phi(-z)=1-\Phi(z)$.
3. **정수 경계는 0.5 보정**: 이산→연속, 반올림 경계 모두.
4. **적합성 점검**: Binomial은 $npq\ge 5$, Poisson은 $\mu\ge 10$.
5. **빠른 sanity check**: 68–95–99 규칙, FWHM $\approx 2.35\sigma$.

---

### 키 공식
- $Z=(X-\mu)/\sigma$
- $P(a\le Z\le b)=\Phi(b)-\Phi(a)$
- $\Phi(-z)=1-\Phi(z)$
- Binomial → Normal: $\mu=np,\;\sigma^2=np(1-p)$ (+ 보정)
- Poisson → Normal: $\mu=\sigma^2$ (+ 보정)
