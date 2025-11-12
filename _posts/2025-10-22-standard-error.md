---
title: 표본분산은 n - 1로 나눈다
sidebar:
  nav: docs-ko
aside:
  toc: true
key: 20251022
tags: 통계
lang: ko
---



## 모분산의 정의 $(\mu:\ \text{population mean})$

$$
\sigma^2
= \frac{(X_1-\mu)^2 + \cdots + (X_N-\mu)^2}{N}
= \frac{1}{N}\sum_{i=1}^{N}(X_i-\mu)^2
$$

## 표본분산의 정의 $(\bar X:\ \text{sample mean})$


$$
S^2
= \frac{(X_1-\bar X)^2 + (X_2-\bar X)^2 + \cdots + (X_n-\bar X)^2}{\,n-1\,}
= \frac{1}{n-1}\sum_{i=1}^{n} (X_i-\bar X)^2
$$

n개의 편차제곱을 더하는데 왜 n - 1로 나눌까? 

## 모수 ↔ 표본통계량 (추정 관계)

$$
\begin{aligned}
\text{모평균: } &\ \mu 
\;\; \xleftarrow{\ \text{추정}\ }\;\;
\text{표본의 평균: }\ \bar X\ (\text{표본평균}) \\[6pt]
\text{모분산: } &\ \sigma^{2} 
\;\; \xleftarrow{\ \text{추정}\ }\;\;
\text{표본의 분산: }\ S^{2}\ (\text{표본분산}) \\[6pt]
\text{모표준편차: } &\ \sigma 
\;\; \xleftarrow{\ \text{추정}\ }\;\;
\text{표본의 표준편차: }\ S\ (\text{표본표준편차})
\end{aligned}
$$

---

## 기대값(불편성) 관계

$$
\begin{aligned}
&\mu\;(\text{모평균})
\;\; \xleftarrow{\ \text{추정}\ }\;\;
\mathbb{E}\!\left(\bar X\right)\;(\text{표본평균의 기대값}) \\[8pt]
&\sigma^{2}\;(\text{모분산})
\;\; \xleftarrow{\ \text{추정}\ }\;\;
\mathbb{E}\!\left(S^{2}\right)\;(\text{표본분산의 기대값})
\end{aligned}
$$

<!-- 필요하면 등식으로도 명시 -->
$$
\mathbb{E}(\bar X)=\mu, \qquad \mathbb{E}(S^{2})=\sigma^{2}.
$$


## 모집단의 특성

모집단의 변량의 개수가 너무 많기 때문에 모평균, 모분산, 모표준편차를 구하기가 쉽지 않다.
(전부 조사하지 않으면 모른다.)

따라서, 모집단의 모수를 추정하기 위해 크기가 n인 표본을 임의 추출한다.

임의로 추출한 표본의 평균, 분산, 표준편차를 구한다.(모집단보다 개수가 작기 때문에 구하기 쉽다.)

### 우리는 표본들이 모집단의 특성을 나타내기를 기대한다


## 모평균, 모분산 유도

$$
\begin{aligned}
\mu
&= \frac{X_1 + X_2 + \cdots + X_N}{N}
= \frac{1}{N}\sum_{i=1}^{N} X_i \\[6pt]
\sigma^{2}
&= \frac{(X_1-\mu)^2 + (X_2-\mu)^2 + \cdots + (X_N-\mu)^2}{N}
= \frac{1}{N}\sum_{i=1}^{N}(X_i-\mu)^2
\end{aligned}
$$

<br><br>

## 표본평균, 표본분산 유도 

$$
\begin{aligned}
\bar X
&= \frac{X_1 + X_2 + \cdots + X_n}{n}
= \frac{1}{n}\sum_{i=1}^{n} X_i \\[6pt]
S^{2}
&= \frac{(X_1-\bar X)^{2} + (X_2-\bar X)^{2} + \cdots + (X_n-\bar X)^{2}}{\,n-1\,}
= \frac{1}{n-1}\sum_{i=1}^{n}(X_i-\bar X)^{2}
\end{aligned}
$$

즉, 모수를 잘 추정할 수 있도록 정의되어야 한다.

<br><br>

## 불편추정량(Unbiased estimator) 

추정량의 기댓값과 모수가 같은 추정량 

$$
\begin{array}{ccl}
\mathbb{E}(\bar X) & = & \mu \\
\text{(표본평균의 기대값)} & & \text{(모평균)} \\[10pt]
\mathbb{E}(S^{2}) & = & \sigma^{2} \\
\text{(표본분산의 기대값)} & & \text{(모분산)}
\end{array}
$$

## 유도 

$$
\begin{aligned}
\bar X
&= \frac{1}{n}\sum_{i=1}^{n} X_i
= \frac{X_1 + X_2 + X_3 + \cdots + X_n}{n}
\end{aligned}
$$

$$
\begin{aligned}
\mathbb{E}(\bar X)
&= \mathbb{E}\!\left(\frac{1}{n}\sum_{i=1}^{n} X_i\right)
= \frac{1}{n}\,\mathbb{E}\!\left(\sum_{i=1}^{n} X_i\right) \\[6pt]
&= \frac{1}{n}\left(\mathbb{E}[X_1]+\mathbb{E}[X_2]+\cdots+\mathbb{E}[X_n]\right)
= \frac{1}{n}\,(n\mu) \\[4pt]
&= \mu
\end{aligned}
$$



**표본 \(X_1, X_2, X_3, \ldots, X_n\) 에 대해서**

$$
S^{2}
= \frac{1}{\,n-1\,}\sum_{i=1}^{n}\bigl(X_i-\bar X\bigr)^{2}
$$

이라 정의하면,



$$
\begin{aligned}
\mathbb{E}(S^{2})
&= \mathbb{E}\!\left(\frac{1}{\,n-1\,}\sum_{i=1}^{n}\bigl(X_i-\bar X\bigr)^{2}\right) \\[4pt]
&= \frac{1}{\,n-1\,}\,\mathbb{E}\!\left(\sum_{i=1}^{n}\bigl(X_i-\bar X\bigr)^{2}\right)
\end{aligned}
$$

<br><br>


$$
\begin{aligned}
\mathbb{E}(S^{2})
&= \mathbb{E}\!\left(\frac{1}{\,n-1\,}\sum_{i=1}^{n}\bigl(X_i-\bar X\bigr)^{2}\right) \\[4pt]
&= \frac{1}{\,n-1\,}\,\mathbb{E}\!\left(\sum_{i=1}^{n}\bigl(X_i-\bar X\bigr)^{2}\right)
\end{aligned}
$$

<br><br>

$$
\begin{aligned}
\sum_{i=1}^{n}\bigl(X_i-\bar X\bigr)^2
&= \sum_{i=1}^{n}\bigl(X_i-\mu+\mu-\bar X\bigr)^2 \\[4pt]
&= \sum_{i=1}^{n}\Big\{(X_i-\mu)^2
    + 2(X_i-\mu)(\mu-\bar X)
    + (\mu-\bar X)^2\Big\} \\[6pt]
&= \sum_{i=1}^{n}(X_i-\mu)^2
   + 2(\mu-\bar X)\sum_{i=1}^{n}(X_i-\mu)
   + n(\mu-\bar X)^2 \\[6pt]
&= \sum_{i=1}^{n}(X_i-\mu)^2
   + 2(\mu-\bar X)\Bigg(\sum_{i=1}^{n}X_i-\sum_{i=1}^{n}\mu\Bigg)
   + n(\mu-\bar X)^2 \\[6pt]
&= \sum_{i=1}^{n}(X_i-\mu)^2
   + 2(\mu-\bar X)\,n(\bar X-\mu)
   + n(\mu-\bar X)^2
\end{aligned}
$$

<br><br>

$$
\sum_{i=1}^{n}\!\bigl(X_i-\bar X\bigr)^2
= \sum_{i=1}^{n}\!\bigl(X_i-\mu\bigr)^2
  - n\bigl(\mu-\bar X\bigr)^2
$$

<br><br>


$$
\begin{aligned}
E(S^2)
&= E\!\left(\frac{1}{\,n-1\,}\sum_{i=1}^{n} (X_i-\bar X)^2\right) \\[2pt]
&= \frac{1}{\,n-1\,}\,
   E\!\left(\sum_{i=1}^{n} (X_i-\bar X)^2\right) \\[2pt]
&= \frac{1}{\,n-1\,}\,
   E\!\left(\sum_{i=1}^{n} (X_i-\mu)^2 - n(\mu-\bar X)^2\right) \\[2pt]
&= \frac{1}{\,n-1\,}\left(\sum_{i=1}^{n} E[(X_i-\mu)^2]
    \;-\; n\,E[(\bar X-\mu)^2]\right) \\[2pt]
&= \frac{1}{\,n-1\,}\left(n\sigma^2 - n\cdot\frac{\sigma^2}{n}\right) \\[2pt]
&= \frac{1}{\,n-1\,}\bigl((n-1)\sigma^2\bigr) \;=\; \sigma^2 .
\end{aligned}
$$


<br><br>


$\displaystyle
\bar X \;=\; \frac{X_1+X_2+\cdots+X_n}{n}
\;=\; \frac{1}{n}\sum_{i=1}^{n} X_i
$

<br><br>

$\displaystyle
S^{2} \;=\;
\frac{(X_1-\bar X)^2+(X_2-\bar X)^2+\cdots+(X_n-\bar X)^2}{\,n-1\,}
\;=\; \frac{1}{\,n-1\,}\sum_{i=1}^{n}(X_i-\bar X)^2
$


<br><br>


이렇게 정의하면,

$\displaystyle
E(\bar X)=\mu \quad \text{(표본평균의 기대값)}
$

$\displaystyle
E(S^2)=\sigma^2 \quad \text{(표본분산의 기대값)}
$

$\displaystyle
E(S^2)
= E\!\left(\frac{1}{n}\sum_{i=1}^{n}(X_i-\bar X)^2\right)
= \frac{1}{n}\,E\!\left(\sum_{i=1}^{n}(X_i-\bar X)^2\right)
$

$\displaystyle
= \frac{1}{n}\,E\!\left(\sum_{i=1}^{n}(X_i-\mu)^2 \;-\; n(\mu-\bar X)^2\right)
= \frac{1}{n}\left( n\sigma^{2} \;-\; n\,E[(\mu-\bar X)^2] \right)
$

$\displaystyle
\text{since }E[(\mu-\bar X)^2]=\operatorname{Var}(\bar X)=\frac{\sigma^2}{n}
\;\Rightarrow\;
E(S^2)=\frac{1}{n}\left(n\sigma^{2}-n\cdot\frac{\sigma^{2}}{n}\right)
= \frac{n-1}{n}\,\sigma^{2}.
$

> 표본분산의 기대값은 모분산이 아니다:  
> $\;E(S^2)=\dfrac{n-1}{n}\sigma^2 \neq \sigma^2$  (편향 있음)

