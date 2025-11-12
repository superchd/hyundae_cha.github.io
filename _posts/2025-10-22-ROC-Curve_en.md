---
title: ROC Curve
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251022
tags: classification, ROC Curve
lang: en
---

# Binary Diagnostic Tests: A Friendly Cheat Sheet

## 3.1 One-page Concepts

- **Sensitivity (Se = TPR)**: ability to correctly flag **patients** as positive
- **Specificity (Sp = TNR)**: ability to correctly flag **healthy** people as negative
- **PPV**: if the test is positive, the chance the person **truly has** the disease
- **NPV**: if the test is negative, the chance the person is **truly healthy**
- **Prevalence**: fraction of patients in **your population** (a prior)

> **Memory TIP**
> - **Se**: “don’t miss cases” (minimize false **negatives**)
> - **Sp**: “avoid false alarms” (minimize false **positives**)
> - **PPV/NPV** depend **heavily on prevalence** (Bayes!)

---

## 2×2 Table (Confusion Matrix)

|              | True Disease \(D\) | True No Disease \(D^{c}\) |
|---|---:|---:|
| **Test Positive \(+\)** | **TP = \(x_1\)** | **FP = \(x_2\)** |
| **Test Negative \(-\)** | **FN = \(x_3\)** | **TN = \(x_4\)** |

- **Sensitivity (Se, TPR)**: \(\displaystyle \frac{x_1}{x_1+x_3}=P(+\mid D)\)
- **Specificity (Sp, TNR)**: \(\displaystyle \frac{x_4}{x_2+x_4}=P(-\mid D^{c})\)
- **PPV**: \(\displaystyle \frac{x_1}{x_1+x_2}=P(D\mid +)\)
- **NPV**: \(\displaystyle \frac{x_4}{x_3+x_4}=P(D^{c}\mid -)\)
- **Prevalence (Prev)**: \(\displaystyle P(D)\)

---

## Why are PPV/NPV sensitive to prevalence? (Bayesian intuition)

\[
PPV=\frac{Se\cdot Prev}{Se\cdot Prev+(1-Sp)(1-Prev)},\qquad
NPV=\frac{Sp(1-Prev)}{(1-Se)Prev+Sp(1-Prev)}.
\]

- With a **rare disease (Prev ↓)**, false positives dominate → **PPV drops**
- Conversely **NPV gets very high** (most people are healthy, so a negative is almost surely true)

**1-minute numeric feel**  
Prev \(=0.01\), Se \(=0.99\), Sp \(=0.98\)

\[
PPV \approx \frac{0.99\cdot 0.01}{0.99\cdot 0.01+0.02\cdot 0.99} \approx 0.33
\]
→ **Only ~1 in 3 positives are true patients**

\(NPV \approx 0.9999\) → **A negative is almost certainly healthy**

**Bigger picture (population 10,000)**  
- 100 patients, 9,900 healthy  
  - TP = 99, FN = 1 (Se 0.99)  
  - TN = 9,702, FP = 198 (Sp 0.98)
- **PPV** = \(99/(99+198) \approx 0.333\)  
- **NPV** = \(9702/(9702+1) \approx 0.9999\)

---

## Which metric when? (Practical cues)

- **Maximize Se** when missing a case is costly (cancer screening, infectious-disease triage)  
  → **Lower** the threshold (Se↑) → follow with a **high-Sp confirmatory** test
- **Maximize Sp** when false positives are costly (invasive confirm, expensive therapy)  
  → **Raise** the threshold (Sp↑)
- **PPV/NPV** are for patient-facing probabilities (“If positive, chance you truly have it is ~%”)  
  → **Recompute with your prevalence** via Bayes
- **Don’t use Accuracy alone**: with rare diseases, saying “all negative” looks accurate but is useless

---

# Thresholds and ROC (Receiver Operating Characteristic)

## 4.1 What’s a threshold?

Many tests/readers output a **score (continuous/ordinal)**.  
Classify as **positive** if score ≥ cutoff; otherwise **negative**.

- Threshold **↑** → **Sp↑, Se↓** (conservative: fewer false alarms, more misses)  
- Threshold **↓** → **Se↑, Sp↓** (aggressive: fewer misses, more false alarms)

> **Se and Sp trade off**. Fixing a single threshold can be misleading.

---

## ROC curve: a performance map over thresholds

- **x-axis**: FPR \(= 1 - Sp\) (false-positive rate)  
- **y-axis**: TPR \(= Se\) (true-positive rate)  
- Sweep thresholds, plot \((\mathrm{FPR}, \mathrm{TPR})\) → **ROC curve**

**AUC (area under curve)**  
- **1.0**: perfect separation  
- **0.5**: random guessing (diagonal)  
- **Interpretation**: probability a random patient scores higher than a random healthy subject

In practice: **threshold-agnostic summary**.  
Even with similar AUCs, choose by the **FPR/TPR region that matters** for your use case.

---

## Empirical ROC vs Parametric ROC

- **Empirical ROC**: compute (Se, Sp) at each threshold → **connect the dots**
- **Parametric ROC**: fit score distributions (e.g., normal for healthy/disease) → can provide **AUC uncertainty (CIs)**

---

## Field cheat sheet

1. **Define your goal**: minimize **misses (Se↑)** or **false alarms (Sp↑)**?
2. **Know your prevalence**: prior from literature/registry/local data.
3. **Re-compute PPV/NPV by Bayes** using your prevalence for patient counseling.
4. **Don’t fixate on one cutoff**: use ROC/AUC and set a threshold in your **operating region**.
5. **Two-stage strategy**: 1st screen (Se↑) → 2nd confirm (Sp↑).

---

## One-liner

- **Se/Sp**: intrinsic test skill (less affected by prevalence)  
- **PPV/NPV**: “probability it’s real” for patients (strongly prevalence-dependent)  
- **ROC/AUC**: fair, threshold-free comparison of overall performance

## Tips

### True Positive? False Positive?

On an ROC curve, **FPR** and **TPR** are plotted on the x and y axes.  
“**Positive**” means the **classifier/test said “yes”**.

- **True** vs **False** indicates whether that decision was **correct**.
  - **True Positive (TP)**: actually diseased and called positive.
  - **False Positive (FP)**: actually healthy but **incorrectly** called positive.

### Relationship between TPR and FPR

Consider the tester (a clinician or classifier):

- **Doctor A** calls almost **everyone positive** (very low threshold).  
  - **TPR high** (few missed cases) **and** **FPR high** (many false alarms).
- **Doctor B** calls almost **everyone negative** (very high threshold).  
  - **TPR low** and **FPR low** (misses many true cases, few false alarms).

Great tests aim for **high TPR with low FPR**.

---

### References
- Source: [https://angeloyeo.github.io/2020/08/05/ROC.html]

<!--
If your theme doesn't auto-enable MathJax, this snippet renders LaTeX.
Works on GitHub Pages (Jekyll).
-->
<script>
window.addEventListener('DOMContentLoaded', function () {
  if (!window.MathJax) {
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.id = 'MathJax-script';
    s.async = true;
    s.src = 'https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js';
    document.head.appendChild(s);
  }
});
</script>
