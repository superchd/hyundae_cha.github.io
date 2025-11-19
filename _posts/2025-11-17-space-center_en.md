---
title: Space Station
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251117
tags: [algorithm, binary_search, implementation, space_station, CHO, recursive, divide_conquer]
lang: en
math: true
---

# Space Station

> ## Idea for solving the problem
>
> * Start from a point and keep dropping the foot of the perpendicular to the other segment. At some point, even if you keep dropping perpendiculars, the distance will no longer decrease much — you stop near that point.
>
> ![screenshot-2]({{ "/assets/images/2025-11-17-space-center/2.png" | relative_url }})
>
> * But it's hard to directly teach the computer what it means to “drop a perpendicular.” So I borrowed the following idea instead.
>
> ![screenshot-1]({{ "/assets/images/2025-11-17-space-center/1.png" | relative_url }})
>
> * Suppose we drop a perpendicular from point **B** to segment **CD**.
> * If the distance from **B** to a point slightly **left** of the midpoint **M** on segment **CD** is **greater** than the distance from **B** to a point slightly **right** of **M**, then the candidate location of the foot of the perpendicular lies between **C** and **M**.  
>   If the opposite inequality holds, then the candidate region is between **M** and **D**.
> * The problem is: we also have to move back and forth between segment **AB** and segment **CD** to find the optimal coordinates, and implementing this “back-and-forth” logic in code is tricky.
> * The first approach I tried is below. The key is how to design the conditionals.

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
    else:
        return mid

def station(A, B, C, D):
    P, Q = A, C
    while True:
        P = shortest_point(P, C, D)
        Q = shortest_point(Q, C, D)
        if True:
            continue
        break


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
    else:
        return mid

def station(A, B, C, D):
    P, Q = A, C
    
    while True:
        before = sum(map(lambda x, y: (x-y) ** 2, P, Q))
        P = shortest_point(P, C, D)
        Q = shortest_point(Q, A, B)
        after = sum(map(lambda x, y: (x-y) ** 2, P, Q))
        if abs(after - before) > 1/1000000:
            continue
        break
    print(after)
    return
    
# Store coordinates for A, B, C, D
inp = [list(map(int, input().strip().split())) for _ in range(4))]
A, B, C, D = inp

# Initial values: P <- A, Q <- C
P, Q = A, C

# tmp: a very small value to decide which side to go (left/right)
# and also to serve as the recursion stopping condition
tmp = 1/100000000

station(A, B, C, D)

```

It feels like I solved it perfectly, but the answer still doesn’t come out…

I get an error: TypeError: 'NoneType' object is not iterable. I don’t understand why before and after are getting NoneType — that’s confusing.


```python
from math import sqrt, ceil, floor
import math

def my_length(a, b):
    length = (a[0] - b[0]) ** 2 + (a[1] - b[1]) ** 2 + (a[2] - b[2]) ** 2
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
    else:
        return mid

def station(A, B, C, D):
    P, Q = A, C
    
    while True:
        before = my_length(P, Q)
        P = shortest_point(P, C, D)
        Q = shortest_point(Q, A, B)
        after = my_length(P, Q)
        if abs(after - before) > 1/1000000:
            continue
        break
    print(after)
    return
    
# Store coordinates for A, B, C, D
inp = [list(map(int, input().strip().split())) for _ in range(4))]
A, B, C, D = inp

# Initial values: P <- A, Q <- C
P, Q = A, C

# tmp: a very small value to decide which side to go (left/right),
# and also to serve as the recursion stopping condition
tmp = 1/100000000

station(A, B, C, D)

```

I honestly have no idea why this doesn’t work… I just don’t get it…
