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
> * Starting from a point, you repeatedly drop a perpendicular (foot of the perpendicular) to the other segment. At some point, even if you keep dropping perpendiculars, the length of the segment will not get significantly shorter, and you will stop near that point.
>
> ![space-station-2]({{ "/assets/images/2025-11-17-space-center/2.png" | relative_url }})
>
> * However, it’s a bit difficult to directly explain “dropping a perpendicular” to the computer. So I borrowed the following idea.
>
> ![space-station-1]({{ "/assets/images/2025-11-17-space-center/1.png" | relative_url }})
>
> * Suppose we drop a perpendicular from point **B** to segment **CD**.
> * If the distance from **B** to a point slightly **left** of the midpoint **M** on segment **CD** is greater than the distance from **B** to a point slightly **right** of **M**, then the candidate location of the foot of the perpendicular lies between **C** and **M**.  
>   If the opposite inequality holds, then the candidate region is between **M** and **D**.
> * The trouble is that we need to move back and forth between segment **AB** and segment **CD** to find the optimal pair of coordinates, and implementing this “back-and-forth” logic cleanly in code is not easy.
> * The direction I tried is roughly as follows – the key is how to design the conditionals.

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
inp = [list(map(int, input().strip().split())) for _ in range(4)]
A, B, C, D = inp

# Initial values: P <- A, Q <- C
P, Q = A, C

# tmp: was intended as a small threshold to decide left/right and termination
tmp = 1/100000000

station(A, B, C, D)
It feels like I solved it perfectly, but the answer doesn’t come out…

I get an error: TypeError: 'NoneType' object is not iterable. I’m not sure why before and after are getting NoneType objects…


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
inp = [list(map(int, input().strip().split())) for _ in range(4)]
A, B, C, D = inp

# Initial values: P <- A, Q <- C
P, Q = A, C

# tmp: small threshold to decide direction (left/right) and also termination
tmp = 1/100000000

station(A, B, C, D)
I honestly have no idea why this still doesn’t work… I just don’t get it… 😵‍💫

