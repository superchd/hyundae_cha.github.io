---
title: DFS Traversal on Graphs (그래프 DFS 기초)
sidebar:
  nav: docs-ko
aside:
  toc: true
key: 20251231_dfs_graph_traversal
tags: [graph, dfs, traversal, python]
lang: ko
math: true

---


## 1. 그래프(Graph)란?

그래프는 **정점(Vertex)** 과 **간선(Edge)** 로 이루어진 자료구조다.

- 정점: 데이터(노드)
- 간선: 정점 사이의 연결 관계

그래프는 **무방향(undirected)** / **방향(directed)** 그래프로 나뉜다.

---

## 2. 그래프 표현 방법

### 2.1 인접 행렬(Adjacency Matrix)

정점 개수를 `V`라고 할 때, `V x V` 2차원 배열 `graph`로 표현한다.

- 두 정점 `i`, `j`가 연결되어 있으면 `graph[i][j] = 1`
- 연결되어 있지 않으면 `graph[i][j] = 0`

**특징**

- 무방향 그래프면 인접 행렬이 **대칭**이다. (`graph[i][j] == graph[j][i]`)
- 공간 복잡도: `O(V^2)`
- 특정 간선 존재 여부 확인: `O(1)`
- 한 정점에서 연결된 모든 정점 탐색: 행 전체를 훑어야 해서 `O(V)`

> 정점 수가 작거나(밀집 그래프) 간선 확인이 잦을 때 유리.

---

### 2.2 인접 리스트(Adjacency List)

정점 `i`에 인접한 정점들을 리스트로 저장한다.

예: `graph[i] = [i와 연결된 정점들...]`

**특징**

- 공간 복잡도: `O(V + E)`
- 간선 존재 여부 확인: 리스트 탐색이라 평균적으로 더 비용이 든다.
- 한 정점에서 인접 정점 탐색: `deg(i)`만큼만 보면 된다.

> 정점 수가 크거나(희소 그래프) 전체 순회가 목적일 때 유리.

---

## 3. DFS(Depth-First Search)란?

DFS는 **한 경로로 가능한 깊게 들어가다가**, 더 갈 곳이 없으면 **되돌아오며(backtracking)** 다른 경로를 탐색하는 그래프 순회 방법이다.

- 시작 정점에서 출발해 방문
- 방문 처리(visited)로 **무한 루프/재방문 방지**
- 재귀(Recursive) 또는 스택(Stack)으로 구현 가능

---

## 4. visited(방문 배열)가 왜 필요한가?

그래프에는 사이클(순환)이 있을 수 있다.

- visited가 없으면 `1 -> 2 -> 1 -> 2 ...` 같은 무한 반복 가능
- DFS의 기본 원칙:
  1) 시작점에서 연결된 정점들을 방문한다
  2) 이미 방문한 정점은 다시 방문하지 않는다

---

## 5. DFS 구현 (Python)

아래 코드는 **정점 번호가 1..V**라고 가정한다.
(코드에서는 편하게 `V+1` 크기의 배열을 쓴다.)

### 5.1 인접 행렬 + DFS(재귀)

```python
def dfs_matrix(v, graph, visited):
    visited[v] = True
    print(v, end=" ")

    V = len(graph) - 1  # 1..V
    for nxt in range(1, V + 1):
        if graph[v][nxt] == 1 and not visited[nxt]:
            dfs_matrix(nxt, graph, visited)
```

### 5.2 인접 리스트 + DFS(재귀)

```python
def dfs_list(v, graph, visited):
    visited[v] = True
    print(v, end=" ")

    for nxt in graph[v]:
        if not visited[nxt]:
            dfs_list(nxt, graph, visited)
```

### 5.3 DFS(스택) - 재귀 없이

```python
def dfs_stack(start, graph):
    visited = [False] * len(graph)
    stack = [start]
    order = []

    while stack:
        v = stack.pop()
        if visited[v]:
            continue
        visited[v] = True
        order.append(v)

        # 작은 번호를 먼저 방문하려면 역순으로 push
        for nxt in reversed(graph[v]):
            if not visited[nxt]:
                stack.append(nxt)

    return order
```

---

## 6. 시간/공간 복잡도 요약

- 인접 행렬:
  - 공간: `O(V^2)`
  - DFS 순회: 보통 `O(V^2)` (각 정점에서 행을 훑기 때문)

- 인접 리스트:
  - 공간: `O(V + E)`
  - DFS 순회: `O(V + E)` (각 간선을 최대 한 번씩 확인)

---

## 7. 실전 팁

- **연결 요소(connected component)**, **사이클 탐지**, **경로 존재 여부**, **그래프/격자 탐색**의 기본이 DFS/BFS다.
- 입력 크기가 크면 보통 인접 리스트가 유리하다.
- 파이썬 재귀 DFS는 깊이가 매우 깊으면 `RecursionError`가 날 수 있어,
  `sys.setrecursionlimit(...)` 또는 스택 DFS를 고려한다.

---

## 8. 한 줄 정리

- 그래프 표현: **인접 행렬(간선 확인 빠름)** vs **인접 리스트(순회/메모리 유리)**
- DFS 핵심: **visited로 재방문 방지 + 깊게 탐색 후 백트래킹**



## 문제1 요약

- 정점 `N`개와 간선 `M`개로 이루어진 **무방향 그래프**가 주어진다  
- **1번 정점에서 시작**해서 간선을 따라 이동할 때, 도달 가능한 **서로 다른 정점의 개수**를 출력한다  
- 단, **정점 1은 개수에서 제외**한다 

---

## 핵심 아이디어

- 그래프를 **인접 리스트**로 만든 뒤 DFS로 탐색한다  
- `visited`로 이미 방문한 정점은 다시 방문하지 않는다  
- DFS 중 새 정점을 방문할 때마다 `vertex_cnt += 1` 해서 방문 정점 수를 센다  

인접 행렬도 가능하지만, `N`이 커지면 매번 `1..N`을 훑게 되어 비효율적이라 인접 리스트가 보통 더 깔끔하다  

---

## 풀이 흐름

1. `graph = [[] for _ in range(n+1)]` 로 인접 리스트 준비  
2. 간선 `(v1, v2)`를 읽어 `graph[v1].append(v2)`, `graph[v2].append(v1)`로 양방향 연결  
3. `visited[1] = True` 후 `dfs(1)` 실행  
4. DFS로 새 정점을 방문할 때마다 카운트 증가  
5. 최종 `vertex_cnt` 출력  

---

## 파이썬 구현 (인접 리스트 + DFS)

```python
n, m = map(int, input().split())

# 1-indexed 인접 리스트
graph = [[] for _ in range(n + 1)]
visited = [False] * (n + 1)
vertex_cnt = 0

def dfs(vertex):
    global vertex_cnt
    for nxt in graph[vertex]:
        if not visited[nxt]:
            visited[nxt] = True
            vertex_cnt += 1
            dfs(nxt)

# 무방향 그래프이므로 양쪽에 간선을 추가
for _ in range(m):
    v1, v2 = map(int, input().split())
    graph[v1].append(v2)
    graph[v2].append(v1)

visited[1] = True
dfs(1)

print(vertex_cnt)
```

---

## 시간 복잡도

- 인접 리스트 기준: `O(N + M)`  
  - 각 정점은 최대 한 번 방문  
  - 각 간선도 리스트에서 한 번씩만 확인  

---

## 자주 실수하는 포인트

- 무방향 그래프인데 한쪽만 간선을 추가하는 실수  
  - `graph[v1].append(v2)` 와 `graph[v2].append(v1)` 둘 다 필요  
- `visited[1] = True` 를 안 해서 1번 정점을 카운트에 포함시키는 실수  
- 재귀 DFS에서 파이썬 재귀 제한이 걸릴 수 있음  
  - 입력이 커질 수 있으면 `import sys; sys.setrecursionlimit(200000)` 같은 설정을 고려  

---

## 예시

입력

```
5 4
1 2
1 3
2 3
4 5
```

설명  

- 1번에서 {2, 3}은 도달 가능  
- {4, 5}는 다른 컴포넌트라 도달 불가  
- 따라서 출력은 `2`  



## 문제2 요약

- `N × M` 격자가 주어짐  
- 각 칸은 `1(이동 가능)` 또는 `0(이동 불가/뱀)`  
- 시작점은 좌상단 `(0, 0)`, 도착점은 우하단 `(N-1, M-1)`  
- **이동은 두 방향(오른쪽, 아래)** 만 가능  
- 도착점까지 갈 수 있으면 `1`, 아니면 `0` 출력
---

## 핵심 아이디어

- DFS(깊이 우선 탐색)로 `(0,0)`에서 출발해 **오른쪽/아래 방향**으로만 확장한다.
- 방문한 칸은 `visited`에 표시하여 중복 방문을 막는다.
- DFS가 끝난 뒤 `visited[N-1][M-1]`이 `1`이면 도착 가능, `0`이면 불가능이다.

---

## 풀이 흐름

1. `grid` 입력  
2. `visited`를 `N × M` 크기로 생성 (0/1 표시)
3. 보조 함수
   - `in_range(x, y)`: 격자 범위 체크
   - `can_go(x, y)`: 범위 안 + 미방문 + `grid[x][y] == 1` 인지 확인
4. `visited[0][0] = 1`로 시작점을 방문 처리
5. `dfs(0,0)` 실행
6. `visited[N-1][M-1]` 출력

---

## 구현 코드 (Python)

```python
# Variable declaration and input
n, m = tuple(map(int, input().split()))
grid = [
    list(map(int, input().split()))
    for _ in range(n)
]

visited = [
    [0 for _ in range(m)]
    for _ in range(n)
]

# Returns whether the given position is out of the grid.
def in_range(x, y):
    return 0 <= x and x < n and 0 <= y and y < m


# Checks whether it is possible to move to the given position.
def can_go(x, y):
    if not in_range(x, y):
        return False

    if visited[x][y] or grid[x][y] == 0:
        return False

    return True


def dfs(x, y):
    # two directions: right, down
    dxs, dys = [0, 1], [1, 0]

    for dx, dy in zip(dxs, dys):
        new_x, new_y = x + dx, y + dy

        if can_go(new_x, new_y):
            visited[new_x][new_y] = 1
            dfs(new_x, new_y)


visited[0][0] = 1
dfs(0, 0)

print(visited[n - 1][m - 1])
```

---

## 시간/공간 복잡도

- 시간: `O(N*M)`  
  - 각 칸은 최대 한 번 방문
- 공간: `O(N*M)`  
  - `visited` 배열 사용 (+ 재귀 호출 스택)

---

## 자주 실수하는 포인트

- **이동 방향을 4방향으로 구현**하면 문제 조건(2방향) 위반으로 오답이 날 수 있음  
- 시작점이나 도착점이 `0`이면 애초에 불가능 (문제에서 보장하는지 확인)
- 재귀 DFS는 입력이 커지면 재귀 제한에 걸릴 수 있으니 필요하면 `sys.setrecursionlimit` 설정 고려