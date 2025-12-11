---
title: Linked list
sidebar:
  nav: docs-ko
aside:
  toc: true
key: 20251210
tags: [data structure, linked-list]
lang: ko
math: true
---

# Singly Linked List 정리

## 1. 배열 vs 연결 리스트

- 배열에서 **중간에 삽입/삭제**를 하면  
  그 뒤의 원소들을 한 칸씩 밀거나 당겨야 해서 보통 `O(N)` 시간이 든다.
- 반면 연결 리스트는 **포인터(링크)만 끊어서 다시 이어 주면 되기 때문에**  
  삽입·삭제 자체는 `O(1)`에 할 수 있다.
- 하지만 원하는 원소를 찾으려면 **앞에서부터 차례대로** 따라가야 해서  
  조회/탐색은 `O(N)` 이다.

그래서

- **검색이 많으면 → 배열**
- **삽입·삭제가 많으면 → 연결 리스트**

를 주로 사용한다.

---

## 2. 노드(Node)의 정의

연결 리스트의 기본 단위가 **노드(node)** 이다.  
간단히 말해, 노드는 “정보를 들고 있으면서 다음 노드를 가리키는 상자”이다.

하나의 노드는 보통 다음 두 정보를 가진다.

1. `data` : 실제 값(정수, 문자열, 구조체 등)
2. `next` : 다음 노드를 가리키는 참조(포인터)
   - 마지막 노드는 **다음 노드가 없으므로** `next = null` 로 둔다.

그림으로 표현하면:

```text
[ Data ] -> [ Next ]
```

---

## 3. Singly Linked List 구조 & 생성

### 3.1 단일 노드 생성

값이 3인 노드를 하나 만들면:

```pseudo
set node1 = node(3)      # data = 3, next = null
```

또는

```pseudo
set node1 = node()       # 비어 있는 노드 생성
node1.data = 3
# next는 기본값으로 null
```

이 상태에서 `node1` 은 혼자 있는 노드이며, 아직 어떤 리스트에도 속하지 않은 셈이다.

---

### 3.2 노드 두 개 연결하기

값이 5인 새 노드 `node2` 를 만들고 `node1` 뒤에 붙여 보자.

```pseudo
set node2 = node(5)      # data = 5, next = null

node1.next = node2       # node1 -> node2 연결
```

이제 구조는 다음과 같다.

```text
node1 (data=3)  ->  node2 (data=5)  ->  null
```

이때

```pseudo
print(node2.data)        # 5
print(node1.next.data)   # 5    (node1.next 가 node2 를 가리키기 때문)
```

---

### 3.3 연결 끊기(삭제의 기본 아이디어)

`node1` 과 `node2` 사이 연결을 끊고 싶다면 `next` 에 `null` 을 대입하면 된다.

```pseudo
node1.next = null        # node1 과 node2 분리
```

이 아이디어가 **노드 삭제**의 기본이다.  
(실제 구현에서는 끊고 난 뒤 불필요한 노드를 free/delete 해 줌)

---

## 4. Head 와 Tail

여러 개의 노드를 연결해서 리스트를 만들면,  
어디서부터 시작해야 할지 **시작점**을 알고 있어야 한다.

- 리스트의 **첫 번째 노드**를 `head` 라고 부른다.
- (필요하다면) **마지막 노드**를 `tail` 이라고 저장해 두기도 한다.

```text
Head -> Node -> Node -> ... -> Tail
```

- 순회(traversal) 할 때는 보통 `head` 부터 시작해서  
  `next` 를 따라가며 `null` 에 도달할 때까지 방문한다.
- `tail` 을 알고 있으면,
  - “현재 노드가 tail인가?” 만 확인해서 순회를 종료할 수 있다.
  - 끝에 새 노드를 붙이는 연산도 `tail.next` 만 바꾸면 되므로 편하다.

---

## 5. Singly Linked List 의 특성 & 시간 복잡도

단일 연결 리스트는 다음 연산들을 지원한다.

1. **순회(Traversal)**  
   - `head` 부터 `next` 를 따라 `tail` 까지 확인해야 하므로  
     시간 복잡도는 `O(N)`.

2. **삽입/삭제(Insertion / Deletion)**  
   - 이미 삽입/삭제 위치의 **이전 노드**를 알고 있다면,  
     링크만 끊어서 다시 이어주면 되므로 `O(1)`에 가능.
   - 예) `prev -> target -> next` 에서 `target` 을 삭제  
     → `prev.next = next`

3. **역방향 이동 불가**  
   - `next` 방향으로만 이동할 수 있는 **단방향 구조**라  
     “이전 노드(prev)” 로 바로 거슬러 올라갈 수 없다.  
   - 이것이 **singly** linked list 의 특징이다.

---

## 6. 삽입(Insertion) 연산

여기서는 `head` 와(선택적으로) `tail` 을 따로 변수로 들고 있다고 가정한다.

### 6.1 맨 앞에 삽입 (Insert at Head)

```pseudo
function push_front(x):
    new_node = node(x)
    new_node.next = head    # 새 노드가 기존 head 를 가리키게
    head = new_node         # head 를 새 노드로 변경
    if tail is null:        # 리스트가 비어 있었던 경우
        tail = new_node
```

- 시간 복잡도: `O(1)`

### 6.2 맨 뒤에 삽입 (Insert at Tail)

`tail` 포인터가 있다고 가정하면:

```pseudo
function push_back(x):
    new_node = node(x)      # new_node.next 는 기본적으로 null
    if tail is not null:    # 기존 노드가 있다면
        tail.next = new_node
        tail = new_node
    else:                   # 리스트가 비어 있었던 경우
        head = new_node
        tail = new_node
```

- 시간 복잡도: `O(1)`  (tail 을 알고 있으므로)

`tail` 이 없다면, 맨 뒤까지 순회해서 찾아야 하므로 `O(N)` 이 된다.

### 6.3 중간에 삽입 (Insert After Given Node)

어떤 노드 `prev` 뒤에 값을 삽입한다고 하자.

```pseudo
function insert_after(prev, x):
    if prev is null:
        # prev 가 없으면 의미 없음
        return

    new_node = node(x)
    new_node.next = prev.next   # 새 노드가 prev 의 다음 노드를 가리키게
    prev.next = new_node        # prev 가 새 노드를 가리키도록 변경

    if prev == tail:            # prev 가 tail 이었다면 tail 갱신
        tail = new_node
```

- `prev` 노드를 이미 알고 있다고 가정하면 시간 복잡도는 `O(1)`.

---

## 7. 삭제(Deletion) 연산

삭제의 기본은 “이전 노드가 다음 노드를 건너뛰도록 링크를 다시 잇는 것” 이다.

### 7.1 맨 앞 삭제 (Delete Head)

```pseudo
function pop_front():
    if head is null:
        return   # 삭제할 것이 없음

    removed = head          # 제거할 노드 저장(필요하면 나중에 free)
    head = head.next        # head 를 다음 노드로 옮김

    if head is null:        # 리스트가 비게 되었다면
        tail = null
```

- 시간 복잡도: `O(1)`

### 7.2 중간/맨 뒤 삭제 (Delete After Given Node)

`prev` 뒤의 노드를 삭제한다고 하자. (즉, `prev.next` 를 지우는 경우)

```pseudo
function erase_after(prev):
    if prev is null or prev.next is null:
        return              # 삭제할 노드가 없음

    target = prev.next
    prev.next = target.next # target 을 건너뛰어 연결

    if target == tail:      # 지운 노드가 tail 이었다면 tail 갱신
        tail = prev
```

- 시간 복잡도: `O(1)` (prev 를 이미 알고 있을 때)

### 7.3 값으로 삭제 (Delete by Value)

특정 값 `x` 를 가진 첫 번째 노드를 찾아 삭제해 보자.  
이 경우는 **찾는 과정 때문에** `O(N)` 시간이 든다.

```pseudo
function erase_by_value(x):
    if head is null:
        return

    # 1) head 가 x 인 경우
    if head.data == x:
        pop_front()
        return

    # 2) 그 외: 이전 노드를 추적하면서 찾기
    prev = head
    curr = head.next
    while curr is not null:
        if curr.data == x:
            prev.next = curr.next
            if curr == tail:
                tail = prev
            break           # 첫 번째 것만 삭제
        prev = curr
        curr = curr.next
```

- 탐색: 최악의 경우 `O(N)`
- 실제 링크 변경: `O(1)`

---

## 8. 한 줄 요약

> Singly Linked List 는  
> **“data + next 포인터로 이루어진 노드들을 한 줄로 이어 붙인 구조”** 이고,  
> 삽입/삭제는 위치(또는 이전 노드)를 알고 있으면 `O(1)` 에 가능하지만,  
> 검색/위치 찾기는 `head` 부터 차례대로 가야 해서 `O(N)` 이 걸리며,  
> 한 방향으로만 이동할 수 있는 단방향 연결 리스트이다.



# Merging Linked Lists 정리

## 1. 문제 설명

- 오름차순으로 **정렬되어 있는 단일 연결 리스트** `SLL1`, `SLL2` 가 두 개 있다.
- 이 둘을 **하나의 새로운 연결 리스트**로 합치되,
- **결과 리스트도 오름차순 정렬** 상태가 되도록 만들어야 한다.

즉, 정렬된 두 배열을 merge 하듯이, **정렬된 두 linked list 를 merge** 하는 문제다.

---

## 2. 기본 아이디어

1. `node1` 은 `SLL1.head` 에서,  
   `node2` 는 `SLL2.head` 에서 시작한다.
2. 매 반복마다
   - `node1.data` 와 `node2.data` 중 **더 작은 값**을 `result` 리스트의 끝에 붙인다.
   - 해당 값을 사용한 쪽 포인터(`node1` 또는 `node2`)를 한 칸 앞으로 이동한다.
3. 둘 중 한 리스트를 다 썼다면
   - 남아 있는 다른 리스트의 값들을 그대로 뒤에 붙인다.
4. 이렇게 하면 항상 **작은 값부터 차례대로** 들어가므로,  
   `result` 도 오름차순 정렬이 유지된다.

---

## 3. 주어진 의사코드

문제에서 준 뼈대 코드는 다음과 같다.

```pseudo
function solution(SLL1, SLL2)
    set result = empty SLL
    node1 = SLL1.head
    node2 = SLL2.head

    while node1 != null or node2 != null
        if node2 == null or (1)
            result.insert_end((2))
            node1 = node1.next
        else
            result.insert_end((3))
            node2 = node2.next

    return result
```

여기서 `(1)`, `(2)`, `(3)` 에 들어갈 내용을 이해하면 된다.

---

## 4. 각 부분 채우기

### 4.1 `(2)` 와 `(3)` – 어느 리스트에서 값을 가져올까?

- if 블록은 **첫 번째 리스트(SLL1, node1)** 쪽 값을 사용한다.
- else 블록은 **두 번째 리스트(SLL2, node2)** 쪽 값을 사용한다.

따라서

- `(2)` → `node1.data`
- `(3)` → `node2.data`

```pseudo
result.insert_end(node1.data)   # (2)
...
result.insert_end(node2.data)   # (3)
```

---

### 4.2 `(1)` – 언제 node1 쪽 값을 써야 할까?

if 의 조건은

```pseudo
if node2 == null or (1)
```

이므로 두 가지 경우에 `node1` 쪽 값을 사용한다.

1. `node2 == null`  
   → 두 번째 리스트는 이미 끝났음.  
     남은 값은 전부 `node1` 에서 가져와야 한다.
2. 두 리스트 모두 남아 있고,  
   **node1 의 값이 node2 의 값보다 작거나 같을 때**  
   → 정렬을 유지하려면 작은 값인 `node1.data` 를 먼저 넣어야 한다.

그래서 (1) 은 다음처럼 쓸 수 있다.

```pseudo
(node1 != null and node1.data <= node2.data)
```

> `node1 != null` 을 같이 넣는 이유:  
> `node1` 이 null 이면 `node1.data` 에 접근하면 안 되기 때문  
> (단락 평가가 있다고 가정)

---

## 5. 완성된 의사코드

```pseudo
function solution(SLL1, SLL2)
    set result = empty SLL
    node1 = SLL1.head
    node2 = SLL2.head

    while node1 != null or node2 != null
        if node2 == null or (node1 != null and node1.data <= node2.data)
            # SLL1 쪽 값이 더 작거나 같거나, SLL2 가 이미 끝난 경우
            result.insert_end(node1.data)
            node1 = node1.next
        else
            # SLL2 쪽 값이 더 작은 경우
            result.insert_end(node2.data)
            node2 = node2.next

    return result
```

이 알고리즘은

- 두 리스트의 길이를 각각 `N`, `M` 이라 하면,
- 각 노드를 한 번씩만 방문하므로 시간 복잡도는 `O(N + M)` 이다.

---