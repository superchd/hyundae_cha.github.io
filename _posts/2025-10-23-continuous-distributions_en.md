---
title: Continuous Random Variables
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251023
tags: [statistics]
lang: en
math: true
---

## From Discrete to Continuous: Concept Shift

- **Continuous random variable**: takes **uncountably many** values on \(\mathbb{R}\) (or an interval).  
- Instead of a **PMF** (discrete case), we use a **PDF** \(f_X(x)\) for continuous variables.

### PDF: Definition and Properties

- Interval probability:

  $$
  P(a \le X \le b) = \int_a^b f_X(x)\,dx
  $$

- **Nonnegativity**: \(f_X(x) \ge 0\).  

- **Normalization**:

  $$
  \int_{-\infty}^{\infty} f_X(x)\,dx = 1
  $$

- Point probability:

  $$
  P(X = a) = \int_a^a f_X(x)\,dx = 0
  $$

  For continuous variables, **single points have probability 0**.

### Multivariate PDF

- **Joint density** \(f_{X,Y}(x,y)\):

  $$
  P(a \le X \le b,\; c \le Y \le d)
  = \int_c^d \int_a^b f_{X,Y}(x,y)\,dx\,dy
  $$

- If \(X\) and \(Y\) are **independent**, then

  $$
  f_{X,Y}(x,y) = f_X(x)\,f_Y(y).
  $$

### Marginal and Conditional Density

- Marginal density of \(X\):

  $$
  f_X(x) = \int_{-\infty}^{\infty} f_{X,Y}(x,y)\,dy
  $$

- Conditional density of \(X\) given \(Y = y\):

  $$
  f_{X\mid Y}(x \mid y)
  = \frac{f_{X,Y}(x,y)}{f_Y(y)}
  $$

---

## Expectation and Variance (Continuous Case)

- Mean:

  $$
  \mu_X = E[X] = \int_{-\infty}^{\infty} x\,f_X(x)\,dx
  $$

- Variance:

  $$
  \sigma_X^{2}
  = \mathrm{Var}(X)
  = \int_{-\infty}^{\infty} (x - \mu_X)^2 f_X(x)\,dx
  = E[X^2] - \mu_X^2
  $$

---

## Example: Continuous Uniform Distribution on a Finite Interval

Let the interval be centered at \(a\) with width \(b\), so the support is  
\([\,a - \tfrac{b}{2},\; a + \tfrac{b}{2}\,]\).

- PDF:

  $$
  f_X(x)
  = \frac{1}{b}\,\mathbf{1}\!\left\{ a - \frac{b}{2} \le x \le a + \frac{b}{2} \right\}
  $$

- Mean and variance:

  $$
  \mu = a, \qquad \sigma^2 = \frac{b^2}{12}.
  $$

---

## Linear Combinations, Covariance, and Correlation

### Linear Combination

Let

$$
L = \sum_{i=1}^{n} c_i X_i.
$$

- Expectation:

  $$
  E[L] = \sum_{i=1}^{n} c_i\,E[X_i]
  $$

  (linearity of expectation; does **not** require independence).

- Variance:

  $$
  \mathrm{Var}(L)
  = \sum_{i=1}^{n} c_i^2\,\mathrm{Var}(X_i)
    + 2 \sum_{1 \le i < j \le n} c_i c_j\,\mathrm{Cov}(X_i, X_j)
  $$

  If the \(X_i\) are **mutually independent**, all covariance terms are 0, so

  $$
  \mathrm{Var}(L) = \sum_{i=1}^{n} c_i^2\,\mathrm{Var}(X_i).
  $$

### Covariance and Correlation Coefficient

- Covariance:

  $$
  \mathrm{Cov}(X, Y) = E[XY] - E[X]\,E[Y]
  $$

- Correlation coefficient:

  $$
  \rho_{XY}
  = \frac{\mathrm{Cov}(X, Y)}{\sigma_X \sigma_Y},
  \qquad -1 \le \rho_{XY} \le 1.
  $$

> **Important**: \(\mathrm{Cov}(X,Y) = 0\) (or \(\rho_{XY} = 0\)) does **not** in general imply independence.

#### Counterexample (Lecture-style Example)

Let \(X\) be a standardized continuous random variable (mean 0, variance 1) that is **symmetric** about 0, and define \(Y = X^2\) (a nonlinear function of \(X\)).

- We have \(E[X^3] = 0\) by symmetry.  
- One can show \(\mathrm{Cov}(X, Y) = 0\), but clearly \(Y\) is determined by \(X\), so they are **dependent**.

---

## Normal Distribution: Introduction

- Notation: \(X \sim \mathcal{N}(\mu, \sigma^{2})\).

- **PDF**:

  $$
  f(x)
  = \frac{1}{\sqrt{2\pi}\,\sigma}
    \exp\!\left( -\frac{(x - \mu)^2}{2\sigma^2} \right)
  $$

- Parameter effects:
  - Changing \(\mu\): **shifts** the center horizontally, shape unchanged.  
  - Increasing \(\sigma\): **spreads out** the distribution (wider and lower peak).

---

## Practical Notes

- For continuous distributions, **probability = area under the PDF**.  
  We compute probabilities via integrals; point probabilities are 0.

- **Independence check**:
  - In theory: look at the cross terms in the variance of a sum.  
  - With data: do **not** conclude independence from \(\rho \approx 0\) alone.

- **Standardization**:  

  $$
  Z = \frac{X - \mu}{\sigma} \sim \mathcal{N}(0, 1),
  $$

  which allows us to use standard normal tables or built-in functions.

---

## Normal Distribution \(\mathcal{N}(\mu, \sigma^2)\)

**PDF**

$$
f(x)
= \frac{1}{\sqrt{2\pi}\,\sigma}
  \exp\!\left( -\frac{(x - \mu)^2}{2\sigma^2} \right)
$$

- Increasing \(\mu\): **horizontal shift**.  
- Increasing \(\sigma\): **broader and lower** curve.

- Standard normal: \(Z \sim \mathcal{N}(0, 1)\).  
  CDF: \(\Phi(z) = P(Z \le z)\).

- **Symmetry**:

  $$
  \Phi(-z) = 1 - \Phi(z).
  $$

- **Interval probability**:

  $$
  P(a \le Z \le b)
  = \Phi(b) - \Phi(a).
  $$

**68–95–99 Rule (Approximate)**

- \(P(|X - \mu| \le 1\sigma) \approx 0.68\)  
- \(P(|X - \mu| \le 2\sigma) \approx 0.95\)  
- \(P(|X - \mu| \le 3\sigma) \approx 0.99\)

**Shape facts**

- **Inflection points** at \(x = \mu \pm \sigma\).  
- **FWHM** (full width at half maximum):

  $$
  \text{FWHM} \approx 2.35\,\sigma
  $$

  (useful approximation specifically for the normal distribution).

---

## Percentiles and Z-scores

- The \(p\)-th percentile \(z_p\) satisfies

  $$
  P(Z \le z_p) = p.
  $$

  Example: \(z_{0.75} \approx +0.675\), \(z_{0.25} \approx -0.675\).

**Standardization**

$$
Z = \frac{X - \mu}{\sigma}
\quad \Rightarrow \quad
P(a \le X \le b)
= \Phi\!\left( \frac{b - \mu}{\sigma} \right)
  - \Phi\!\left( \frac{a - \mu}{\sigma} \right).
$$

**Example – Exceeding an Upper Limit**

Let \(X \sim \mathcal{N}(8, 2^2)\). Then

$$
P(X > 12) = 1 - \Phi(2) \approx 0.023,
$$

so there is about a 2.3% chance.

---

## Continuity Correction

- When approximating a **discrete** distribution with a **continuous** one (e.g., binomial or Poisson with a normal), we use a **\(\pm 0.5\)** continuity correction at boundaries.  
- This also matters when a continuous variable is **rounded** to the nearest integer.

**Example – Intraocular Pressure (IOP)**

Let \(X \sim \mathcal{N}(16, 3^2)\). “Normal” range is 12–20 mmHg, but measurements are **rounded to integers**.

- Use corrected boundaries: \(11.5 \le X \le 20.5\).

  $$
  P(11.5 \le X \le 20.5)
  = \Phi(1.5) - \Phi(-1.5)
  \approx 0.866 \quad (86\text{–}87\%).
  $$

- Without continuity correction (using 12–20 directly), the probability is about **82%**.  
  So the correction can be important in practice.

---

## Normal Approximation to Binomial and Poisson

### Binomial \(\mathrm{Bin}(n, p)\)

- Conditions for a good normal approximation:
  - \(np(1 - p) \gtrsim 5\),  
  - \(p\) not extremely close to 0 or 1.

- Approximation:

  $$
  X \approx \mathcal{N}(np,\; np(1 - p)).
  $$

- Continuity correction:

  - \(P(X \le k) \approx P(Y \le k + 0.5)\)  
  - \(P(X \ge k) \approx P(Y \ge k - 0.5)\)  
  - \(P(X = k) \approx P(k - 0.5 \le Y \le k + 0.5)\)

  For boundary values 0 and \(n\), use \((-\infty, 0.5]\) and \([n - 0.5, \infty)\).

### Poisson \(\mathrm{Pois}(\mu)\)

- If \(\mu \gtrsim 10\), the distribution is fairly symmetric and the normal approximation is reasonable.

- Approximation:

  $$
  X \approx \mathcal{N}(\mu, \mu)
  $$

  with the same continuity correction ideas.

> **Warning**: When the distribution is highly skewed (small \(n\), extreme \(p\), or small \(\mu\)), the normal approximation can be poor.

---

## Practical Cheat Sheet

1. **Standardize first**: arbitrary \((\mu, \sigma)\) to \(Z\), then use \(\Phi\).  
2. **Use symmetry**: \(\Phi(-z) = 1 - \Phi(z)\).  
3. **Use 0.5 continuity correction** for discrete-to-continuous approximations and for rounded measurements.  
4. **Check conditions**:  
   - Binomial: \(np(1 - p) \ge 5\).  
   - Poisson: \(\mu \ge 10\).  
5. **Quick sanity check**: 68–95–99 rule and FWHM \(\approx 2.35\,\sigma\).

---

### Key Formulas

- \(Z = (X - \mu)/\sigma\)  
- \(P(a \le Z \le b) = \Phi(b) - \Phi(a)\)  
- \(\Phi(-z) = 1 - \Phi(z)\)  
- Binomial \(\to\) Normal:

  $$
  \mu = np, \quad \sigma^2 = np(1 - p)
  $$

  (plus continuity correction)

- Poisson \(\to\) Normal:

  $$
  \mu = \sigma^2 = \lambda
  $$

  (plus continuity correction)
