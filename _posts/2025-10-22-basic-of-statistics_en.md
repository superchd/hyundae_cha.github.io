---
title: Basic of Statistics
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251022_en
tags: Statistics
lang: en
---

# Lecture 1(What is statistics)

## Purpose & Scope of Statistics

- Statistics is the science of making inferences about a population from sample data.
- This course focuses on probability and core statistical methods.
- Methods under Gaussian assumptions are the baseline, with brief pointers to nonparametric approaches when the underlying distribution is unknown.

## Descriptive vs Inferential vs Statistical Computing

- **Descriptive statistics:** Summarize and communicate structure in data using means, standard deviations, ranges, and clear graphs or tables.
- **Inferential statistics:** Draw population-level conclusions from samples via hypothesis tests, confidence intervals, and related procedures.
- **Statistical computing:** Apply computation grounded in probability and statistics, e.g., Monte Carlo simulation and image reconstruction, to solve practical problems.

## Case Study 1: Comparing Blood Pressure Devices (Rosner Ch.1)

- Question: Are pharmacy-style automated BP machines comparable to technician-operated manual cuffs.
- Design considerations: measurement order effects, subject characteristics (sex, age, weight, hypertension history), de-identification, and outlier checks.
- Finding at one site (Location C): mean machine–manual difference ≈ 14 mmHg.
- Null hypothesis: the population mean difference is 0.
- Evaluate how unlikely the observed difference is under an appropriate probability model (e.g., Gaussian) to assess evidence against the null.

## Case Study 2: Descriptive Visualization Examples (Rosner Ch.2)

- **Vitamin A intake vs cancer status:** Binned histograms/bar charts comparing distributions between cases and controls, with high-intake bins rarer among cancer cases.
- **CO exposure over time:** Scatter or time-series plots for nonsmokers vs passive smokers showing similar early-morning levels, higher midday exposure for passive smokers, and convergence again in the evening.

## Practical Takeaways

- Before modeling, use descriptive statistics and visualization to inspect structure, trends, and outliers.
- In study design, predefine measurement order, simultaneous-measurement feasibility, subject metadata, and data management/privacy.
- In hypothesis testing, interpret results as the probability of observing data as extreme as yours if the null hypothesis were true.





# Lecture 2(Practical Tips for Descriptive Statistics)

## Make self-contained graphics

- Write captions with **minimal context**: what/why, dataset, key takeaway.
- Label **axes with units**, define symbols/variables, and include a clear **legend**.

## Show only what’s needed

- Plot just enough to reveal **trends**; avoid clutter and decorative styling.

## Don’t trust software defaults

- Adjust **axis ranges/ratios/line styles** to match your goal.
- **ROC example:** both x (FPR) and y (TPR) are in $[0,1]$ → use **equal axis lengths** for faithful interpretation.

## File formats

- Prefer **vector** (EPS/PS/PDF/SVG) for publication—scales cleanly.
- Raster (JPEG/PNG) pixelates when enlarged.
- Screenshots → vector **won’t recover** lost quality.

------

# Notation & Measures of Location

- Sample:  

$$x_1,\ldots,x_n  (vector {x}).$$



- **Mean**: (과연 집단을 대표할 수 있는지)
  $$ \displaystyle \bar{x}=\frac{1}{n}\sum_{i=1}^n x_i $$
  

  - Linear/affine transform: if
    $$ y_i=ax_i+b, \bar{y}=a\bar{x}+b $$

  - **Sensitive to outliers.**

- **Median**: middle value after sorting (average the two middles if $n$ is even).

  - Transform: 

  - $$ y_i=ax_i+b \Rightarrow \operatorname{median}(y)=a\,\operatorname{median}(x)+b $$

    

  - **Robust** to outliers (requires sorting).

  - 

- **Mode**: most frequent value (useful for categorical data; limited as a location measure when values are widely dispersed).

- **Geometric mean**:
  $$ \operatorname{GM}=\exp\!\Big(\tfrac{1}{n}\sum_{i=1}^n \log x_i\Big) \;=\;\Big(\prod_{i=1}^n x_i\Big)^{1/n} $$

  - Good for **wide-scale** data (power-law/exponential, concentrations/exposures), and **log-axis** plots.
  - Consider as an alternative to the mean for **highly skewed** data.



## Mean vs Median (Assignment Focus)

- ==Median is preferred== when data are **skewed** or contain **outliers** (e.g., concentrations, income).
- ==Arithmetic Mean==: \(\bar{x}=\frac{1}{n}\sum x_i\) — **sensitive to outliers**.
- ==Median==: robust to outliers; good for ordinal data or censored measurements.

## Arithmetic vs Geometric Mean (Kidney Study)

- **Arithmetic Mean**: \(\bar{x}=\frac{1}{n}\sum x_i\)
- **Geometric Mean**: \(\mathrm{GM}=\exp\!\big(\frac{1}{n}\sum \log x_i\big)\)  ==(use when data are log-normal / multiplicative)==
- **Practice**: zeros → replace by LOD/2 or small \(\epsilon\) and **report your rule**.

> **Conclusion**: For right-skewed concentration data, ==Geometric Mean is the more appropriate location measure==.
>
> 

------

# Distribution Shape & Mean–Median

- **Symmetric** (e.g., bell-shaped): $\bar{x}\approx \text{median}$.
- **Right-skewed** (positive tail): $\bar{x}>\text{median}$.
- **Left-skewed** (negative tail): $\bar{x}<\text{median}$.

------

# Measures of Spread (Variation)

- **Range**: $\max-\min$. Very sensitive to sample size and outliers; hard to compare across different $n$.

- **Percentiles/Quantiles**: $p$-th percentile $v_p$ satisfies $p\%$ of data $\le v_p$.

  - Compute (after sorting): $k=n\cdot p/100$.
    - If $k$ is integer → average of the $k$-th and $(k{+}1)$-th values.
    - Else → **ceil** to $k'$ and take the $k'$-th value.
  - Use IQR (Q3–Q1) for a **robust spread** summary.

- **Sample variance/SD**:
  $$ s^2=\frac{1}{n-1}\sum_{i=1}^n (x_i-\bar{x})^2,\qquad s=\sqrt{s^2} $$

  - The $n{-}1$ term is the **degrees-of-freedom** correction (unbiased estimate).
  - Transform: $y_i=ax_i+b \Rightarrow s_y=|a|\,s_x$ (shift $b$ doesn’t affect spread).



# Why are there two formulas for standard deviation? (n vs. n−1)

## TL;DR

- **Population standard deviation (known population):**
  $$ \sigma=\sqrt{\frac{1}{N}\sum_{i=1}^{N}(x_i-\mu)^2} $$
  Use **denominator $N$**.

- **Estimating population variance/SD from a sample:**
  $$ s=\sqrt{\frac{1}{n-1}\sum_{i=1}^{n}(x_i-\bar x)^2} $$
  Use **denominator $n-1$** (Bessel’s correction).
   Reason: $\tfrac{1}{n}\sum (x_i-\bar x)^2$ **systematically underestimates** the population variance.

------

## Why two formulas? (intuition)

- **Sample mean is unbiased:**
   Even though $\bar x$ can be above or below $\mu$ in any sample, on average $\mathbb E[\bar x]=\mu$.

- **Sample variance (with $n$) is biased low:**
   Because we center at the **sample** mean $\bar x$ (spending one degree of freedom),
   $\tfrac{1}{n}\sum (x_i-\bar x)^2$ is, on average, **too small** → needs a correction.

- **Bessel’s correction:**
  $$ \mathbb E\!\left[\frac{1}{n-1}\sum_{i=1}^n (x_i-\bar x)^2\right]=\sigma^2 $$
  Using **$n-1$** makes the estimator **unbiased** for the population variance.

------

## Tiny example (feel the bias)

Population $\{1,2,3\}$ has SD $\approx 0.8165$.
 All size-2 samples: $\{1,2\}, \{1,3\}, \{2,3\}$.

- Using **$n$** in the variance for each sample gives $0.25, 1, 0.25$ → average is **too small**.
- Using **$n-1$** inflates those sample variances just enough to be **closer to the population variance**.

**Takeaway:** The smaller the sample, the stronger the underestimation → **$n-1$** matters more.

------

## When to use $n$ vs $n-1$

| Situation                                                    | Goal                                   | Denominator | Functions                                  |
| ------------------------------------------------------------ | -------------------------------------- | ----------- | ------------------------------------------ |
| You have the **entire population**                           | Describe that population’s variance/SD | **$N$**     | Excel `STDEV.P`; NumPy `np.std(x, ddof=0)` |
| You have a **sample** and want to **estimate the population** variance/SD | Remove bias (unbiased estimator)       | **$n-1$**   | Excel `STDEV.S`; NumPy `np.std(x, ddof=1)` |

> Practical tip: In reports, **state what you used** (denominator, function name, `ddof`).

------

## Sketch of the algebra

Definition:
$$ s^2=\frac{1}{n-1}\sum_{i=1}^n (x_i-\bar x)^2. $$
Expand:
$$ \sum (x_i-\bar x)^2=\sum x_i^2-n\bar x^2 =\sum x_i^2-\frac{1}{n}\Big(\sum x_i\Big)^2. $$
Take expectations (under i.i.d. sampling):
$$ \mathbb E\!\left[\sum (x_i-\bar x)^2\right]=(n-1)\sigma^2 \;\Rightarrow\; \mathbb E[s^2]=\sigma^2. $$
So **$n-1$** is the correction that yields an **unbiased** estimate.

- **Mean Absolute Deviation (MAD about the mean)**: $\displaystyle \frac{1}{n}\sum |x_i-\bar{x}|$.
  - Useful in **sparse/robust** settings (e.g., compressed sensing).
  - SD is tightly linked to the **Normal** model ($\mu,\sigma$ fully specify it); percentile–SD mappings under Normality come next chapter.

