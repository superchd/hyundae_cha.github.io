---
title: Probability
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251022
tags: [statistics, probability]
lang: en
math: true
---

# Probability

## 1. Variability and Stability Metrics

### Coefficient of Variation (CV)

$$
\mathrm{CV} = \frac{s}{\bar x}
$$

- Purpose: compare **relative variability** after normalizing for the mean level (reproducibility / stability).
- Invariant to scale changes \(x \to a x\) since \(s \to |a|s\) and \(\bar x \to a \bar x\), so the ratio does not change.

### Signal-to-Noise Ratio (SNR)

(Example: standardized difference of two means.)

$$
\mathrm{SNR}
= \frac{\bar x_1 - \bar x_2}{\sqrt{(s_1^2 + s_2^2)/2}}
$$

- Quantifies **separation / detectability** of two distributions.
- Will reappear later in the context of diagnostic tests and ROC curves.

---

## 2. Ways to Present Data

### Raw-data table

- Show all observations directly when the sample size is moderate.  
- Advantage: no information loss.

### Frequency table / Histogram

- Show counts (or cumulative counts) for identical values or within bins.  
- Bin count and boundaries can be **subjective**.  
- Structure **within each bin is lost**.

### Stem-and-leaf plot

- Each value is split into a **stem** (less varying digits) and a **leaf** (remaining digits).  
- Preserves the overall shape like a histogram while keeping (almost) all raw values.  
- A cumulative-count column helps locate the **median** quickly.

### Box-and-whisker plot

- Displays the median, lower and upper quartiles \((Q_1, Q_3)\), the interquartile range \(\mathrm{IQR} = Q_3 - Q_1\), and whiskers (range).  
- Skewness check: for positive skew, the upper whisker and upper quartile side tend to extend farther.

### Outlier rules (convention)

Typical outliers:

$$
[\, Q_1 - 1.5\,\mathrm{IQR}, \; Q_3 + 1.5\,\mathrm{IQR} \,]
$$

Extreme outliers:

$$
[\, Q_1 - 3\,\mathrm{IQR}, \; Q_3 + 3\,\mathrm{IQR} \,]
$$

The constants \(1.5\) and \(3\) are **tunable**. Always document your choice and rationale.

---

## 3. Why Move to Probability?

Inferential statistics often asks:

> What is the probability of observing data this extreme (or more) **if the null hypothesis is true**?

So we need basic probability (sample space, events, axioms, laws).  
This is the backbone that later leads to **hypothesis tests** and **ROC analysis**.

---

## 4. Events, Sample Space, and Set Operations

- **Sample space** \(\Omega\): the set of all possible outcomes.  
- **Event**: a subset of \(\Omega\).  
- **Certain event**: the whole sample space, probability \(1\).  
- **Null event**: impossible event, probability \(0\).  
- **Operations**:  
  - Union \(A \cup B\) (“A or B”),  
  - Intersection \(A \cap B\) (“A and B”),  
  - Complement \(A^c\).

- **Disjoint**: \(A \cap B = \varnothing\).

### De Morgan’s laws

$$
(A \cup B)^{c} = A^{c} \cap B^{c}, \qquad
(A \cap B)^{c} = A^{c} \cup B^{c}.
$$

Note: \(\cup\) and \(\cap\) are **set / event** operations. Do not write \(\cup, \cap\) between probability **numbers** such as \(P(A)\).

---

## 5. Frequentist Probability and Axioms

### Frequentist definition

$$
P(A) = \lim_{n \to \infty} \frac{\#A}{n},
$$

where \(\#A\) is the number of times event \(A\) occurs in \(n\) trials.

### Axioms

$$
0 \le P(A) \le 1, \qquad P(\Omega) = 1,
$$

and for **disjoint** events \(A\) and \(B\),

$$
P(A \cup B) = P(A) + P(B).
$$

If they are not disjoint:

$$
P(A \cup B) = P(A) + P(B) - P(A \cap B).
$$

### Empirical probability (estimate)

$$
\hat P(A) = \frac{\#A}{n}.
$$

As \(n \to \infty\), the **law of large numbers (LLN)** gives \(\hat P(A) \to P(A)\).

---

## 6. Example Probability Models

### Sum of two dice equals 7

Total outcomes:

$$
|\Omega| = 36.
$$

Number of pairs \((i, j)\) such that \(i + j = 7\) is 6, so

$$
P(\text{sum} = 7) = \frac{6}{36} = \frac{1}{6}.
$$

### Comparing group risks (sketch)

Group A: 40 cases in 10,000.  
Group B: 50 cases in 10,000.

Question: Is the difference due to chance or increased risk?  
Under the null (equal risks), we will evaluate the **probability of the observed difference**.

### Fairness check for a die

Even outcome observed 90 times in 200 rolls:

$$
\hat p = \frac{90}{200} = 0.45.
$$

Under fairness (\(p = 0.5\)), we will later test whether this deviation is **plausible** or evidence against fairness.

---

## 7. Independence, Addition, and Multiplication

### Core reminders

**Independence**

$$
P(A \cap B) = P(A)\,P(B).
$$

This is **not** the same as being disjoint.

**Addition rule**

$$
P(A \cup B) = P(A) + P(B) - P(A \cap B).
$$

**Multiplication rule for mutually independent events**

For events \(A_1, \dots, A_n\),

$$
P\!\Big( \bigcap_{i=1}^{n} A_i \Big)
= \prod_{i=1}^{n} P(A_i),
$$

provided **every subset** of the events also satisfies the product condition (mutual independence).

---

## 8. Worked Examples

### Example 1: At least one cancer case among three people

Individual risk \(p = 0.24\) (assume independence).

Use the complement trick:

$$
\begin{aligned}
P(\text{at least one}) 
&= 1 - P(\text{none}) \\
&= 1 - (1 - p)^3 \\
&= 1 - 0.76^3 \approx 1 - 0.438 \approx 0.562.
\end{aligned}
$$

So there is about a 56% chance that at least one of the three has cancer.

---

### Example 2: Overbooked seats

100 seats, 105 passengers. Each passenger gets **exactly one seat** with equal chance.

For a fixed passenger, consider the event “gets some seat”:

$$
A = \bigcup_{i=1}^{100} A_i,
$$

where \(A_i\) is “gets seat \(i\)”.  
Each \(A_i\) has probability \(1/105\), and the \(A_i\) are disjoint, so

$$
P(A) = \sum_{i=1}^{100} P(A_i) = \frac{100}{105} = \frac{20}{21}.
$$

Thus

$$
P(\text{no seat}) = 1 - \frac{20}{21} = \frac{1}{21}.
$$

---

### Example 3: Extension of overbooking

Find \(n\) so that the probability of **at least one missed seat** over \(n\) flights is 50%.

Per flight seat probability \(q = 20/21\) (assume independence between flights).

Probability that the passenger gets a seat on **all** \(n\) flights:

$$
P(\text{seat on all flights}) = q^{n}.
$$

We want

$$
1 - q^{n} = 0.5
\quad \Rightarrow \quad
q^{n} = 0.5
\quad \Rightarrow \quad
n = \frac{\ln 0.5}{\ln(20/21)} \approx 14.2.
$$

- For \(n = 14\): \(P(\text{at least one miss}) \approx 0.495\).  
- For \(n = 15\): \(P(\text{at least one miss}) \approx 0.519\).  

So with 15 flights, the chance of missing at least one seat is just over 50%.

---

## 9. Marginal Probability and Partitions

If \(\{B_1, \dots, B_k\}\) are disjoint and form a partition of the sample space, then

$$
P(A) = \sum_{j=1}^{k} P(A \cap B_j)
$$

(law of total probability).

Equivalently,

$$
P(A) = \sum_{j=1}^{k} P(A \mid B_j)\,P(B_j).
$$

Example: for a test \(A^+\) and a second test with outcomes \(B^+\) and \(B^-\),

$$
P(A^+) = P(A^+ \cap B^+) + P(A^+ \cap B^-).
$$

---

## 10. Quantifying Dependence: Relative Risk (RR)

Definition (“risk of \(B\) depending on \(A\)”):

$$
RR = \frac{P(B \mid A)}{P(B \mid A^c)}.
$$

- Independence gives \(RR = 1\).  
- \(RR \neq 1\) indicates **dependence**.

### Example: Family flu

Suppose

$$
P(A_2 \mid A_1) = 0.20, \qquad
P(A_2 \mid A_1^c) \approx 0.089.
$$

Then

$$
RR \approx \frac{0.20}{0.089} \approx 2.2,
$$

so the second person’s risk is a little more than twice as high if the first person is sick.

---

## 11. Law of Total Probability: Vaccine Example

Law of total probability:

$$
P(A) = \sum_i P(A \mid B_i)\,P(B_i),
$$

where the \(B_i\) form a partition.

Example: mixed vaccine quality:

- 90% of doses are **dead** vaccine, 10% are **live**.  
- \(P(\text{disease} \mid \text{dead}) = 0.05\).  
- \(P(\text{disease} \mid \text{live}) = 0.5\).

Then

$$
P(\text{disease})
= 0.05 \times 0.9 + 0.5 \times 0.1
= 0.095.
$$

If the disease rate in unvaccinated people is 10%, then vaccination slightly lowers the average risk to 9.5%.

---

## 12. One-line Summary

We introduced basic probability (events, axioms, addition and product rules, independence) and linked it to real examples (dice, overbooking, vaccines).  
These tools are the foundation for **hypothesis testing**, **risk comparison**, and **ROC analysis** later in the course.
