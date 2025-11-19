---
title: FMA 점수 예측 모델
sidebar:
  nav: docs-ko
aside:
  toc: true
key: 20251112
tags: FMA, AI, XGBoost, LSTM
lang: ko
math: true
---


### 





## 링크



https://github.com/superchd/AI/blob/main/FMA_Prediction.ipynb





## 현재 진행 상황



7일, 30일, 한달의 FMA 점수로 1년 뒤를 예측하려고 함



다만, 결과값이 다음과 같이 만족스럽지 않음



평균값의 차이가 엄청남



![스크린샷 2025-11-12 175401]({{ "/assets/images/2025-11-12-FMA_Prediction/스크린샷 2025-11-12 175401.png" | relative_url }})





앞으로 해볼 시도는 



1. LSTM 사용
2. 30일 데이터로 3개월 뒤나 6개월 뒤 예측하는 것 
3. 안되면,..... 그냥 접어야 하나 싶다.





현재는 총점을 기준으로 예측을 하는데 



FMA 32개 항목에 대해서 모든 항목을 예측하고 차후에 총점으로 합하는 방법은 어떨까?



를 생각해보다가 현재의 데이터로는 6개월뒤 점수를 예측할 수 없다는 것을 깨달았다.....



극값을 제외하거나, 그룹을 3개로 나누거나(high, medium, low), 분류로 가기로 했다.

현재의 그룹은 correlation 값이 너무 낮기 때문에 











