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

## Variability & Stability Metrics

**Coefficient of Variation (CV)**  
\[
\mathrm{CV}=\frac{s}{\bar x}
\]
- Goal: compare **relative variability** after normalizing for mean level (reproducibility/stability).
- Invariant to scale changes \(x\to a x\) since \(s\to |a|s,\ \bar x\to a\bar x\Rightarrow\) the ratio is unchanged.

**Signal-to-Noise Ratio (SNR)**  
(example: standardized difference of two means)
\[
\mathrm{SNR}
=\frac{\bar x_1-\bar x_2}{\sqrt{(s_1^2+s_2^2)/2}}
\]
- Quantifies **separation/detectability** of two distributions. Reappears in Ch.3 (diagnostic tests/ROC).

---

## Ways to Present Data (Grouping/Summarizing)

**Raw-data table:** show directly when the sample size is moderate—no information loss.

**Frequency table / Histogram**  
- Show counts (or cumulative counts) for identical values or within bins.  
- Bin count/boundaries can be **subjective**; **within-bin structure is lost**.

**Stem-and-Leaf Plot**  
- Each value \(\to\) **stem** (less varying digits) + **leaf** (remaining digits).  
- Preserves the shape like a histogram while **retaining the raw values** (near lossless).  
- A cumulative-count column helps locate the **median** quickly.

**Box/Whisker Plot**  
- Median, lower/upper quartiles \((Q_1,Q_3)\), interquartile range \(\mathrm{IQR}=Q_3-Q_1\), whiskers (range).  
- **Skewness check**: with positive skew, the upper whisker/upper quartile side tends to extend further.

**Outlier Rules (convention)**  
\[
\text{Typical outliers: }\ [\,Q_1-1.5\,\mathrm{IQR},\ Q_3+1.5\,\mathrm{IQR}\,]
\]
\[
\text{Extreme outliers: }\ [\,Q_1-3\,\mathrm{IQR},\ Q_3+3\,\mathrm{IQR}\,]
\]
- The constants \(1.5, 3\) are **tunable**—document your choice and rationale.

---

## Why Move to **Probability**?

Inferential statistics asks: the **probability** of observing data this extreme (or more) **if the null is true**.  
→ Basic probability (events, sample space, axioms, laws) is the **backbone**.  
→ Leads into **diagnostic tests & ROC** (TPR/FPR–based performance).

---

## Sets/Events and Notation

- **Sample space** \(\Omega\): set of all possible outcomes.  
- **Event**: subset of \(\Omega\).  
- **Operations**: union \(A\cup B\) (“A or B”), intersection \(A\cap B\) (“A and B”), complement \(A^{c}\).  
- **Disjoint**: \(A\cap B=\varnothing\).  
- **De Morgan’s laws**
\[
(A\cup B)^{c}=A^{c}\cap B^{c},\qquad
(A\cap B)^{c}=A^{c}\cup B^{c}.
\]
> Note: \(\cup,\cap\) are **set/event** operations. Do not write \(\cup,\cap\) between probability **numbers** \(P(\cdot)\).

---

## Frequentist Probability & Axioms

**Frequentist definition**
\[
P(A)=\lim_{n\to\infty}\frac{\#A}{n}.
\]

**Axioms**
\[
0\le P(A)\le 1,\qquad P(\Omega)=1,
\]
\[
A,B\ \text{disjoint}\Rightarrow P(A\cup B)=P(A)+P(B).
\]
(If not disjoint: \(P(A\cup B)=P(A)+P(B)-P(A\cap B)\) — covered next.)

**Empirical probability (estimate)**
\[
\hat P(A)=\frac{\#A}{n}\quad(\text{as } n\to\infty,\ \hat P(A)\xrightarrow{\text{LLN}} P(A)).
\]

---

## Example Probability Models

**Sum of two dice equals 7**  
\[
|\Omega|=36,\quad \#\{(i,j):i+j=7\}=6\ \Rightarrow\ P=6/36=1/6.
\]

**Comparing group risks (sketch)**  
A: 40 cases in \(10{,}000\); B: 50 in \(10{,}000\).  
“Chance or increased risk?” \(\Rightarrow\) under the null (equal risks), evaluate the **probability of the observed difference**.

**Fairness check**  
Even outcome observed 90 times in 200 die rolls \(\Rightarrow \hat p=0.45\).  
Under fairness (\(p=0.5\)), is this deviation **plausible**? \(\Rightarrow\) later via hypothesis tests.

---

## One-liner Summary

Ch.2 closes with summary metrics (**CV/SNR**) and display tools (**histogram, stem-and-leaf, boxplot**).  
Next we cement the **language of probability** (events, axioms, models, empirical \(P\)) en route to **inference/tests & ROC**.



# Events · Sample Space · Addition Law · Independence

## Events, Sample Space, Venn diagrams

- **Event space (sample space)**: all possible outcomes.  
- **Event**: subset of the space.  
- **Certain event**: the whole space, probability \(=1\).  
- **Null event**: impossible, probability \(=0\).  
- **Symbols**: union \(\cup\), intersection \(\cap\), complement \(A'\).

**De Morgan’s laws**  
\[
(A\cup B)'=A'\cap B',\qquad (A\cap B)'=A'\cup B'.
\]

---

## Empirical Probability & Repeatability

- **Empirical probability**: relative frequency in repeated trials; as trials ↑, **LLN** ⇒ convergence to true \(P\).  
- **Caution**: perfect repetition is rare (subjects/environments drift).  
- **Subjective probability**: for rare/nonrepeatable events via belief/betting odds.

**Odds**  
\[
\text{odds for }A \;=\; P(A):P(A').
\]
**Even odds** \(\Leftrightarrow\) \(P(A)=P(A')=0.5\).

---

##  Addition (Sum) Rule

**General form**  
\[
P(A\cup B)=P(A)+P(B)-P(A\cap B).
\]
> Subtract the overlap \(P(A\cap B)\) to avoid double-counting.

- **Disjoint**: \(P(A\cap B)=0\Rightarrow P(A\cup B)=P(A)+P(B)\).  
- For \(3+\) events use **inclusion–exclusion**.

**Ex 1: Two-screen positive ⇒ referral**  
Given \(P(A^+),\,P(B^+),\,P(A^+\cap B^+)\),
\[
P(\text{referral})=P(A^+\cup B^+)=P(A^+)+P(B^+)-P(A^+\cap B^+).
\]

**Ex: Two children with flu**  
\(P(A_1)=P(A_2)=0.2,\; P(A_1\cap A_2)=0.1\).
\[
P(A_1\cup A_2)=0.2+0.2-0.1=0.3.
\]

---

## Independence vs. Disjointness

**Independence**  
\[
P(A\cap B)=P(A)\,P(B).
\]
> Knowing one does **not** change the probability of the other.

- Not the same as **disjoint**: disjoint events with positive probabilities are **not** independent.  
- Complements: \(A\) and \(A'\) are generally **dependent** (unless \(P(A)\in\{0,1\}\)).

**Checks**

- **Two screening tests**: if \(P(A^+\cap B^+)\ne P(A^+)P(B^+)\) ⇒ **dependent**.  
- **Family flu** (\(A_1\): mom, \(A_2\): dad) cohabitation ⇒ likely **dependent**.  
- **Dice**  
  - \(A\): even; \(B=\{1,2\}\), \(C=\{1,2,3\}\)  
  - \(P(A\cap B)=\tfrac{1}{6}=P(A)P(B)=\tfrac12\cdot\tfrac13\) ⇒ **A & B independent**  
  - \(P(A\cap C)=\tfrac{1}{6}\neq P(A)P(C)=\tfrac12\cdot\tfrac12\) ⇒ **A & C dependent**

**Conditional view**  
Knowing \(A\) (even) does **not** change \(P(B)=\tfrac{1}{3}\) ⇒ independent.

---

## Exam/Assignment Pointers

- Use Venn diagrams to visualize **overlap/disjoint/complements**.  
- Write the **addition rule** correctly and explain **why subtract the intersection**.  
- Test **independence** via numbers or **conditional probability** reasoning.  
- Practice converting empirical probabilities to **odds** (for/against, even).  
- For independent events, **multiply** to get the probability that all occur.

---

## One-liner Summary

We reviewed set/Venn operations and drilled the **addition rule** and **independence (product condition)**.  
Independence is not about overlap but about **\(P(A\cap B)=P(A)P(B)\)**.



# Probability: Independence / Addition / Multiplication & Examples

## Core Reminders

- **Independence**  
  \[
  P(A\cap B)=P(A)\,P(B)
  \]
  *Not* the same as disjointness.

- **Addition rule**  
  \[
  P(A\cup B)=P(A)+P(B)-P(A\cap B)
  \]

- **Multiplication (mutual independence of \(n\) events)**  
  \[
  P\!\Big(\bigcap_{i=1}^{n} A_i\Big)=\prod_{i=1}^{n} P(A_i)
  \]
  Mutual independence requires the product condition for **all** subsets.

---

## Example 1: **At least one** cancer case among 3 people

- Individual risk \(p=0.24\) (assume **independent**).
- Complement trick:
  \[
  P(\ge 1)=1-P(0)=1-(1-p)^3
  =1-0.76^3\approx 1-0.438=0.562.
  \]

---

## Example 2: **Overbooked seats**

- 100 seats, 105 passengers. Each passenger gets **exactly one seat** with equal chance.
- For a fixed passenger, event “gets **some** seat”:
  \[
  A=\bigcup_{i=1}^{100} A_i,
  \quad P(A_i)=\frac{1}{105}.
  \]
  The \(A_i\) are **disjoint**, so
  \[
  P(A)=\sum_{i=1}^{100}P(A_i)=\frac{100}{105}=\frac{20}{21}.
  \]
- “No seat”:
  \[
  P(A^{c})=1-\frac{20}{21}=\boxed{\frac{1}{21}}.
  \]

---

## Example 2 — Extension: find \(n\) so that **≥1 miss** over \(n\) flights has 50%

- Per flight seat probability \(q=\frac{20}{21}\) (assume independence).
- All flights boarded:
  \[
  P(\text{all})=q^{\,n}.
  \]
- **Even odds**:
  \[
  1-q^{\,n}=0.5\Rightarrow q^{\,n}=0.5
  \Rightarrow n=\frac{\ln 0.5}{\ln(20/21)}\approx 14.2.
  \]
- Integer choice:
  - \(n=14\Rightarrow P(\ge 1\text{ miss})\approx 0.495\)
  - \(n=15\Rightarrow \approx 0.519\) (**exceeds 50%**, conservative pick: 15)

---

## Marginal Probability & Partitions

- If \(\{B_1,\dots,B_k\}\) are **disjoint** and **partition** the space,
  \[
  P(A)=\sum_{j=1}^{k} P(A\cap B_j)\quad\text{(law of total probability)}.
  \]
- Example: screen \(A^+\) with \(B^+/B^-\)
  \[
  P(A^+)=P(A^+\cap B^+)+P(A^+\cap B^-).
  \]



## Quantifying Dependence: Relative Risk (RR)

- Definition (“risk of B depending on A”):
  \[
  RR=\frac{P(B\mid A)}{P(B\mid A^c)}
  \]
  - **Independence** ⇒ \(RR=1\)  
  - \(RR\neq 1\) ⇒ **dependence**

### Example: Family flu
Given
\[
P(A_2\mid A_1)=0.20,\quad P(A_2\mid A_1^c)\approx 0.089
\Rightarrow RR\approx \frac{0.20}{0.089}\approx 2.2,
\]
so dad’s risk is **~2×** higher if mom is sick.

---

##  Law of Total Probability (Partition Rule)

\[
P(A)=\sum_i P(A\mid B_i)\,P(B_i)
\]
(\(B_i\): disjoint & exhaustive)

### Example: Mixed vaccine quality
- Assume \(90\%\) **dead**, \(10\%\) **live**  
- \(P(\text{disease}\mid \text{dead})=0.05,\; P(\text{disease}\mid \text{live})=0.5\)

\[
P(\text{disease})=0.05\times 0.9 + 0.5\times 0.1 = 0.095
\]

If the unvaccinated group’s disease rate is \(10\%\), vaccination **slightly lowers** the average risk (to **9.5%**).  
Personal decisions can split into high/low risk \(HR, LR\):
\[
P(\text{disease})=P(\text{disease}\mid HR)P(HR)+P(\text{disease}\mid LR)P(LR),
\]
reflecting individual risk.

- **Prevalence**: fraction **currently** diseased at a time point.  
- **Incidence**: probability of **new** cases over a time window (cumulative vs. rate).
