---
title: Correlation & Spearman
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251211
tags: [correlation, spearman, nonparametric]
lang: en
math: false

---

# Correlation & Spearman Rank Correlation 


## 1. Rough guideline for correlation magnitude

Here, r or ρ means “some correlation coefficient”.

- |r| < 0.1  → almost none / very weak  
- |r| ≈ 0.3 → weak to modest  
- |r| ≈ 0.5 → clearly noticeable correlation  
- |r| ≥ 0.7 → fairly strong correlation  
- |r| ≥ 0.9 → very strong correlation  

These cutoffs are only rules of thumb; different books or fields may use slightly different labels.

---

## 2. What is Spearman rank correlation?

### 2.1 Basic idea

Spearman rank correlation (usually denoted ρ or ρ_s) measures the association between **ranks**, not the raw values.

Steps (conceptual):

1. Sort the X values from smallest to largest and assign ranks: 1, 2, 3, …  
2. Do the same for Y values and assign ranks.  
3. Compute the ordinary Pearson correlation between the two sets of ranks.  
4. That Pearson correlation of the ranks is the Spearman ρ_s.

Advantages:

- Much less sensitive to outliers.  
- More robust when the distribution is skewed.  
- Great when you care about **ordering** (“who is higher/lower”) rather than exact numeric values.

---

## 3. Spearman correlations in VALID.DAT (Exercises 11.72–11.75)

Data: VALID.DAT, n = 173 subjects.

- DR (diet record): short-term record of what was actually eaten.  
- FFQ (food frequency questionnaire): questionnaire-based estimate of usual intake.

We compare DR vs FFQ using **Spearman rank correlation** to see how well  
they agree on “who eats more” vs “who eats less” of each nutrient.

---

### 3.1 11.72 Alcohol intake (alco_dr vs alco_ffq)

- X = alco_dr (alcohol intake from DR)  
- Y = alco_ffq (alcohol intake from FFQ)

Spearman correlation:

- ρ_s ≈ 0.899

Significance:

- Corresponding t statistic ≈ 26.8  
- Degrees of freedom df = 171  
- p-value ≈ 4.3 × 10^(-63) (essentially 0)

Interpretation:

- ρ_s ≈ 0.90 → **very strong positive rank correlation**.  
- Subjects who drink a lot according to the DR almost always rank high according to the FFQ as well.  
- For alcohol, DR and FFQ agree on the **ranking** of subjects almost perfectly.

---

### 3.2 11.73 Total fat intake (tfat_dr vs tfat_ffq)

- X = tfat_dr (total fat from DR)  
- Y = tfat_ffq (total fat from FFQ)

Spearman correlation:

- ρ_s ≈ 0.371

Significance:

- t ≈ 5.22, df = 171  
- p ≈ 5.1 × 10^(-7) (p < 0.0001)

Interpretation:

- ρ_s ≈ 0.37 → **weak to modest positive correlation**.  
- Statistically highly significant (p < 0.0001).  
- DR and FFQ are reasonably consistent about “high-fat eaters vs low-fat eaters,”  
  but far from perfect, and clearly weaker than alcohol (ρ_s ≈ 0.90).

---

### 3.3 11.74 Saturated fat intake (sfat_dr vs sfat_ffq)

- X = sfat_dr (saturated fat from DR)  
- Y = sfat_ffq (saturated fat from FFQ)

Spearman correlation:

- ρ_s ≈ 0.422

Significance:

- t ≈ 6.09, df = 171  
- p ≈ 7.3 × 10^(-9) (p < 0.0001)

Interpretation:

- ρ_s ≈ 0.42 → a **bit stronger** than for total fat.  
- DR and FFQ show a reasonably good agreement on who eats more saturated fat.  
- Again, statistically very significant.

---

### 3.4 11.75 Total calorie intake (cal_dr vs cal_ffq)

- X = cal_dr (total calories from DR)  
- Y = cal_ffq (total calories from FFQ)

Spearman correlation:

- ρ_s ≈ 0.340

Significance:

- t ≈ 4.72, df = 171  
- p ≈ 4.8 × 10^(-6) (p < 0.0001)

Interpretation:

- ρ_s ≈ 0.34 → **weak to modest positive correlation**.  
- DR and FFQ do a decent job at distinguishing “high-calorie eaters” from “low-calorie eaters,”  
  but the agreement is not very strong.  
- Definitely weaker than alcohol (ρ_s ≈ 0.90).

---

## 4. Are ρ_s = 0.34, 0.37, 0.42 “real” correlations?

### 4.1 Effect size view

Using the rough thresholds above:

- ρ_s ≈ 0.34 (calories)  
- ρ_s ≈ 0.37 (total fat)  
- ρ_s ≈ 0.42 (saturated fat)

All of these fall in the **weak to modest** range.

So:

- The measurements are **not useless**.  
- They provide **some** ordering information about who eats more or less.  
- But they are **far from very strong** or perfect agreement (especially compared with alcohol ρ_s ≈ 0.90).

Typical wording in a paper:

- “There was a modest positive correlation between DR and FFQ for total fat intake (ρ_s ≈ 0.37).”

---

### 4.2 Statistical significance vs size of correlation

Important distinction:

- **p-value** answers:  
  “Is the true correlation exactly 0 or not?”

- **Correlation magnitude** (e.g., 0.34, 0.37, 0.42) answers:  
  “How strong is the relationship in practice?”

In these problems:

- p-values are extremely small (all < 0.0001) →  
  **we can confidently say the true correlation is not 0.**

- But the effect sizes (around 0.3–0.4) are **modest** →  
  **the relationship is real, but not very strong.**

---

## 5. How do we get p-values from ρ_s?

All p-values above are calculated **from the sample Spearman correlation ρ_s**.

### 5.1 Testing structure

- Null hypothesis H0: ρ_s = 0 (no rank correlation in the population)  
- Alternative hypothesis H1: ρ_s ≠ 0

We first compute ρ_s from the data (say 0.37), then ask:

> “If the true ρ_s were 0,  
> how likely is it to observe a sample correlation as extreme as 0.37 (or more)  
> just by random chance?”

That probability is the **p-value**.

### 5.2 Practical calculation (large n, t-approximation)

When sample size n is large (here n = 173), we can approximate:

1. Compute ρ_s from the data.  

2. Convert ρ_s to a t statistic:

   t = ρ_s * sqrt( (n - 2) / (1 - ρ_s^2) )

   with degrees of freedom df = n - 2.

3. Use the t distribution with df = n - 2 to find the two-sided p-value.

Example (total fat, 11.73):

- ρ_s ≈ 0.371, n = 173  
- t ≈ 0.371 * sqrt( 171 / (1 - 0.371^2) ) ≈ 5.22  
- For df = 171, t = 5.22 gives p ≈ 5.1 × 10^(-7).

This p is tiny, so we reject H0: ρ_s = 0.

### 5.3 One-line summary

To get the p-value, we go:

> data → ranks → ρ_s → t statistic → t distribution → p-value.

So yes, the p-value is **based on the sample Spearman correlation**.

---

## 6. Exercise 11.76 – Parametric vs nonparametric (final verdict)

Characteristics of dietary intake data in VALID.DAT:

- Likely skewed and not normally distributed.  
- Contains zeros and potential outliers (especially for alcohol and some nutrients).  
- For validation, we mostly care about **rank ordering** of subjects (who eats more/less).

Therefore:

- A **nonparametric** method like Spearman rank correlation is more appropriate and robust.  
- It does not require normality and is less affected by extreme values.  

In practice, a good strategy is:

- Use Spearman to assess rank agreement,  
- Optionally apply transformations (e.g., log) and also report Pearson correlations,  
- Compare both to get a fuller picture.

