---
title: Test Statistic
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251112
tags: [test statistic]
lang: en
math: true
---

## What Is a Test Statistic?

A **test statistic** is a numerical value calculated from sample data in order to decide whether to reject or not reject a statistical hypothesis.

- It is computed from **sample statistics** (sample means, variances, proportions, etc.).  
- Often it is a **transformation** or “second processing” of those basic sample statistics, designed so that its distribution under the null hypothesis is known (e.g., t, z, F, chi-square).

Formally:

> A test statistic is a function of the data used to summarize the evidence against the null hypothesis.

---

## T-value (t Test Statistic)

The **t-value** is a type of test statistic used mainly to compare **means**.

Typical use:

- To test the difference between **two sample means**.  
- Or to test whether a single sample mean differs from a hypothesized population mean.

When comparing two groups, we are interested in:

1. The **difference in sample means**, and  
2. The **uncertainty** (standard error) of that difference.

The t statistic combines these into a single number:

$$
t
= \frac{\overline{X}_1 - \overline{X}_2}{\,S_{\overline{X}_1 - \overline{X}_2}\,}
$$

where

- \(\overline{X}_1\), \(\overline{X}_2\): sample means of group 1 and group 2,  
- \(S_{\overline{X}_1 - \overline{X}_2}\): **standard error** of the difference in sample means.

Intuition:

- **Numerator**: how large is the observed difference in means?  
- **Denominator**: how much **random variation** do we expect in that difference?

A large absolute t-value means that the difference in means is **big relative to the noise**, suggesting stronger evidence against the null hypothesis (e.g., “the two population means are equal”).

---

## What Counts as a “Large” T-value?

The t-value by itself is not enough; we need to know its **sampling distribution** under the null hypothesis.  
Under typical assumptions (normality, equal variances, etc.), the t statistic follows a **t distribution** with some **degrees of freedom**.

For a **two-sided test** at significance level \(\alpha = 0.05\):

- We look at the upper **2.5%** and lower **2.5%** tails of the t distribution.  
- These cutoffs (critical values) define what is “sufficiently large” in magnitude.

In other words:

- If \(t\) falls in the **top 2.5%** or **bottom 2.5%** of the t distribution (i.e., \(|t|\) is greater than the critical value),  
- Then we reject the null hypothesis at the **5% significance level**.

So:

> “Sufficiently large t-value” = a t-value that lies in the rejection region (extreme tails) of the t distribution for the chosen significance level and degrees of freedom.
