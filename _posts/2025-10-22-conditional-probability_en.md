---
title: Conditional Probability & Bayes’ Theorem
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251022
tags: [conditional probability, Bayes' theorem]
lang: en
mathjax: true
---

> A quick, example-driven note on **Bayes’ theorem** that computes **prior/posterior** probabilities step by step.

## 1) Bayes’ Theorem

Bayes’ theorem:

$$
P(H\mid E) \;=\; \frac{P(E\mid H)\,P(H)}{P(E)} \tag{1}
$$

Here \(H\) is a **hypothesis** and \(E\) is the **evidence** (new information).  
- \(P(H)\): **prior** — belief *before* seeing the evidence  
- \(P(H\mid E)\): **posterior** — updated belief *after* seeing the evidence

Interpreting probability as a **degree of belief** is the **Bayesian** view (as opposed to the traditional **frequentist** view).

## 2) Why it matters (in one line)

Bayes’ theorem **updates belief** about \(H\) using new information \(E\).  
Frequentist workflows are more **deductive** (given a fixed model/space), while Bayesian workflows are **inductive**—they start with a prior and move closer to the truth by iteratively updating with data.

## 3) Quick glossary

- \(H\): Hypothesis — the claim/event of interest  
- \(E\): Evidence — newly observed information  
- \(P(H)\): prior, \(P(H\mid E)\): posterior

“Prior/Posterior” simply mean **before/after** observing the evidence.

## 4) Example 1 — Posterior after **one** positive test

- Prevalence (prior): \(P(H)=0.001\) (0.1%)
- Sensitivity: \(P(E\mid H)=0.99\)
- Specificity: \(P(E^c\mid H^c)=0.98 \Rightarrow P(E\mid H^c)=0.02\)

Using the law of total probability for \(P(E)\):

$$
P(H\mid E)
=\frac{P(E\mid H)P(H)}{P(E\mid H)P(H)+P(E\mid H^c)P(H^c)}
=\frac{0.99\times 0.001}{0.99\times 0.001+0.02\times 0.999}
\approx 0.047. \tag{2}
$$

> **Interpretation:** With one positive test only, the posterior is about **4.7%** (very low prevalence makes false positives matter a lot).

## 5) Example 2 — Posterior after **two** consecutive positives

Use the **posterior from Example 1 (0.047)** as the **new prior**, then update again:

- New \(P(H)=0.047\), still \(P(E\mid H)=0.99\), \(P(E\mid H^c)=0.02\)

$$
P(H\mid E)
=\frac{0.99\times 0.047}{0.99\times 0.047 + 0.02\times 0.953}
\approx 0.709. \tag{3}
$$

> **Interpretation:** Two positives in a row raise the posterior to **≈70.9%**—a classic illustration of Bayesian updating.

## 6) Practical notes

- \(P(A\mid B)=1-P(A^c\mid B)\) **does** hold, but \(P(A\mid B)\) is **not** related to \(1-P(A\mid B^c)\).  
- With very small priors (low prevalence), the **PPV** can be low even for accurate tests → consider repeat testing or combining with a more specific test.  
- Decision-making should also weigh **cost/utility** (costs, invasiveness, treatment benefits/risks), not just probabilities.

---

### References
- Source (in Korean): <https://angeloyeo.github.io/2020/01/09/Bayes_rule.html>

<!--
If your theme doesn’t auto-enable MathJax via front matter,
the script below will render LaTeX on GitHub Pages as well.
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
