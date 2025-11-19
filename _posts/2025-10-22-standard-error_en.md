---
title: Why the Sample Variance Divides by n − 1
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251022
tags: [statistics]
lang: en
math: true
---

## Population variance (with population mean \(\mu\))

$$
\sigma^2
= \frac{(X_1-\mu)^2 + \cdots + (X_N-\mu)^2}{N}
= \frac{1}{N}\sum_{i=1}^{N}(X_i-\mu)^2
$$

## Sample variance (with sample mean \(\bar X\))

$$
S^2
= \frac{(X_1-\bar X)^2 + (X_2-\bar X)^2 + \cdots + (X_n-\bar X)^2}{\,n-1\,}
= \frac{1}{n-1}\sum_{i=1}^{n} (X_i-\bar X)^2
$$

Why do we divide the sum of \(n\) squared deviations by **\(n-1\)** instead of \(n\)?

---

## Parameters and sample statistics (estimation map)

$$
\begin{aligned}
\text{Population mean: } &\ \mu 
\;\; \xleftarrow{\ \text{estimated by}\ }\;\;
\text{Sample mean: }\ \bar X \\[6pt]
\text{Population variance: } &\ \sigma^{2} 
\;\; \xleftarrow{\ \text{estimated by}\ }\;\;
\text{Sample variance: }\ S^{2} \\[6pt]
\text{Population std. deviation: } &\ \sigma 
\;\; \xleftarrow{\ \text{estimated by}\ }\;\;
\text{Sample std. deviation: }\ S
\end{aligned}
$$

---

## Expectation (unbiasedness) relationships

$$
\mathbb{E}(\bar X)=\mu, 
\qquad 
\mathbb{E}(S^{2})=\sigma^{2}.
$$

So, on average, the sample mean and the Bessel–corrected sample variance match the **true** mean and variance.

---

## Motivation

The population is usually too large to compute \(\mu,\sigma^2,\sigma\) directly.  
Instead, we draw a random sample of size \(n\), compute \(\bar X, S^2, S\), and expect these sample statistics to capture the main features of the population.

---

## Population mean and variance

$$
\begin{aligned}
\mu
&= \frac{X_1 + X_2 + \cdots + X_N}{N}
= \frac{1}{N}\sum_{i=1}^{N} X_i, \\[6pt]
\sigma^{2}
&= \frac{(X_1-\mu)^2 + (X_2-\mu)^2 + \cdots + (X_N-\mu)^2}{N}
= \frac{1}{N}\sum_{i=1}^{N}(X_i-\mu)^2.
\end{aligned}
$$

## Sample mean and (Bessel-corrected) sample variance

$$
\begin{aligned}
\bar X
&= \frac{X_1 + X_2 + \cdots + X_n}{n}
= \frac{1}{n}\sum_{i=1}^{n} X_i, \\[6pt]
S^{2}
&= \frac{(X_1-\bar X)^{2} + (X_2-\bar X)^{2} + \cdots + (X_n-\bar X)^{2}}{\,n-1\,}
= \frac{1}{n-1}\sum_{i=1}^{n}(X_i-\bar X)^{2}.
\end{aligned}
$$

---

## Why do we divide by \(n-1\)? (intuition)

1. We first **estimate \(\mu\)** using the same data: \(\bar X\).  
   This makes the observations appear slightly closer to the center than they really are, so their spread around \(\bar X\) is **systematically too small**.

2. Therefore,
   \[
   \frac{1}{n}\sum_{i=1}^{n}(X_i-\bar X)^2
   \]
   underestimates the true variance on average; it is a **biased** estimator.

3. Dividing by a smaller number, \(n-1\), inflates the value just enough so that
   \(\mathbb{E}(S^2)=\sigma^2\).  
   This correction is called **Bessel’s correction** and makes \(S^2\) an **unbiased** estimator of the population variance.

---

## Sketch: unbiasedness of \(S^2\)

Assume \(X_1,\dots,X_n\) are i.i.d. with mean \(\mu\) and variance \(\sigma^2\).  

One can show that

$$
\sum_{i=1}^{n}(X_i-\bar X)^2
=
\sum_{i=1}^{n}(X_i-\mu)^2
- n(\bar X-\mu)^2.
$$

Taking expectations on both sides and using
\(\mathbb{E}(X_i-\mu)^2 = \sigma^2\) and
\(\mathbb{E}(\bar X-\mu)^2 = \sigma^2/n\), we get

$$
\begin{aligned}
\mathbb{E}\!\left[\sum_{i=1}^{n}(X_i-\bar X)^2\right]
&= n\sigma^2 - n\cdot\frac{\sigma^2}{n} \\
&= (n-1)\sigma^2.
\end{aligned}
$$

Therefore,

$$
\mathbb{E}(S^2)
= \mathbb{E}\!\left[\frac{1}{n-1}
\sum_{i=1}^{n}(X_i-\bar X)^2\right]
= \sigma^2.
$$

So dividing by **\(n-1\)** instead of \(n\) makes \(S^2\) an unbiased estimator of the true variance.
