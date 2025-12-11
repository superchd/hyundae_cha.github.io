---
title: Linked list
sidebar:
  nav: docs-en
aside:
  toc: true
key: 20251210
tags: [data structure, linked-list]
lang: en
math: true

---

# Singly Linked List Summary

## 1. Array vs Linked List

- When you **insert/delete in the middle of an array**,  
  you usually have to shift all the elements after that position by one,  
  which takes `O(N)` time.
- In contrast, a linked list only needs to **break and reconnect links (pointers)**,  
  so the insertion/deletion itself can be done in `O(1)`.
- However, to find the element you want, you must **follow the nodes one by one from the front**,  
  so lookup/traversal takes `O(N)` time.

So in practice:

- **If there are many searches → use an array**
- **If there are many insertions/deletions → use a linked list**

---

## 2. Definition of a Node

The basic unit of a linked list is a **node**.  
Roughly speaking, you can think of a node as *“a box that stores information and points to the next box.”*

A single node usually has two kinds of information:

1. `data` : the actual value (integer, string, struct, etc.)
2. `next` : a reference (pointer) to the next node
   - For the last node, there is **no next node**, so we set `next = null`.

We can draw it like this:

```text
[ Data ] -> [ Next ]
```

---

## 3. Singly Linked List Structure & Creation

### 3.1 Creating a Single Node

Let’s create a node with the value 3:

```pseudo
set node1 = node(3)      # data = 3, next = null
```

Or, create an empty node and then set its data:

```pseudo
set node1 = node()       # empty node
node1.data = 3
# next has default value null
```

At this point, `node1` is just a standalone node and does not yet belong to any list.

---

### 3.2 Connecting Two Nodes

Now create a new node `node2` with value 5 and attach it after `node1`:

```pseudo
set node2 = node(5)      # data = 5, next = null

node1.next = node2       # connect node1 -> node2
```

The structure becomes:

```text
node1 (data=3)  ->  node2 (data=5)  ->  null
```

Now:

```pseudo
print(node2.data)        # 5
print(node1.next.data)   # 5    (because node1.next points to node2)
```

---

### 3.3 Disconnecting Nodes (Basic Deletion Idea)

If you want to disconnect `node1` and `node2`, assign `null` to `node1.next`:

```pseudo
node1.next = null        # separate node1 and node2
```

This is the core idea of **deleting a node**.  
(In an actual implementation, you would also free/delete the removed node.)

---

## 4. Head and Tail

When you connect multiple nodes to form a list,  
you must know **where to start**.

- The **first node** of the list is called the `head`.
- Optionally, you can also store the **last node** as `tail`.

```text
Head -> Node -> Node -> ... -> Tail
```

- For traversal, you usually start from `head` and  
  follow `next` until you reach `null`.
- If you know `tail`, then:
  - You can stop traversal just by checking, “Is the current node the tail?”
  - Appending a new node at the end is easy: simply set `tail.next` to the new node.

---

## 5. Properties & Time Complexity of a Singly Linked List

A singly linked list supports the following operations:

1. **Traversal**
   - You must follow `next` from `head` to `tail`,  
     so the time complexity is `O(N)`.

2. **Insertion/Deletion**
   - If you already know the **previous node** of the insertion/deletion position,  
     you only need to break and reconnect links, which is `O(1)`.
   - Example: deleting `target` in `prev -> target -> next`  
     → `prev.next = next`

3. **No Backward Movement**
   - Because it only has `next`, it is **one-directional**.  
     You cannot directly move to the previous node (`prev`) from the current node.
   - This one-way nature is why it’s called a **singly** linked list.

---

## 6. Insertion Operations

Here we assume we have separate variables `head` and (optionally) `tail`.

### 6.1 Insert at the Front (Head)

```pseudo
function push_front(x):
    new_node = node(x)
    new_node.next = head    # new node points to the old head
    head = new_node         # head is updated to the new node
    if tail is null:        # list was empty before
        tail = new_node
```

- Time complexity: `O(1)`

### 6.2 Insert at the Back (Tail)

Assume we have a `tail` pointer:

```pseudo
function push_back(x):
    new_node = node(x)      # new_node.next is null by default
    if tail is not null:    # list already has nodes
        tail.next = new_node
        tail = new_node
    else:                   # list was empty
        head = new_node
        tail = new_node
```

- Time complexity: `O(1)` (because we know the tail)

If you don’t have `tail`, you must traverse to the end, which is `O(N)`.

### 6.3 Insert in the Middle (Insert After a Given Node)

Suppose we want to insert after a certain node `prev`:

```pseudo
function insert_after(prev, x):
    if prev is null:
        # inserting after “nothing” makes no sense
        return

    new_node = node(x)
    new_node.next = prev.next   # new node points to prev's next
    prev.next = new_node        # prev points to the new node

    if prev == tail:            # if prev was the tail, update tail
        tail = new_node
```

- Time complexity: `O(1)` (assuming we already know `prev`)

---

## 7. Deletion Operations

The core idea of deletion is  
“make the previous node skip the target node and point to the next one.”

### 7.1 Delete the Front Node (Head)

```pseudo
function pop_front():
    if head is null:
        return   # nothing to delete

    removed = head          # store the node to delete (free later if needed)
    head = head.next        # move head to the next node

    if head is null:        # list became empty
        tail = null
```

- Time complexity: `O(1)`

### 7.2 Delete a Middle or Last Node (Delete After a Given Node)

Delete the node right after `prev` (i.e., `prev.next`):

```pseudo
function erase_after(prev):
    if prev is null or prev.next is null:
        return              # nothing to delete

    target = prev.next
    prev.next = target.next # skip target

    if target == tail:      # if the deleted node was the tail, update tail
        tail = prev
```

- Time complexity: `O(1)` (again, assuming we already know `prev`)

### 7.3 Delete by Value

Now delete the first node with value `x`.  
This requires **searching first**, so `O(N)` time in total.

```pseudo
function erase_by_value(x):
    if head is null:
        return

    # 1) head has the value x
    if head.data == x:
        pop_front()
        return

    # 2) otherwise: track the previous node while searching
    prev = head
    curr = head.next
    while curr is not null:
        if curr.data == x:
            prev.next = curr.next
            if curr == tail:
                tail = prev
            break           # delete only the first occurrence
        prev = curr
        curr = curr.next
```

- Search: worst-case `O(N)`
- Actual link change: `O(1)`

---

## 8. One-Line Summary

> A singly linked list is a structure where  
> **“nodes of (data + next pointer) are connected in a line.”**  
> Insertion/deletion is `O(1)` when you know the position (or the previous node),  
> but searching/finding a position requires walking from `head`, which is `O(N)`,  
> and movement is only in one direction.

---

# Merging Linked Lists Summary

## 1. Problem Description

- We have two **sorted singly linked lists**, `SLL1` and `SLL2`, in ascending order.
- We want to merge them into **one new linked list**,
- And the **result** must also be in ascending order.

This is essentially the same as merging two sorted arrays,  
but the data structure is a **linked list** instead of an array.

---

## 2. Core Idea

1. Let `node1` start at `SLL1.head`,  
   and `node2` start at `SLL2.head`.
2. At each step:
   - Take the **smaller** value between `node1.data` and `node2.data`,
     and append it to the end of the `result` list.
   - Move the pointer (`node1` or `node2`) one step forward in the list from which you took the value.
3. If one list is exhausted,
   - Append all remaining nodes from the other list to `result`.
4. Because we always append the smaller of the two candidates,
   the `result` list remains sorted in ascending order.

---

## 3. Given Pseudocode Skeleton

The problem provides the following skeleton:

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

We need to determine what goes in `(1)`, `(2)`, and `(3)`.

---

## 4. Filling in the Missing Parts

### 4.1 `(2)` and `(3)` – Which List Should We Take the Value From?

- The `if` block uses a value from the **first list (`SLL1`, `node1`)**.
- The `else` block uses a value from the **second list (`SLL2`, `node2`)**.

So,

- `(2)` → `node1.data`
- `(3)` → `node2.data`

```pseudo
result.insert_end(node1.data)   # (2)
...
result.insert_end(node2.data)   # (3)
```

---

### 4.2 `(1)` – When Should We Use `node1`'s Value?

The `if` condition is:

```pseudo
if node2 == null or (1)
```

So there are two cases where we use `node1`'s value:

1. `node2 == null`  
   → The second list has already ended.  
     All remaining values must come from `node1`.
2. Both lists still have nodes, and  
   **`node1`'s value is less than or equal to `node2`'s value**.  
   → To keep the result sorted, we must insert `node1.data` first.

Therefore, (1) can be written as:

```pseudo
(node1 != null and node1.data <= node2.data)
```

> We include `node1 != null` because we must not access `node1.data`  
> when `node1` is `null` (assuming short-circuit evaluation).

---

## 5. Completed Pseudocode

```pseudo
function solution(SLL1, SLL2)
    set result = empty SLL
    node1 = SLL1.head
    node2 = SLL2.head

    while node1 != null or node2 != null
        if node2 == null or (node1 != null and node1.data <= node2.data)
            # Take from SLL1 when its value is smaller/equal,
            # or when SLL2 is already exhausted.
            result.insert_end(node1.data)
            node1 = node1.next
        else
            # Otherwise take from SLL2
            result.insert_end(node2.data)
            node2 = node2.next

    return result
```

If the lengths of the lists are `N` and `M`:

- Each node is visited exactly once,
- So the time complexity is `O(N + M)`.

---