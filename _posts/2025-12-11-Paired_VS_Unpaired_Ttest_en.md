---
title: Paired vs Unpaired Tests
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251211
tags: [t-test, paired, unpaired, nonparametric]
lang: en
math: false


---

# Paired vs Unpaired Tests 

In this note we

1. Explain **paired vs unpaired** data and tests in simple terms, and  
2. Solve the baby-weight extra credit problem using  
   - a paired t‑test  
   - an unpaired two-sample t‑test  
   - a nonparametric Wilcoxon rank-sum (Mann–Whitney) test.

All formulas are written in plain text so that nothing breaks on GitHub Pages.

---

## 1. Intuition: What Are Paired vs Unpaired Data?

### 1.1 Paired data

**Key idea: the two measurements belong together as a meaningful pair.**

Examples:

- Same person measured twice (before vs after a treatment)
  - Weight before vs weight after a diet pill  
- Same patient, two body parts
  - Left arm vs right arm strength  
- Matched pairs: two people are matched on age, risk level, etc.,  
  and then one is assigned to treatment and the other to control.

In a spreadsheet, paired data often look like this:

| Pair | Treatment | Control |
| ---- | --------- | ------- |
| 1    | T1        | C1      |
| 2    | T2        | C2      |
| ...  | ...       | ...     |

Each row is one **pair**, and the two numbers in that row belong together.

In this situation we usually analyze the **difference**:

- d = Treatment − Control (for each pair)

Then we test:

- H0: mean(d) = 0  
- H1: mean(d) ≠ 0 (or > 0, < 0 depending on the question)

This is the **paired t‑test**.  
Conceptually, it is just a **one-sample t‑test on the differences**.

**Why is this good?**

Because within each pair, many nuisance factors (age, genetics, baseline risk, etc.) are similar.  
When we look at the difference inside each pair, those shared effects largely cancel out,  
so we get:

- smaller noise  
- larger signal-to-noise ratio  
- often a **smaller p‑value** (higher power) than an unpaired analysis.

---

### 1.2 Unpaired (independent) data

**Key idea: the two groups consist of different individuals, with no natural pairing.**

Examples:

- 20 men vs 20 women, comparing height  
- Hospital A patients vs Hospital B patients  
- 15 treatment patients vs 15 control patients, but you do *not* use pair info

In a spreadsheet, this often looks like:

| ID   | Group | Weight |
| ---- | ----- | ------ |
| 1    | T     | ...    |
| 2    | T     | ...    |
| …    | C     | ...    |

There is no “row-wise” relationship between treatment and control values.

Here we use the **independent (unpaired) two-sample t‑test**:

- H0: mean_T = mean_C  
- H1: mean_T ≠ mean_C

This ignores any potential pairing and just compares group means.

---

### 1.3 One-line rule of thumb

- If each row has a **Treatment value and a Control value for the same pair/person**,  
  you have **paired data** → use paired methods.  

- If you only have two separate groups with no natural linking,  
  you have **unpaired (independent) data** → use unpaired methods.

In the extra credit problem, the women are enrolled in **pairs** (two similar patients at a time) and one of each pair is randomized to treatment and the other to control. So we *can* treat data as paired. But we can also ignore that structure and treat them as unpaired for comparison.

---

## 2. Extra Credit Problem – Data Setup

A trial studies 30 pregnant women:

- 15 in the **treatment** group (drug)  
- 15 in the **control** group (placebo)  

For each “patient #”, we have a treatment weight and a control weight (paired by enrollment).

### 2.1 Raw data

Treatment group (lb):

- 6.9, 7.6, 7.3, 7.6, 6.8, 7.2, 8.0, 5.5, 5.8, 7.3, 8.2, 6.9, 6.8, 5.7, 8.6

Control group (lb):

- 6.4, 6.7, 5.4, 8.2, 5.3, 6.6, 5.8, 5.7, 6.2, 7.1, 7.0, 6.9, 5.6, 4.2, 6.8

Basic summary:

- n1 = 15 (treatment), n2 = 15 (control)  
- treatment mean ≈ 7.08 lb  
- control mean ≈ 6.26 lb  
- mean difference (T − C) ≈ 0.82 lb

---

## 3. Part (a): Paired vs Unpaired Analyses

### 3.1 Paired t‑test

We treat each row as a matched pair and compute:

- d_i = Treatment_i − Control_i  for i = 1,…,15

Differences d:

- 0.5, 0.9, 1.9, −0.6, 1.5, 0.6, 2.2, −0.2, −0.4, 0.2, 1.2, 0.0, 1.2, 1.5, 1.8

Now compute:

- mean difference d_bar ≈ 0.82 lb  
- standard deviation of differences sd_d ≈ 0.887  
- standard error SE_d = sd_d / sqrt(15) ≈ 0.229  

t statistic:

- t = d_bar / SE_d ≈ 0.82 / 0.229 ≈ 3.58  
- degrees of freedom df = 15 − 1 = 14  

Two-sided p‑value:

- t = 3.58 with df = 14 → p ≈ 0.003 (well below 0.01)

95% confidence interval for the mean difference:

- roughly 0.33 lb to 1.31 lb

**Conclusion (paired analysis)**

There is strong evidence that the treatment group has heavier babies:

- mean difference ≈ 0.8 lb  
- p ≈ 0.003 → statistically significant

So, using paired t‑test, the drug seems to increase birth weight.

---

### 3.2 Unpaired (independent) two-sample t‑test

Now we **ignore pairing** and treat the two groups as independent samples.

From the raw data we obtain:

- treatment mean ≈ 7.08, sd ≈ 0.90  
- control mean ≈ 6.26, sd ≈ 0.96  

Pooled standard deviation:

- s_p ≈ 0.93  

Standard error of the difference in means:

- SE = s_p * sqrt(1/n1 + 1/n2)  
  ≈ 0.93 * sqrt(1/15 + 1/15)  
  ≈ 0.93 * sqrt(2/15)  
  ≈ 0.340  

t statistic:

- t = (7.08 − 6.26) / 0.340 ≈ 2.41  
- df = n1 + n2 − 2 = 28  

Two-sided p‑value:

- t = 2.41, df = 28 → p ≈ 0.02

95% confidence interval:

- roughly 0.12 lb to 1.52 lb

**Conclusion (unpaired analysis)**

Even ignoring pairing, the treatment group still has significantly higher baby weights:

- p ≈ 0.02 < 0.05 → statistically significant

---

### 3.3 Does the type of analysis change the assessment?

- **Paired t‑test:** p ≈ 0.003 → very strong evidence of an effect  
- **Unpaired t‑test:** p ≈ 0.02 → still significant, but weaker evidence

So:

- The **direction** and **basic conclusion** are the same: treatment > control.  
- However, the **strength of evidence** differs:
  - Paired analysis uses more information (within-pair structure),  
    reduces noise, and yields a smaller p‑value.

In other words, the choice of analysis **does not change the conclusion**,  
but the **paired analysis shows the effect more clearly**.

---

## 4. Part (b): Nonparametric Analysis (Assume Unpaired)

Now we reanalyze the same data using **nonparametric methods**,  
treating the samples as **unpaired**.

The natural choice is the **Wilcoxon rank-sum test (Mann–Whitney U test)**.

### 4.1 Conceptual steps

1. Combine all 30 baby weights (15 treatment + 15 control).
2. Rank them from smallest (rank 1) to largest (rank 30).  
   - If there are ties, assign average ranks.
3. Compute the sum of ranks for the treatment group, R_T.
4. Convert the rank sum to a U statistic and then to a z-score.  
   From this, obtain a p‑value.

### 4.2 Key numbers (from the rank calculations)

From the full rank calculation we obtain approximately:

- treatment rank sum R_T ≈ 290.5  
- control rank sum R_C ≈ 174.5  

With n1 = n2 = 15:

- U1 = R_T − n1(n1 + 1)/2  
  = 290.5 − (15*16)/2  
  = 290.5 − 120  
  = 170.5  

We can also define:

- U2 = n1*n2 − U1 = 225 − 170.5 = 54.5

Using normal approximation:

- E(U) = n1*n2/2 = 112.5  
- Var(U) = n1*n2*(n1 + n2 + 1)/12  
  = 15*15*31/12 ≈ 581.25  
- sd_U = sqrt(Var(U)) ≈ 24.11  

z statistic:

- z = (U1 − E(U)) / sd_U  
  ≈ (170.5 − 112.5) / 24.11  
  ≈ 2.41  

Two-sided p‑value:

- z ≈ 2.41 → p ≈ 0.016

**Conclusion (nonparametric, unpaired)**

The Wilcoxon rank-sum test also finds a statistically significant difference:

- p ≈ 0.016 < 0.05  
- Babies in the treatment group tend to be heavier than those in the control group.

So the nonparametric method agrees with the parametric methods on the direction and significance.

---

## 5. Part (c): Are Nonparametric Methods Preferable Here?

We now compare:

- **Parametric methods**
  - Paired t‑test  
  - Unpaired two-sample t‑test  

- **Nonparametric method**
  - Wilcoxon rank-sum (Mann–Whitney) test

### 5.1 Data characteristics

- Outcome: baby weight (in pounds)
  - A typical continuous biological measurement  
  - Often reasonably close to a normal distribution in many populations  
- No extreme outliers are present in this small data set.  
- Sample size: 15 vs 15 (moderate, but not tiny)

These conditions are usually quite friendly for t‑tests:

- The normality assumption is not badly violated.  
- The t‑test is fairly robust to mild departures from normality, especially when n ≥ 15 per group.

### 5.2 Which method is better here?

- The **paired t‑test** is the most **natural** analysis:
  - The subjects were enrolled in pairs and randomized within each pair.
  - The design is matched; the paired test corresponds directly to the design.
  - It uses within-pair differences and therefore has **higher power**.

- The **unpaired t‑test**  
  - Ignores the matching information.  
  - Still works reasonably well here but is less efficient than the paired test.

- The **Wilcoxon rank-sum test**  
  - Does not assume normality and is more robust to heavy tails and outliers.  
  - However, when the normality assumption is roughly OK,   
    it can be slightly less powerful than the t‑test.  

In this particular data set:

- All three methods agree: treatment group babies are heavier, with p around 0.003–0.02.  
- There is no strong evidence of serious non-normality or extreme outliers.

**Therefore:**

- In this context, the **parametric paired t‑test is preferable**.  
- The nonparametric method is useful as a **check** or if we suspect strong non-normality,  
  but it is not strictly necessary here and does not change the conclusion.

A concise exam-style answer for part (c) could be:

> The outcome (birth weight) is a roughly continuous, approximately normal variable with no extreme outliers, and the sample size is moderate. Under these conditions, parametric t‑tests are appropriate and more efficient, especially the paired t‑test which matches the study design. Nonparametric methods are not required here and would generally be less powerful, so they are not preferable as the primary analysis.

---

## 6. Final Summary

- **Paired vs unpaired**
  - Paired: each row contains a Treatment value and a Control value for the same pair/person.  
  - Unpaired: two separate groups with no natural pairing.

- **In the baby weight problem**
  - Paired t‑test: p ≈ 0.003 → strong evidence the drug increases baby weight.  
  - Unpaired t‑test: p ≈ 0.02 → still significant, but weaker evidence.  
  - Wilcoxon rank-sum: p ≈ 0.016 → nonparametric method agrees.

- **Best choice here**
  - The **paired t‑test** is most appropriate and powerful,  
    while the nonparametric method is a reasonable secondary check but not superior in this setting.