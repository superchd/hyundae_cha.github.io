---
title: 시간복잡도
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251121
tags: [asymptotic notation, algorithm]
lang: en
math: true

---

---
title: Asymptotic Notation for Kids
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251121-en
tags: [algorithm, big-o, complexity]
lang: en
math: true
---

# Asymptotic Notation, Super Simple

“Asymptotic notation” sounds scary,  
but you can think of it as:

> **A way to compare how fast functions grow.**

---

## 1. First: Feel the “growth speed”

When the number \(n\) gets bigger, these functions grow like this:

- \(n\) : 1, 2, 3, 4, ... → **slow**
- \(n^2\) : 1, 4, 9, 16, ... → **faster**
- \(2^n\) : 2, 4, 8, 16, 32, 64, ... → **very fast**

Think of a race:

- \(n\) : **a person walking**
- \(n^2\) : **a person on a bicycle**
- \(2^n\) : **a sports car**

At the beginning they may look similar,  
but as they go far, the **sports car ( \(2^n\) ) wins by a lot.**

---

## 2. Big-O, Ω, Θ = “speed labels”

Asymptotic notation has three common symbols:

- \(O\) (Big-O)
- \(\Omega\) (Big-Omega)
- \(\Theta\) (Big-Theta)

They are **labels** that tell us  
“how fast this function grows.”

Let’s use this example:

\[
f(n) = n^3 + n^2 + n - 1
\]

Here, the strongest (fastest-growing) term is **\(n^3\)**.

---

### 2-1. Big-O: “No faster than this”

We want to say:

> “\(f(n)\) does **not** grow faster than \(n^3\).”

Then we write:

\[
f(n) = O(n^3)
\]

- Big-O is like a **ceiling above the function**.
- “This function grows **at most** this fast.”

---

### 2-2. Big-Ω: “At least this fast”

Now we say:

> “\(f(n)\) grows **at least** as fast as \(n\).”

Then we write:

\[
f(n) = \Omega(n)
\]

- Big-Ω is like a **floor under the function**.
- “This function grows **at least** this fast.”

---

### 2-3. Big-Θ: “About the same speed”

For our function \(f(n)\), the main term is \(n^3\).

So we say:

> “\(f(n)\) grows **about as fast as** \(n^3\).”

and write:

\[
f(n) = \Theta(n^3)
\]

- The ceiling from above is \(n^3\).  
- The floor from below is also \(n^3\).

So **\(n^3\) is the right speed label**.

---

## 3. Why do we ignore constants in front?

Look at these three:

- \(9n^3\)
- \(100n^3\)
- \(0.5n^3\)

All of them grow like **\(n^3\)**.

Think of:

- 200 km/h sports car vs. 220 km/h sports car.  
- When you compare them to a walker or a bicycle (\(n\), \(n^2\)),  
  **both are just “very fast cars.”**

So in asymptotic notation we write:

\[
9n^3 = O(n^3),\quad 100n^3 = O(n^3)
\]

The constants **9, 100, 0.5** are ignored.

---

## 4. Is \(n^{100}\) in \(O(2^n)\)?

Question:

> “Is \(n^{100}\) slower than or equal to \(2^n\) in growth?”

Answer: **Yes.**

- \(n^{100}\): still a **polynomial** (just many \(n\)'s multiplied)
- \(2^n\): an **exponential function** (doubles every step)

As \(n\) gets large:

- “doubling every step” grows much faster than  
- “multiplying more \(n\)’s together.”

So we can say:

\[
n^{100} = O(2^n)
\]

It’s like:

- “a super strong runner with 100 boosters” vs.  
- “a sports car that keeps doubling its speed.”

In the long run, the **sports car ( \(2^n\) ) always wins.**

---

## 5. One-page summary

- Asymptotic notation =  
  **“How fast does this function grow?”**

- **Big-O, \(O(\cdot)\)**  
  - “Grows **no faster than** this speed” (ceiling)

- **Big-Ω, \(\Omega(\cdot)\)**  
  - “Grows **at least as fast as** this speed” (floor)

- **Big-Θ, \(\Theta(\cdot)\)**  
  - “Grows **about the same speed as** this”

- Constants in front (like 9, 100) are ignored.

- Exponential \(2^n\) will eventually beat any polynomial  
  like \(n^{100}\), \(n^{1000}\), etc.

With this, you can start to read algorithm time complexity  
and get a feel for **which algorithms are fast or slow**!
