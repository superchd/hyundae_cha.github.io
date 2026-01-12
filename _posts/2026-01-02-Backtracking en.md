---
title: "K개 중 하나를 N번 선택하기 (Simple)"
sidebar:
  nav: docs-ko
aside:
  toc: true
key: 20260102
tags: [backtracking, recursion, brute-force, permutation-with-repetition]
lang: ko
math: true

---

> Codetree 강의 화면(스크린샷 PDF)을 바탕으로 정리했습니다. fileciteturn0file0

## 1. 문제 요약

- **선택지 K개**가 있고, 이를 **N번** 선택해 길이 `N`의 수열을 만든다.
- 매번 선택은 독립적이라 **같은 값을 여러 번 선택할 수 있다(중복 허용)**.
- 목표: 가능한 모든 수열을 **사전식(또는 지정된 순서)** 으로 출력(또는 탐색)한다.

예) `K=2, N=3`이면 가능한 수열 수는 `2^3 = 8`개  
`(1,1,1) ... (2,2,2)` 처럼.

---

## 2. 핵심 개념: 중복순열(= K진수처럼 생각)

길이가 `N`인 수열의 각 자리에 `K`가지 선택지가 있으므로 전체 경우의 수는

$$
K^N
$$

즉, 완전탐색(Brute Force)로 전부 만들어도 입력 범위가 작으면 충분히 가능하다.

---

## 3. 재귀/백트래킹 설계

### 상태 정의

- `answer`: 현재까지 만든 수열(리스트)
- `curr_num`: **몇 번째 자리**를 채우고 있는지 (1부터 시작한다고 가정)

### 재귀 함수 의미

`choose(curr_num)`  
→ `curr_num`번째 값을 정하고, 다음 자리로 넘어간다.

### 종료 조건(Base case)

- `curr_num == N + 1`  
  → 1~N번째까지 전부 채운 상태이므로 `answer`를 출력하고 종료

### 점화(재귀 전개)

- `curr_num`번째 자리에는 `1..K` 중 하나를 넣을 수 있다.
- 하나 넣고 → 다음 자리로 재귀 → 되돌리기(pop)

---

## 4. 동작 예시 (K=2, N=3)

루트에서 첫 번째 자리를 고르고, 그 다음 두 번째, 세 번째 자리를 고르는 트리 구조가 된다.

```
depth 1        depth 2        depth 3
  ( )            ( )            ( )
 /   \          /   \          /   \
1     2        1     2        1     2
```

결과 출력(한 줄에 하나씩):

```
1 1 1
1 1 2
1 2 1
1 2 2
2 1 1
2 1 2
2 2 1
2 2 2
```

---

## 5. Python 구현 예시

```python
import sys

k, n = map(int, sys.stdin.readline().split())
answer = []

def choose(curr_num: int):
    # base case: 1..n 까지 모두 채움
    if curr_num == n + 1:
        sys.stdout.write(" ".join(map(str, answer)) + "\n")
        return

    # curr_num 자리에 1..k 중 하나 선택
    for x in range(1, k + 1):
        answer.append(x)
        choose(curr_num + 1)
        answer.pop()

choose(1)
```

---

## 6. 시간 / 공간 복잡도

- 생성되는 수열 수: `K^N`
- 각 수열을 출력하려면 길이 `N`만큼 처리(출력 문자열 생성 등)가 들어가므로

$$
\text{Time} = O(K^N \cdot N)
$$

- 재귀 깊이: `N`, `answer` 크기도 최대 `N`

$$
\text{Space} = O(N)
$$

---

## 7. 자주 하는 실수 체크리스트

- **종료 조건 오프바이원**
  - `curr_num == n`에서 출력해버리면 마지막 자리를 못 채우고 출력할 수 있음  
  - 보통 `curr_num == n + 1`이 깔끔함(1-index 기준)
- **pop() 누락**
  - 백트래킹에서 되돌리기 없으면 `answer`가 계속 쌓여서 틀림
- **출력 형식**
  - 문제에서 요구하는 공백/줄바꿈 형식 꼭 맞추기
- **속도**
  - 출력이 많으면 `print()` 대신 `sys.stdout.write()`가 유리할 때가 있음

---

## 8. 확장 아이디어

이 패턴은 “자리를 하나씩 채운다”는 구조라서 아래 문제들로 그대로 확장된다.

- **중복 허용 X** → 방문 체크(`visited`) 추가해서 순열 만들기
- **조합(순서 무시)** → 다음에 선택 가능한 시작값(start) 관리
- **조건 만족 수열만 출력** → 중간에 조건 위배 시 가지치기(pruning)

