---
title: Chi-square Test
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251103
tags: [chi-square]
lang: en
math: true
---

## Introduction

The chi-square test is a statistical test used for **categorical data**.

Example: in a survey about favorite fruits, responses fall into **categories** such as “apple,” “grape,” “banana,” etc.

- Categorical example:  
  Favorite fruit → apple ![example-apple]({{ "/assets/images/2025-11-03-chi-square/image-20251110113446789.png" | relative_url }}), grape, banana…

- Continuous example:  
  Height, weight → often analyzed with **t-tests**, **ANOVA**, etc.

---

## Why Do We Need the Chi-square Test?

If we want a **statistically meaningful** answer, we might ask:

> Is there a pattern in how people choose fruits?

For example:

> Do people in their 30s like apples more than people in their 20s?

From the raw survey counts alone, it is hard to tell whether any apparent pattern is **real** or just **random noise**.

This is where the **chi-square test** comes in.

In simple terms, the chi-square test is a tool to check **how different** the **observed data** are from what we would **expect** under some assumption.

Example:

- Suppose we expect apple, banana, and grape to be equally popular (1:1:1).  
- But in the actual survey, apples are far more popular than the others.

The chi-square test lets us **quantify** that difference and test whether it is statistically significant.

---

## Types of Chi-square Tests

### 1. Goodness-of-fit Test

Checks whether the data fit a **specified distribution**.

Example:

- Test whether fruit preferences follow the 1:1:1 ratio for apple, banana, and grape.

### 2. Test of Independence

Checks whether **two categorical variables** are **independent** or **associated**.

Example:

- Does “favorite fruit” depend on “age group”?

![contingency-table]({{ "/assets/images/2025-11-03-chi-square/image-20251110113521964.png" | relative_url }})

From this contingency table we can set up hypotheses.

- **Null hypothesis** \(H_0\): fruit preference does **not** differ by age group (fruit and age are independent).  
- **Alternative hypothesis** \(H_1\): fruit preference **does** differ by age group (fruit and age are associated).

---

## Observed Counts vs. Expected Counts

- **Observed counts**: the actual counts in each cell of the table (what we see in the survey).  
- **Expected counts**: what we would expect **before** seeing the data, **if** the null hypothesis (independence) is true.

Under the null hypothesis of independence:

- “Age” and “favorite fruit” are independent.  
- By the multiplication rule, each expected cell count can be computed as:

  $$
  E_{ij}
  = \frac{(\text{row total}_i) \times (\text{column total}_j)}{\text{grand total}}.
  $$

![expected-counts]({{ "/assets/images/2025-11-03-chi-square/image-20251110113736814.png" | relative_url }})

Once we have all observed counts \(O_{ij}\) and expected counts \(E_{ij}\), we can compute the **chi-square statistic**:

$$
\chi^2 = \sum_{i,j} \frac{(O_{ij} - E_{ij})^2}{E_{ij}}.
$$

### Degrees of Freedom

For an \(r \times c\) contingency table:

$$
\text{df} = (r - 1) \times (c - 1).
$$

---

## Chi-square Distribution and Critical Values

Given the chi-square statistic \(\chi^2\) and the degrees of freedom \(\text{df} = k\), we can:

- Use software (R, Python, calculator) to find the **p-value**, or  
- Use a **chi-square table** to find the **critical value** for a chosen significance level \(\alpha\).

![chi-square-table]({{ "/assets/images/2025-11-03-chi-square/image-20251110113805056.png" | relative_url }})

For those curious, the p-value for a chi-square statistic \(x\) with \(k\) degrees of freedom is given by the upper-tail integral:

$$
p
= \int_{x}^{\infty}
  \frac{1}{2^{k/2}\,\Gamma\!\left(\frac{k}{2}\right)}
  t^{\frac{k}{2}-1} e^{-t/2} \, dt.
$$

(In practice, we do **not** compute this integral by hand; we use tables or software.)

### Example of Decision

Suppose:

- \(\text{df} = 2\),  
- Significance level \(\alpha = 0.05\),  
- The chi-square table gives a critical value of **5.99**.

If our computed \(\chi^2 = 6.8\) is **greater** than 5.99, then:

- \(\chi^2 > \chi^2_{\text{critical}}\)  
- We **reject the null hypothesis**.

This means the observed pattern is unlikely to be due to chance alone, under the assumption of independence.

---

## What to Remember

- A chi-square test measures **how far** the observed data deviate from the **expected** counts under the null hypothesis.

- Use **chi-square tests** for **categorical data**:  
  - Goodness-of-fit: does the data follow a specified distribution?  
  - Independence: are two categorical variables independent?

- For **continuous data** (e.g., height, weight), we usually use other tests such as **t-tests** or **ANOVA** instead of chi-square.

Reference (Korean video):  
https://www.youtube.com/watch?v=lmNZr1EDyNA&t=60s
