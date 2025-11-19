---
title: Central Limit Theorem
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251031
tags: [central limit theorem]
lang: en
math: true
---

## Introduction

Many statistics in practice appear to follow a **normal distribution**.

For example, if you take a large integer and count the number of prime factors it has, the distribution of that count (over many integers) is also close to a normal distribution.

Why does this happen?

Consider a Galton board: we drop a huge number of balls through a triangular array of pins.

(We assume each ball passes independently, like a “ghost,” not interacting with others.)

As we record the positions where the balls land, the distribution of counts across bins becomes more and more **bell-shaped**.

---

## General Idea

Let \(X\) be a random variable, and take a sample of size \(N\):

- Sum: \(S_N = X_1 + X_2 + \cdots + X_N\)  
- (Later we can also consider the sample mean \(\bar X_N = S_N / N\).)

As \(N \to \infty\), the distribution of the **sum** (after proper centering and scaling) gets closer and closer to a **normal** (bell-shaped) distribution.

We then ask:

> Can we find an interval in which the sum falls with, say, 95% confidence?

To answer this, we first recall how variance and standard deviation behave under sums.

---

## Variance and Standard Deviation of Sums

Let \(X_1, \dots, X_n\) be independent and identically distributed (i.i.d.) random variables with variance \(\sigma_{X_1}^2\).

Then

$$
\sigma_{X_1 + \cdots + X_n}^2
= n \cdot \sigma_{X_1}^2
$$

and

$$
\sigma_{X_1 + \cdots + X_n}
= \sqrt{n}\,\sigma_{X_1}.
$$

Even as we increase the sample size \(n\), the spread of the **sum** grows only at the rate \(\sqrt{n}\).

If we then:

1. Recentre the distribution around its **mean**, and  
2. Rescale it so that the **standard deviation is 1**,

the shape of the distribution starts to converge to a **universal form**.

Surprisingly, this holds regardless of the specific distribution of a single trial (for example, a single die roll), as long as basic conditions are satisfied. Even if the original distribution is not uniform, the standardized sum tends toward the same bell-shaped curve.

In the limit, the density approaches

$$
\frac{1}{\sqrt{2\pi}} e^{-\frac{1}{2} x^{2}}.
$$

---

## Meaning of the Normal Density Formula

The general normal density is

$$
\frac{1}{\sigma \sqrt{2\pi}} \,
\exp\!\left(
  -\frac{1}{2}
  \left(\frac{x - \mu}{\sigma}\right)^2
\right).
$$

This is the density of a distribution whose **total area is 1** and whose **standard deviation is \(\sigma\)**.

When \(\mu = 0\) and \(\sigma = 1\), we call it the **standard normal distribution**.

In the context of the central limit theorem, we consider sums (or means) of samples and then:

- subtract the mean,  
- divide by the appropriate standard deviation,

to obtain a **standardized** random variable. For large sample size \(N\), this standardized quantity behaves approximately like a standard normal variable \(Z\).

Formally, for suitable i.i.d. random variables:

$$
\lim_{N \to \infty} P\bigl(a < Z_N < b\bigr)
= \int_{a}^{b}
  \frac{1}{\sqrt{2\pi}} e^{-x^{2}/2} \, dx,
$$

where \(Z_N\) is the appropriately standardized sum (or mean). The right-hand side is the probability that a standard normal variable falls between \(a\) and \(b\).

---

## The Central Limit Theorem (Informal Statement)

Let \(X_1, X_2, \dots\) be i.i.d. random variables with

- mean \(E[X_i] = \mu\), and  
- variance \(\mathrm{Var}(X_i) = \sigma^2 \in (0, \infty)\).

Define the **sample mean**

$$
\bar X_N = \frac{1}{N}\sum_{i=1}^{N} X_i.
$$

Then, as \(N \to \infty\),

$$
Z_N = \frac{\bar X_N - \mu}{\sigma / \sqrt{N}}
$$

converges in distribution to a standard normal variable \(Z \sim \mathcal{N}(0, 1)\).

In other words, for large \(N\),

$$
P(a < Z_N < b)
\approx
P(a < Z < b)
=
\int_{a}^{b}
\frac{1}{\sqrt{2\pi}} e^{-x^{2}/2} \, dx.
$$

This is the **central limit theorem**: the distribution of the standardized sample mean becomes approximately normal, even if the original data are not normally distributed.

---

## Useful Rule for Normal Distributions

The **68–95–99.7 rule** (empirical rule):

- About **68%** of values lie within **1 standard deviation** of the mean.  
- About **95%** of values lie within **2 standard deviations**.  
- About **99.7%** of values lie within **3 standard deviations**.

So, if \(X \sim \mathcal{N}(\mu, \sigma^2)\),

- \(P(|X - \mu| \le \sigma) \approx 0.68\),  
- \(P(|X - \mu| \le 2\sigma) \approx 0.95\),  
- \(P(|X - \mu| \le 3\sigma) \approx 0.997\).

These are approximations, but very useful for quick intuition.

---

## Caveats and Misuse

- Not **every** variable should be modeled as normal.  
- Saying “this is a 3-sigma event, so \(p < 0.003\)” is not always justified unless you have a good reason to assume a normal model for that variable and appropriate independence assumptions.

The CLT provides a powerful approximation, but you should always check whether its **assumptions** are reasonably satisfied.

---

## Assumptions of the Central Limit Theorem

In a basic i.i.d. version of the CLT, we assume:

1. All \(X_i\) are **independent**.  
2. All \(X_i\) are identically distributed (same distribution).  
3. The variance is finite and strictly positive:
   $$
   0 < \mathrm{Var}(X_i) < \infty.
   $$

Under these conditions, the standardized sum (or sample mean) converges to a normal distribution as the sample size becomes large.
