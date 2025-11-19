---
title: Maximum Likelihood Estimation
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251103
tags: [maximum likelihood]
lang: en
math: true
---

## Introduction

Imagine a “magic” gum box that contains **infinitely many** pieces of gum.

- Each gum has a natural number written inside it: \(1, 2, 3, \dots\).  
- Every time you draw a piece, each number in the box is **equally likely**.  
- You and a goblin start a betting game.

However, the goblin chose a **maximum number** in advance. All numbers on the gums are in

$$
\{1, 2, \dots, M\}
$$

for some unknown integer \(M\). This unknown \(M\) is the **parameter** (the thing we want to guess).

> Goblin: “If you can guess my maximum number \(M\), you win the box of gum.”

Only the goblin knows the true value of \(M\). You are allowed to draw, say, three times and then must guess \(M\).

---

## Intuition from Simple Draws

Suppose your first draw is **10**.

- Would it make sense to say “The maximum is 5”?  
- No. If \(M = 5\), then numbers larger than 5 **cannot** appear at all.  
- Since you already saw 10, the probability that \(M = 5\) is effectively **0**.

Now suppose your second draw is **12**. So your observed values are:

- Observed sample: \(10, 12\).

If you now say “The maximum is 5,” that is obviously impossible.  
But what if you say “The maximum is 12000”?

- This sounds crazy, but it is still **more reasonable than 5**, because at least 12000 is **consistent** with observing 10 and 12.  
- The goblin’s maximum could be 12000, but then most numbers from 1 to 12000 are possible and you just happened to see 10 and 12.

Now imagine your three draws are:

- \(5, 10, 12\).

What if you guess:

- \(M = 14\)?

This sounds much more **plausible** than \(M = 5\) or \(M = 12000\).  

If the candidates are just \(\{5, 14, 12000\}\), your **instinct** is that 14 is the most reasonable guess, because:

- 5 is **impossible** (we already saw 10 and 12).  
- 12000 is possible but feels **too large** given how small the observed numbers are.  
- 14 is **just above** the maximum observed value 12, and “fits” the data in a natural way.

In many such problems, the **maximum observed value** is used as the estimate of the unknown maximum parameter \(M\). This is exactly the kind of reasoning behind **maximum likelihood estimation**.

---

## Intuitive Meaning of MLE

Maximum likelihood estimation (MLE) can be described informally as:

> “Given the observed data, choose the parameter value that makes those data **most likely**.”

More precisely, if the parameter is \(\theta\) and your data are \(x_1, \dots, x_n\), you can write the **likelihood function**:

$$
L(\theta)
= P_\theta(X_1 = x_1, \dots, X_n = x_n),
$$

which is the probability of seeing this particular dataset, **viewed as a function of \(\theta\)**.

The **maximum likelihood estimator (MLE)** is the value \(\hat{\theta}\) that **maximizes** this likelihood:

$$
\hat{\theta}_{\text{MLE}}
= \arg\max_\theta L(\theta).
$$

In the gum-box story:

- The parameter is \(\theta = M\).  
- The data are your draws (e.g., 5, 10, 12).  
- Among all possible values of \(M\), the value that gives the **highest probability** of seeing this sample is (informally) the **maximum observed number**.

---

## Intuitive Definition (Density / Distribution View)

You can also think of MLE as:

> Estimating the **probability distribution or density** that best explains the observed data, within a given model family.

Different types of data suggest different families of distributions:

- Continuous, symmetric data around a center → normal distribution.  
- Counts of events in a fixed time window → Poisson distribution.  
- Binary outcomes (success/failure) → Bernoulli / binomial distribution.

For example, if you are modeling the weight of mice:

- A normal model says: “Most mice have weights close to the mean, and the data are roughly symmetric around that mean.”  
- The parameters \(\mu\) and \(\sigma\) (mean and standard deviation) can be estimated by MLE.

The idea is:

1. Choose a **model family** (e.g., normal, Poisson, binomial).  
2. Write down the **likelihood** \(L(\theta)\) of the observed data under this model.  
3. Find the parameter value \(\hat{\theta}\) that **maximizes** this likelihood.  

That \(\hat{\theta}\) is the **maximum likelihood estimate**: the parameter value that makes your actual data look as “unsurprising” (i.e., as likely) as possible under the chosen model.
