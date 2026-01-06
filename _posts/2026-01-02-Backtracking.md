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


> Codetree 문제: **K개의 숫자 중 1개를 N번 뽑기** (중복 허용, 사전순 출력) 

## 문제 요약

- 1부터 K까지의 숫자 중에서 **N번** 숫자를 뽑는다.
- **중복 선택 가능**.
- 가능한 모든 수열을 **사전순(오름차순)** 으로 출력한다.

예) `K=3, N=2`라면

```
1 1
1 2
1 3
2 1
2 2
2 3
3 1
3 2
3 3
```

---

## 내가 처음에 했던 실수: `answer = []`로 초기화

처음 코드는 이런 형태였음(핵심만 발췌):

```python
def choose(curr_num, idx):
    if idx == N + 1:
        print_answer()
        answer = []   # <- 여기!
        return
```

여기서 생긴 문제는 2가지야.

### 1) 파이썬 스코프 때문에 에러가 난다 (UnboundLocalError)

함수 안에서 `answer = []`처럼 **대입(assign)** 을 해버리면,
파이썬은 `answer`를 **지역변수**로 판단한다.

그럼 함수 앞부분에서 `answer.append(...)`를 하려는 순간:

- “지역변수 answer를 쓰려 했는데, 아직 값이 할당되기 전이다”
- 그래서 `UnboundLocalError`가 난다.

즉, `answer = []` 한 줄이 **스코프 규칙을 바꿔버린 것**.

### 2) 사실상 초기화가 필요 없다 (백트래킹이 원상복구함)

백트래킹의 핵심 패턴은 아래 한 줄로 요약됨:

> **선택(append) → 재귀 → 취소(pop)**

이 `pop()`이 “리스트를 다시 원래 상태로 되돌리는 역할”을 해서  
출력 후에 `answer`를 새로 만들 필요가 없어.

---

## 왜 초기화 없이도 정답이 되나? (append/pop이 모든 걸 해결)

너가 만든 정답 코드는 이거:

```python
K, N = map(int, input().split())
answer = []

def print_answer():
    for elem in answer:
        print(elem, end=" ")
    print()

def choose(idx):
    if idx == N:
        print_answer()
        return

    for i in range(1, K + 1):
        answer.append(i)      # 1) 선택
        choose(idx + 1)       # 2) 다음 칸으로
        answer.pop()          # 3) 선택 취소 (원상복구)

choose(0)
```

이게 가능한 이유를 “재귀 호출 스택” 관점에서 보면 더 명확해.

### 예시: K=3, N=2일 때 흐름

처음 `answer = []` 에서 시작

1. `answer.append(1)` → `[1]`
2. 재귀 들어감 → 거기서 다시
   - `append(1)` → `[1, 1]` → 출력 → `pop()` → `[1]`
   - `append(2)` → `[1, 2]` → 출력 → `pop()` → `[1]`
   - `append(3)` → `[1, 3]` → 출력 → `pop()` → `[1]`
3. 재귀 끝나고 돌아오면 `pop()` 해서 `[ ]`로 복구
4. 다음 i=2로 진행 → `[2]` … 반복

**중요 포인트**

- `print_answer()`는 **현재 상태의 answer**만 출력하고 끝.
- 출력이 끝난 뒤에는 `pop()`이 계속 실행되면서 **이전 상태로 되돌아감**.
- 그래서 전역 `answer` 하나를 계속 재사용해도 안전함.

---

## 종료 조건은 왜 `idx == N`이 깔끔한가?

- `idx`를 “현재까지 뽑은 개수”라고 두면,
- N개 뽑았을 때(`idx == N`) 출력하면 딱 맞음.

처음 호출이 `choose(0)`인 이유도:

- 아직 0개 뽑았으니까.

---

## 참고: 만약 진짜로 `answer = []`를 쓰고 싶다면?

굳이 “리스트 자체를 새로 할당”하려면 `global`이 필요해.

```python
def choose(idx):
    global answer
    ...
```

하지만 백트래킹에서는 **재할당이 필요 없고**, 오히려 버그를 만들기 쉬워서 비추천.

---

## 시간 복잡도

- 각 자리마다 K가지 선택 → 총 경우의 수 `K^N`
- 출력도 매번 N개 찍으니 대략 `O(N * K^N)` 정도가 됨.

---

## 정리 한 줄

> `answer`는 “현재까지의 선택 경로”를 담는 **공용 리스트**고,  
> `append → 재귀 → pop`이 항상 원상복구를 해주기 때문에 초기화가 필요 없다.