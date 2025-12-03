---
title: Nonparametric Methods
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251201-en
tags: [Wilcoxon-signed-rank-test]
lang: en
math: true
---

## 1. Why do we need nonparametric tests?

Classical **parametric tests** like the t-test usually assume:

- The data follow a **normal distribution**.
- We estimate parameters of that distribution (mean, variance, …) and use them in the test.

In real data, the normality assumption is often violated.  
Then we start to worry:

> “Is it okay to use a t-test when the normality assumption is broken?”

When the sample size is large, the sample mean is approximately normal  
(thanks to the Central Limit Theorem), so people often **assume** the data
come from a normal distribution and proceed.

But if those assumptions are not reasonable?

In such cases we can use a **nonparametric test**.

Here we focus on

> **The nonparametric counterpart of the one-sample t-test**  
> → **Wilcoxon signed-rank test**

It is used to test a hypothesis about the **population median** without
assuming normality.

---

## 2. Assumptions of the Wilcoxon signed-rank test

This test needs only two assumptions:

1. The population distribution is **symmetric**.  
2. The data come from a **continuous** probability distribution.

These are much weaker than the normality assumption.

---

## 3. What are we trying to know?

Example:

- Suppose we have test scores of 12 students.
- We want to know whether the **population median** of these scores is 10.

We set up the hypotheses as

- Null hypothesis: $H_0: \eta = 10$ (population median = 10)  
- Alternative: $H_1: \eta \neq 10$

The t-test asks “Is the population mean $\mu = 10$?”  
Here we ask “**Is the population median $\eta = 10$?**”.

---

## 4. Test statistic $W^+$

The key test statistic of the Wilcoxon signed-rank test is

$$
W^{+} = \sum_{i=1}^{n} \psi\bigl(X_i - \eta_0\bigr)\, R_i
$$

where

- $X_i$: the $i$-th observation  
- $\eta_0$: the hypothesized median (e.g., 10)  
- $R_i$: **rank** of the $i$-th observation  
  - computed by ranking $|X_i - \eta_0|$ from smallest to largest  
    and assigning ranks 1, 2, …, $n$  
- $\psi(\cdot)$: an indicator (or sign) function  
  - if $X_i - \eta_0 > 0$, then $\psi = 1$  
  - if $X_i - \eta_0 \le 0$, then $\psi = 0$

So,

> **“Take only the observations larger than $\eta_0$,  
> and sum their ranks”**  

That sum is exactly $W^+$.

---

## 5. How to compute $W^+$ (algorithm style)

Let the data be $X_1, \dots, X_n$ and the hypothesized median be $\eta_0$.

1. **Compute the differences**

   $$
   d_i = X_i - \eta_0
   $$

2. **Compute distances (absolute values)**

   $$
   |d_i| = |X_i - \eta_0|
   $$

3. **Rank the absolute differences**

   - Order $|d_i|$ from smallest to largest and assign  
     **rank $R_i$** to each observation.  
   - If there are ties, use average ranks, etc.

4. **Keep only the values larger than $\eta_0$**

   - If $d_i > 0$, set $\psi(d_i) = 1$  
   - If $d_i \le 0$, set $\psi(d_i) = 0$

5. **Compute the test statistic**

   $$
   W^{+} = \sum_{i=1}^{n} \psi(d_i)\, R_i
   $$

   → This is the **sum of ranks for positive $d_i$**,  
   i.e., for observations on the **right side of $\eta_0$**.

We use $W^+$ to obtain a p-value and then decide whether to reject $H_0$.  
Here we focus on the **intuition** of $W^+$ rather than the exact p-value formula.

---

## 6. What does $W^+$ tell us? (intuition)

First, the sum of $n$ ranks $1,2,\dots,n$ is

$$
1 + 2 + \cdots + n = \frac{n(n+1)}{2}.
$$

This is the **maximum possible rank sum**.

### 6.1 Three extreme cases

#### (1) When $\eta_0$ is the true median (ideal case under $H_0$)

If the data are **symmetric** around $\eta_0$,

- about half of the observations are larger than $\eta_0$,  
- the other half are smaller, and
- their ranks are also roughly balanced on both sides.

Then the rank sum on the “positive side” should be around half of the total:

$$
W^{+} \approx \frac{1}{2} \times \frac{n(n+1)}{2}
      = \frac{n(n+1)}{4}.
$$

So **about half of the maximum rank sum** is natural in this case.

---

#### (2) When $\eta_0$ is much larger than the true median

Imagine all observations lie to the **left** of $\eta_0$  
(i.e., we chose $\eta_0$ far too large).

Then $X_i - \eta_0 < 0$ for all $i$, so

- $\psi(d_i) = 0$ for all $i$

and

$$
W^{+} = 0.
$$

This is the **smallest possible value** of $W^+$.

---

#### (3) When $\eta_0$ is much smaller than the true median

Now imagine all observations lie to the **right** of $\eta_0$  
(i.e., we chose $\eta_0$ far too small).

Then $X_i - \eta_0 > 0$ for all $i$, so

- $\psi(d_i) = 1$ for all $i$

and

$$
W^{+} = 1 + 2 + \cdots + n = \frac{n(n+1)}{2}.
$$

This is the **largest possible value** of $W^+$.

---

### 6.2 Interpretation

Putting this together:

- If $W^+$ is **close to 0**, or  
- if $W^+$ is **close to $\dfrac{n(n+1)}{2}$** (its maximum),

then the data are heavily skewed to one side of $\eta_0$. We would say

> “It is hard to believe that $\eta_0$ is the true median”  
> → we tend to **reject $H_0$**.

On the other hand,

- if $W^+$ is **close to $\dfrac{n(n+1)}{4}$**,

the data look fairly symmetric around $\eta_0$, so we may say

> “$\eta_0$ is quite plausible as the true median”  
> → we **do not reject $H_0$** (we have evidence supporting it).

In practice we compute a p-value using the exact or approximate distribution of $W^+$,  
for example $P(W^{+} \ge w_{\text{obs}})$ in a two-sided test.

---

## 7. One-sentence summary

> **The Wilcoxon signed-rank test**  
> looks at the **rank sum $W^+$ of observations larger than the hypothesized median $\eta_0$**.  
> If $W^+$ is near the center of its range (≈ $n(n+1)/4$),  
> $\eta_0$ is consistent with being the true median;  
> if $W^+$ is near either extreme (0 or the maximum),  
> we conclude that $\eta_0$ is **not** the median.  
> It is thus a **nonparametric one-sample test for the median** that does not rely on normality.
