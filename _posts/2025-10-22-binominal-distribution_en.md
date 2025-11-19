---
title: Binomial Distribution
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251022
tags: [binomial distribution]
lang: en
math: true
---

# Discrete Random Variables and PMF/CDF – One-page Summary

- **Discrete random variable \(X\)**: maps outcomes of an experiment to **countable values** such as \(0, 1, 2, \dots\).  
- **PMF** \(p_X(k) = P(X = k)\): probability assigned to each value \(k\).  
  - Properties: \(0 \le p_X(k) \le 1\), \(\sum_k p_X(k) = 1\).
- **CDF** \(F(x) = P(X \le x)\): for discrete variables, this is a **step-shaped** cumulative function.

> **Practical tips**  
> - Always distinguish **PMF (model)** from **frequency table (sample)**.  
> - Use complements to compute cumulative probabilities quickly:  
>   \(P(X \ge a) = 1 - P(X \le a - 1)\).

---

## Small Example (PMF vs. Observed Frequencies)

Suppose we test an antihypertensive drug on 4 patients.  
The manufacturer provides an **expected PMF** for the possible response counts, and frequencies from 100 clinics show a **similar pattern**.  
This qualitative agreement suggests that the model is reasonable.

---

# Formulas for Mean, Variance, Moments, and CDF

- **Expectation (distribution mean)**  

  $$
  \mu = E[X] = \sum_k k \, p_X(k)
  $$

  The sample mean is an estimator of \(\mu\), and by the **law of large numbers (LLN)** it converges to \(\mu\).

- **Variance**

  $$
  \sigma^2 = \sum_k (k - \mu)^2 p_X(k)
            = E[X^2] - (E[X])^2
  $$

- **Moments**

  - \(m\)-th moment: \(E[X^m]\).  
  - **Central moments**: \(E[(X - \mu)^m]\). The first central moment is 0, and the standardized third central moment is a measure of **skewness**.

- **CDF (cumulative distribution function)**  

  $$
  F(x) = P(X \le x)
  $$

  For a discrete random variable, the CDF is a **step plot**, jumping at integer values.

---

# Combinatorics Refresher (Preparation for the Binomial)

- **Permutation**:  

  $$
  P(n, k) = \frac{n!}{(n - k)!}
  $$

  order matters.

- **Combination**:  

  $$
  \binom{n}{k} = \frac{n!}{k!(n - k)!}
  $$

  order does not matter.

**Class mini example**

- Selecting 3 students (same role) out of 10. Probability that a particular student is selected:

  $$
  \frac{\binom{9}{2}}{\binom{10}{3}}
  = \frac{36}{120}
  = 0.3.
  $$

- Assigning **three different roles** to 3 students out of 10.  
  The probability that a specific student is assigned the “presentation” role is \(1/10\).

---

# Binomial Distribution \(\mathrm{Binomial}(n, p)\)

## Definition and PMF

- Background: the binomial distribution models a sequence of trials with only two outcomes (for example, coin toss: success / failure). Under suitable conditions it can be **approximated by a normal distribution**, so it also serves as a bridge to the normal distribution.

- Idea: we perform \(n\) **independent** trials, each with success probability \(p\), and count how many times the event of interest occurs.

- Assumptions:  
  - \(n\) independent trials,  
  - constant success probability \(p\) on each trial,  
  - failure probability \(q = 1 - p\).

- **PMF**:

  $$
  P(X = k) = \binom{n}{k} p^k q^{\,n - k},
  \quad k = 0, 1, \dots, n.
  $$

- **Mean** \(E[X] = np\), **variance** \(\mathrm{Var}(X) = npq\).

> **Normalization check**  
> $$
> \sum_{k = 0}^{n} \binom{n}{k} p^k q^{n - k} = (p + q)^n = 1.
> $$

---

## Simulation Histograms (Empirical Probabilities)

### \(n = 10,\ p = 0.05\)

![n10p005]({{ "/assets/img/binom-n10-p005.png" | relative_url }})

### \(n = 10,\ p = 0.95\)

![n10p095]({{ "/assets/img/binom-n10-p095.png" | relative_url }})

### \(n = 10,\ p = 0.50\)

![n10p050]({{ "/assets/img/binom-n10-p050.png" | relative_url }})

---

## Example 1 – Sex at Birth

Suppose the probability of a male birth is \(p = 0.51\).  
What is the probability of having exactly 2 sons among 5 children?

$$
P(X = 2)
= \binom{5}{2} \, 0.51^2 \, 0.49^3
\approx 0.30.
$$

---

## Example 2 – Infant Bronchitis (“At Least 3 Cases?”)

Assume the national average probability of infant bronchitis is \(p = 0.05\).  
Consider 20 independent families; then

$$
X \sim \mathrm{Bin}(20, 0.05).
$$

The probability of observing **at least 3 cases** is

$$
\begin{aligned}
P(X \ge 3)
&= 1 - \big[ P(X = 0) + P(X = 1) + P(X = 2) \big] \\
&\approx 1 - \big(0.358 + 0.377 + 0.189\big) \\
&\approx \mathbf{0.077}.
\end{aligned}
$$

> **Interpretation**: the chance of seeing 3 or more cases purely by random variation is about **7.7%**.  
> Whether this is considered “unusually high” depends on the chosen significance level and the broader context (multiple comparisons, prior expectations, etc.).

---

# Key Points and Cheat Sheet

- Use **PMF / CDF** to model discrete probabilities and compare them to **sample frequencies**.  
- **Mean and variance** describe location and spread; sample statistics are their **estimators**.  
- **Moments and skewness** help characterize tail behavior and asymmetry.  
- The **binomial distribution** is the basic model for repeated “success / failure” experiments.  
- For cumulative binomial probabilities, use **complements** whenever convenient:  
  \(P(X \ge a) = 1 - P(X \le a - 1)\).
