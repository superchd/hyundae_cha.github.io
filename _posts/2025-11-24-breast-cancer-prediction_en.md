---
title: Breast cancer model predicition
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251124
tags: [multimodal, prediction, machine learning]
lang: en
math: true
---



### 전체적인 코드 그림 

**3가지 모달리티(임상 + CNA + Mutation) → 하나의 큰 feature matrix → 공통 전처리 → 여러 ML 모델 비교 → 베스트 모델 시각화**



**임상 데이터 + 10년 생존 라벨 생성**

- `clinical = pd.read_csv(..., sep="\t")`

- `Overall Survival (Months)`와 `Overall Survival Status`가 있는 환자만 필터

- `Overall Survival Status`에서 `"DECEASED"` 여부로 **사망 이벤트(event_death)** 계산

- **10년 이내 사망이면 1, 그 외(10년 넘게 생존 or 10년 안에 생존 상태)면 0** 으로 `OS10_event` 라벨 생성

- `Sample ID`를 index로 설정

- ID/생존 관련 컬럼들(Study ID, Patient ID, OS, RFS, Vital, OS10_event)을 제외하고 **순수 임상 feature(`clin_features`)만 분리**

  - ## brca_metabric_clinical_data.tsv

    **→ 환자 정보(임상 데이터) 엑셀 시트**

    - **행(row)**: 한 줄 = 한 명의 환자 / 샘플
    - **열(column)**:
      - 나이, 진단 시기, ER/PR/HER2, 서브타입, stage 같은 임상 변수
      - OS(months), OS_status 같은 생존 정보
    - 느낌:
       👉 “이 환자는 누구인가?” 에 대한 요약표

    이 파일에서 제일 중요한 건 나중에 **샘플 ID** (예: `MTS-T0058`, `MB-0000` 이런 것들)로 다른 파일과 매칭하는 것

    

**CNA 모달리티 전처리**

- `data_cna.txt` 읽어서 첫 컬럼을 `Hugo_Symbol`로 rename 후 gene index로 사용

- `X_cna = cna.T` 해서 **(샘플 × 유전자)** 형태로 전치

- 숫자로 변환 후 `float32`

- 분산이 0인 유전자(모든 샘플에서 값이 동일) 제거 → 정보 없는 feature drop

  - ## 이 파일은 “유전자 × 환자” 표

    위에 붙어 있는 엄청 긴 헤더는 사실 구조가 **딱 하나**야:

    ```
    Hugo_Symbol | Entrez_Gene_Id | MB-0000 | MB-0039 | MB-0045 | MB-0046 | ...
    ```

    - **열(column)** 의미

      - `Hugo_Symbol` : 유전자 이름 (A1BG, TP53, PIK3CA, …)
      - `Entrez_Gene_Id` : 그 유전자에 대한 숫자 ID(Entrez ID)
      - `MB-0000`, `MB-0039`, `MB-0045`, … : **각각 하나의 환자/샘플 ID**
        - clinical 파일의 `Sample ID`랑 1:1로 매칭되는 그 ID들

    - **행(row)** 의미

      - 한 줄 = **하나의 유전자**

      - 예를 들어:

        ```
        Hugo_Symbol  Entrez_Gene_Id  MB-0000  MB-0039  MB-0045  ...
        A1BG         1               0       0        -1      ...
        A1BG-AS1     503538          0       0        -1      ...
        A1CF         29974           0       0         0      ...
        ```

      - 이건 “A1BG라는 유전자에 대해, 각 환자에서 복제수 상태가 어떻게 되느냐”를 쭉 나열해 놓은 거야.

    ------

    ## 2. 0, -1, 1, 2, -2 값이 의미하는 것

    각 칸에 들어 있는 숫자들은 **복제수 변이(Copy Number Alteration)** 상태를 정리한 거야.

    보통 METABRIC/cBioPortal CNA 데이터는 이런 식으로 코딩돼 있어:

    - `0` : **neutral** (정상 copy number, 딥로이드)
    - `-1` : **single-copy loss** (한 개 어레이가 빠진 상태)
    - `-2` : **deep deletion** (더 심한 deletion, homozygous deletion 같은 것)
    - `1` : **low-level gain** (한 개 copy 증가)
    - `2` : **high-level amplification** (강한 증폭)

    그래서 예를 들어 이 줄을 보면:

    ```
    A1BG    1    0   0   -1   0   0   ...
    ```

    - `Hugo_Symbol = A1BG`
    - `Entrez_Gene_Id = 1`
    - `MB-0000` 환자에서 A1BG = 0  → 복제수 정상
    - `MB-0045` 환자에서 A1BG = -1 → 하나 copy가 빠진 loss
    - 어떤 환자에서 `2`면 → 그 유전자 locus가 amplification 되어 있다는 뜻

    ------

    ## 3. 이 파일이 clinical / mutation이랑 방향이 다른 이유

    지금 CNA 파일은 **형태가 이렇게 생김**:

    > **행 = 유전자, 열 = 환자**

    반대로 우리 ML 모델에 넣고 싶은 최종 설계행렬은:

    > **행 = 환자, 열 = feature들(임상, CNA, mutation)**

    그래서 노트북 코드에서 이런 짓을 했던 거야:

    ```
    cna = pd.read_csv("data_cna.txt", sep="\t")
    cna = cna.set_index("Hugo_Symbol")          # 유전자 이름을 index로
    cna = cna.drop(columns=["Entrez_Gene_Id"])  # 숫자 ID는 버리고
    X_cna = cna.T                               # ← 여기서 전치!
    ```

    - `X_cna = cna.T`를 하면 구조가 이렇게 뒤집힘:
      - **원래**
        - 행: gene
        - 열: sample (MB-0000, MB-0039, …)
      - **전치 후**
        - 행: sample (MB-0000, MB-0039, …)
        - 열: gene (A1BG, A1BG-AS1, A1CF, …)

    → 이렇게 해야 clinical (sample × clinical feature), mutation (sample × gene) 이랑 **행 방향이 맞아서** 나중에 `pd.concat([...], axis=1)`로 쉽게 붙일 수 있어.

**Mutation 모달리티 전처리**

- `data_mutations.txt` (MAF 형식) 읽기

- `Variant_Classification` 기준으로 Silent, UTR, Intron 등 **silent-like 변이 제거**

- `Tumor_Sample_Barcode` vs `Hugo_Symbol`로 피벗 →
   **샘플 × 유전자 binary matrix (해당 유전자에 non-silent mutation 있으면 1, 없으면 0) = `X_mut`**

- `Mutated_Genes.txt`가 있으면, 여기에 있는 gene panel 기준으로 필터링

- 마지막에 `X_cna`와 `X_mut`의 공통 유전자 교집합만 남겨서
   **두 모달리티가 동일한 gene set을 쓰도록 정렬**

  - ##  Mutated_Genes.txt

    **→ 관심 있는 유전자들 리스트 (필터용)**

    이건 보통:

    - 한 줄에 하나씩 유전자 이름이 들어 있는 경우가 많아
      - 예:
        - PIK3CA
        - TP53
        - GATA3
        - MAP3K1
        - CDH1 …
    - 용도:
      - `data_mutations.txt`에서 **이 리스트에 있는 유전자만** 골라서 서브셋 만들 때 사용
      - “driver gene만 보고 싶다” 이런 느낌의 필터셋

  

**세 모달리티(임상 + CNA + MUT) 통합**

- `common_ids = clin_features.index ∩ X_cna.index ∩ X_mut.index`
   → 세 모달리티 모두 데이터가 있는 샘플만 사용

- `clin_features`, `X_cna`, `X_mut`를 모두 `common_ids` 기준으로 subset

- 라벨 `y = clinical.loc[common_ids, "OS10_event"]`

- CNA 컬럼 이름 앞에 `"CNA_"`, Mutation 앞에 `"MUT_"` prefix 추가

- 최종 feature matrix:

  ```
  X_all = pd.concat([clin_features, X_cna_renamed, X_mut_renamed], axis=1)
  ```

  → **(샘플 × [임상 + CNA + MUT])** 통합 **멀티모달 feature matrix** 완성





**전처리 파이프라인(Preprocessor) 정의**

- `X_all` 컬럼들 중 숫자형 / 범주형 나누기

  - `numeric_cols` : 숫자형
  - `categorical_cols` : 나머지

- 숫자형 파이프라인:

  ```
  numeric_transformer = Pipeline([
      ("imputer", SimpleImputer(strategy="median")),
      ("scaler", StandardScaler()),
  ])
  ```

- 범주형 파이프라인:

  ```
  categorical_transformer = Pipeline([
      ("imputer", SimpleImputer(strategy="most_frequent")),
      ("onehot", OneHotEncoder(handle_unknown="ignore")),
  ])
  ```

- 이 둘을 `ColumnTransformer`로 합쳐서:

  ```
  preprocessor = ColumnTransformer([
      ("num", numeric_transformer, numeric_cols),
      ("cat", categorical_transformer, categorical_cols),
  ])
  ```

→ **여기까지가 “공통 전처리 파이프라인”**





1. **Train/Test Split + 공통 Pipeline으로 반복 학습 & 평가**

   - `train_test_split(X_all, y, test_size=0.2, stratify=y, random_state=42)`
      → 80% train, 20% test, stratified split

   - 5-fold StratifiedKFold 정의

   - **for 루프**:

     ```
     for name, base_clf in models.items():
         clf = Pipeline([
             ("preprocess", preprocessor),
             ("clf", base_clf),
         ])
     ```

     → **여기서 말하는 sklearn `Pipeline` = (전처리 + 분류기)** 를 합친 모델

   - 각 모델에 대해:

     1. `cross_val_score(..., scoring="roc_auc")`로 전체 데이터에 5-fold CV AUC 계산
     2. Train split에 `clf.fit(X_train, y_train)`
     3. Test split에 대해:
        - LinearSVM: `decision_function`
        - 나머지: `predict_proba(..., 1 클래스 확률)`
     4. threshold 0.5로 0/1 예측
     5. **metrics 계산:**
         AUC, Accuracy, F1, Precision, Recall(=Sensitivity), Specificity, Balanced Accuracy, classification_report
     6. 결과를 딕셔너리로 `results` 리스트에 append

   - 마지막에 `results_df = pd.DataFrame(results).sort_values("Test_AUC", ascending=False)`
      → **Test AUC 기준으로 모델 비교 테이블**

2. **베스트 모델 선택 + 시각화**

   - `best_name = results_df.iloc[0]["Model"]`

   - `best_clf = models[best_name]`

   - 다시 공통 파이프라인 묶어서:

     ```
     best_pipeline = Pipeline([
         ("preprocess", preprocessor),
         ("clf", best_clf),
     ])
     best_pipeline.fit(X_train, y_train)
     ```

   - test에서 best model로:

     - ROC curve 그리기
     - Confusion matrix 그리기
     - 예측 점수(score) 분포 히스토그램 (event=0 vs 1)
     - 모든 모델의 Test AUC bar chart





PPT 참조~

https://docs.google.com/presentation/d/1hBB9hKuHHTYjotL9pTy8lJVeNenyR4pFB2NLfYy1gcc/edit?slide=id.p12#slide=id.p12
