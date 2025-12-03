---
title: 조건부 확률과 베이즈 정리
sidebar:
  nav: docs-ko
aside:
  toc: true
key: 20251022
tags: 조건부 확률, 베이즈 정리
lang: ko
---



> 이 글은 베이즈 정리(Bayes’ theorem)를 **사전/사후 확률**과 함께 예제로 바로 계산해 보는 노트입니다.

## 1) 베이즈 정리 공식

베이즈 정리:

$$
P(H\mid E) \;=\; \frac{P(E\mid H)\,P(H)}{P(E)} \tag{1}
$$

여기서 \(H\)는 **가설(Hypothesis)**, \(E\)는 **증거(Evidence)** 입니다.  
- \(P(H)\): **사전 확률(prior)** — 증거를 보기 **전**의 신뢰도  
- \(P(H\mid E)\): **사후 확률(posterior)** — 증거를 본 **후**의 갱신된 신뢰도

확률을 “사건에 대한 **신뢰도**”로 보는 관점은 **베이지안(Bayesian)** 관점입니다(전통적인 **빈도주의(frequentist)** 관점과 대비).

## 2) 왜 중요한가(요지)

베이즈 정리는 “**새로운 정보** \(E\)를 바탕으로 가설 \(H\)에 대한 **신뢰도를 갱신**하는 방법”입니다.  
전통적 통계는 미리 정의된 확률공간/분포에 근거한 **연역적** 추론에 가깝고, 베이지안은 경험적 사전정보를 두고 **귀납적**으로 갱신해 진리에 가까워집니다.

## 3) 용어 빠르게 정리

- \(H\): Hypothesis — “어떤 사건이 발생했다”는 주장  
- \(E\): Evidence — “새로 관측된 정보”  
- \(P(H)\): 사전 확률, \(P(H\mid E)\): 사후 확률

사전/사후는 **증거 관측 전/후**를 뜻합니다.

## 4) 예제 1 — 1회 양성일 때의 사후 확률

- 유병률(사전확률): \(P(H)=0.001\) (0.1%)
- 민감도: \(P(E\mid H)=0.99\)
- 특이도: \(P(E^c\mid H^c)=0.98 \Rightarrow P(E\mid H^c)=0.02\)

전확률 \(P(E)\)를 분해하면:

$$
P(H\mid E)
=\frac{P(E\mid H)P(H)}{P(E\mid H)P(H)+P(E\mid H^c)P(H^c)}
=\frac{0.99\times 0.001}{0.99\times 0.001+0.02\times 0.999}
\approx 0.047. \tag{2}
$$

> **해석**: 1회 양성만으로는 사후확률이 약 **4.7%**에 그칩니다(유병률이 매우 낮아서 거짓양성의 영향이 큼).

## 5) 예제 2 — 연속 2회 양성일 때

예제 1의 **사후 확률 0.047**을 **새 prior**로 사용해 한 번 더 갱신:

- 새 \(P(H)=0.047\), 여전히 \(P(E\mid H)=0.99\), \(P(E\mid H^c)=0.02\)

$$
P(H\mid E)
=\frac{0.99\times 0.047}{0.99\times 0.047 + 0.02\times 0.953}
\approx 0.709. \tag{3}
$$

> **해석**: 2회 연속 양성이면 사후확률이 약 **70.9%**로 **크게 상승**합니다(베이즈 갱신의 전형적인 효과).

## 6) 실전 메모

- \(P(A\mid B)=1-P(A^c\mid B)\)는 **성립**, 그러나 \(P(A\mid B)\)와 \(1-P(A\mid B^c)\)는 **관계없음**.  
- 사전확률(유병률)이 작으면 **양성 예측도(PPV)**가 낮아질 수 있음 → 반복검사, 더 높은 특이도 검사 조합 고려.  
- 의사결정은 수치뿐 아니라 **비용/효용**(검사비용, 침습성, 치료 이득/위험)까지 함께 평가.



---

### References
- 원문/출처: [https://angeloyeo.github.io/2020/01/09/Bayes_rule.html]

<!--
만약 테마가 mathjax: true 를 자동 처리하지 않으면 아래 스크립트가 수식을 렌더링합니다.
GitHub Pages(Jekyll)에서도 작동합니다.
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
