---
title: Why the Sample Variance Divides by n − 1
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251022
tags: statistics
lang: en
---

## Population variance (with population mean \(\mu\))

\[
\sigma^2
= \frac{(X_1-\mu)^2 + \cdots + (X_N-\mu)^2}{N}
= \frac{1}{N}\sum_{i=1}^{N}(X_i-\mu)^2
\]

## Sample variance (with sample mean \(\bar X\))

\[
S^2
= \frac{(X_1-\bar X)^2 + (X_2-\bar X)^2 + \cdots + (X_n-\bar X)^2}{\,n-1\,}
= \frac{1}{n-1}\sum_{i=1}^{n} (X_i-\bar X)^2
\]

Why do we divide the sum of \(n\) squared deviations by **\(n-1\)**?

---

## Parameters \(\leftrightarrow\) Sample statistics (estimation map)

\[
\begin{aligned}
\text{Population mean: } &\ \mu 
\;\; \xleftarrow{\ \text{estimated by}\ }\;\;
\text{Sample mean: }\ \bar X \\[6pt]
\text{Population variance: } &\ \sigma^{2} 
\;\; \xleftarrow{\ \text{estimated by}\ }\;\;
\text{Sample variance: }\ S^{2} \\[6pt]
\text{Population std. dev.: } &\ \sigma 
\;\; \xleftarrow{\ \text{estimated by}\ }\;\;
\text{Sample std. dev.: }\ S
\end{aligned}
\]

---

## Expectation (unbiasedness) relationships

\[
\mathbb{E}(\bar X)=\mu, 
\qquad 
\mathbb{E}(S^{2})=\sigma^{2}.
\]

---

## Motivation

The population size is typically too large to compute \(\mu,\sigma^2,\sigma\) directly.  
We draw a random sample of size \(n\), compute \(\bar X, S^2, S\), and **expect** these sample statistics to reflect the population features.

---

## Population mean and variance

\[
\begin{aligned}
\mu
&= \frac{X_1 + X_2 + \cdots + X_N}{N}
= \frac{1}{N}\sum_{i=1}^{N} X_i, \\[6pt]
\sigma^{2}
&= \frac{(X_1-\mu)^2 + (X_2-\mu)^2 + \cdots + (X_N-\mu)^2}{N}
= \frac{1}{N}\sum_{i=1}^{N}(X_i-\mu)^2.
\end{aligned}
\]

## Sample mean and (Bessel-corrected) sample variance

\[
\begin{aligned}
\bar X
&= \frac{X_1 + X_2 + \cdots + X_n}{n}
= \frac{1}{n}\sum_{i=1}^{n} X_i, \\[6pt]
S^{2}
&= \frac{(X_1-\bar X)^{2} + (X_2-\bar X)^{2} + \cdots + (X_n-\bar X)^{2}}{\,n-1\,}
= \frac{1}{n-1}\sum_{i=1}^{n}(X_i-\bar X)^{2}.
\end{a
