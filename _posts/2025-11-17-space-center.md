---
title: 우주 정거장
sidebar:
  nav: docs-ko
aside:
  toc: true
key: 20251117
tags: [Algorithm, binary_search, implementation, space_station, CHO, recursive, divide_conquer]
lang: ko
math: true
---



# 우주정거장

> ## 문제풀이방법
>
> * 한 점에서 다른 선분으로 계속 수선의 발을 내리면서, 어느순간 수선의발을 내려도 길이가 크게 짧아지지 않는점에서 멈출것이다.

<img src="../images/2022-08-10-space_station/스크린샷 2022-08-10 오후 4.07.21.png" alt="스크린샷 2022-08-10 오후 4.07.21" style="zoom:33%;" />

> * 다만, 컴퓨터에게 수선의 발을 내린다를 이해시키기는 좀 어려운것 같다. 이때 차용한, 아이디어가 다음과 같은 아이디어.

<img src="../images/2022-08-10-space_station/스크린샷 2022-08-10 오후 4.11.57.png" alt="스크린샷 2022-08-10 오후 4.11.57" style="zoom:33%;" />

> * 점 B에서 선분CD에 수선의발을 내린다고 가정하자. 
> * 이때, 선분BM의 살짝 왼쪽에 있는 점과 B의 거리 >  선분BM의 살짝 오른쪽에 있는 점과 B의 거리이면, C와M사이에 수선의 발 후보가 있다고 보면되고, 그 반대의 경우에는 M과D사이에 수선의발 후보가 있다고 보면 된다. 
> * 근데, 문제가 생긴게, 이제 선분AB, 선분CD를 왔다갔다 하면서 최적의 좌표들을 찾아야 하는데, 이 왔다갔다 하는게 코드로 구현하기 쉽지가 않아서 고민이 많다. 
> * 우선 도전해볼 방향은 다음과 같다. 조건문을 어떻게 구성할지가 관건인것 같다.

```python
from math import sqrt, ceil, floor

def shortest_point(R, head, tail):
    mid = [head[i] * (1/2) + tail[i] * (1/2) for i in range(3)]
    left_distance = sum(map(lambda x, y: (x-y) ** 2, head, R))
    right_distance = sum(map(lambda x, y: (x-y) ** 2, R, tail))
    if left_distance > right_distance:
        shortest_point(R, mid, tail)
    elif left_distance > right_distance:
        shortest_point(R, head, mid)
    else : return mid

def station(A, B, C, D):
    P, Q = A, C
    while True:
        P = shortest_point(P, C, D)
        Q = shortest_point(Q, C, D)
        if True: continue
        break

```



```python
from math import sqrt, ceil, floor
import math

def shortest_point(R, head, tail):

    mid = [head[i] * (1/2) + tail[i] * (1/2) for i in range(3)]

    left_distance = sum(map(lambda x, y: (x-y) ** 2, head, R))
    right_distance = sum(map(lambda x, y: (x-y) ** 2, R, tail))

    if left_distance > right_distance:
        shortest_point(R, mid, tail)
    elif left_distance > right_distance:
        shortest_point(R, head, mid)
    else : return mid

def station(A, B, C, D):
    P, Q = A, C
    
    while True:
        before = sum(map(lambda x, y: (x-y) ** 2, P, Q))
        P = shortest_point(P, C, D)
        Q = shortest_point(Q, A, B)
        after = sum(map(lambda x, y: (x-y) ** 2, P, Q))
        if abs(after - before) > 1/1000000 : continue
        break
    print(after)
    return
    
#  A, B, C, D 좌표 저장
inp = [list(map(int, input().strip().split())) for _ in range(4)]
A, B, C, D = inp

# 초기값설정 : P <- A, Q <- C
P, Q = A, C

#  tmp : 왼쪽을 갈지, 오른쪽으로 갈지 판단해주는 함수
#  매우 작은 값으로 설정해준다. 또한, 재귀함수의 종료조건까지 담당
tmp = 1/100000000

station(A, B, C, D)
```

> * 아 완벽하게 푼것 같은데, 왜 안나오지 답이... 
> * TypeError: 'NoneType' object is not iterable 이런 에러가 뜨는데, 왜 before 와 after에서 Nonetype object를 리턴하는지 의문이야....



```python
from math import sqrt, ceil, floor
import math

def my_length(a, b):
    length = (a[0] -b[0]) ** 2 + (a[1] -b[1]) ** 2 + (a[2] -b[2]) ** 2 
    return length

def shortest_point(R, head, tail):

    mid = [head[i] * (1/2) + tail[i] * (1/2) for i in range(3)]

    left_distance = sum(map(lambda x, y: (x-y) ** 2, head, R))
    right_distance = sum(map(lambda x, y: (x-y) ** 2, R, tail))

    if left_distance > right_distance:
        shortest_point(R, mid, tail)
    elif left_distance < right_distance:
        shortest_point(R, head, mid)
    elif left_distance - right_distance < 0.1:
        return mid
    else : return mid

def station(A, B, C, D):
    P, Q = A, C
    
    while True:
        before = my_length(P, Q)
        P = shortest_point(P, C, D)
        Q = shortest_point(Q, A, B)
        after = my_length(P, Q)
        if abs(after - before) > 1/1000000 : continue
        break
    print(after)
    return
    
#  A, B, C, D 좌표 저장
inp = [list(map(int, input().strip().split())) for _ in range(4)]
A, B, C, D = inp

# 초기값설정 : P <- A, Q <- C
P, Q = A, C

#  tmp : 왼쪽을 갈지, 오른쪽으로 갈지 판단해주는 함수
#  매우 작은 값으로 설정해준다. 또한, 재귀함수의 종료조건까지 담당
tmp = 1/100000000

station(A, B, C, D)
```

* 도대체가 왜 동작을 안하는지 의문이다.... 모르겠음.........
