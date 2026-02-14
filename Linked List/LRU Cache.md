# LRU Cache

[LeetCode Problem Link](https://leetcode.com/problems/lru-cache/description/)

## Problem Statement

Design a data structure that follows the constraints of a **Least Recently Used (LRU)** cache.

Implement the `LRUCache` class:

*   `LRUCache(int capacity)` — Initialize the LRU cache with positive size `capacity`.
*   `int get(int key)` — Return the value of the key if the key exists, otherwise return `-1`.
*   `void put(int key, int value)` — Update the value of the key if the key exists. Otherwise, add the key-value pair to the cache. If the number of keys exceeds the `capacity` from this operation, evict the least recently used key.

The functions `get` and `put` must each run in **O(1)** average time complexity.

## Examples

### Example 1:

```plaintext
Input:
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]

Output:
[null, null, null, 1, null, -1, null, -1, 3, 4]

Explanation:
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // cache is {1=1}
lRUCache.put(2, 2); // cache is {1=1, 2=2}
lRUCache.get(1);    // return 1
lRUCache.put(3, 3); // LRU key was 2, evicts key 2, cache is {1=1, 3=3}
lRUCache.get(2);    // returns -1 (not found)
lRUCache.put(4, 4); // LRU key was 1, evicts key 1, cache is {4=4, 3=3}
lRUCache.get(1);    // return -1 (not found)
lRUCache.get(3);    // return 3
lRUCache.get(4);    // return 4
```

### Example 2:

```plaintext
Input:
["LRUCache", [2], "put", [1, 10], "get", [1], "put", [2, 20], "put", [3, 30], "get", [2], "get", [1]]

Output:
[null, null, 10, null, null, 20, -1]

Explanation:
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 10);  // cache: {1=10}
lRUCache.get(1);      // return 10
lRUCache.put(2, 20);  // cache: {1=10, 2=20}
lRUCache.put(3, 30);  // cache: {2=20, 3=30}, key=1 was evicted
lRUCache.get(2);      // returns 20
lRUCache.get(1);      // return -1 (not found)
```

## Constraints

*   `1 <= capacity <= 3000`
*   `0 <= key <= 10^4`
*   `0 <= value <= 10^5`
*   At most `2 * 10^5` calls will be made to `get` and `put`.

## Prerequisites

*   **Doubly Linked Lists** — Implementing nodes with prev/next pointers for O(1) insertion and removal from both ends
*   **Hash Maps** — Using dictionaries for O(1) key-to-node lookup to enable fast cache access
*   **Data Structure Design** — Combining multiple data structures (hash map + linked list) to achieve desired time complexity for all operations

## Solutions

### Approach 1: Brute Force

We store all `(key, value)` pairs in a list. To follow LRU behavior:

*   Whenever we access a key, we move it to the end of the list (most recently used).
*   When inserting a new key:
    *   If the key already exists → update its value and move it to the end.
    *   If the cache is full → remove the first element (least recently used).
    *   Then add the new key at the end.

```python
class LRUCache:

    def __init__(self, capacity: int):
        self.cache = []
        self.capacity = capacity

    def get(self, key: int) -> int:
        for i in range(len(self.cache)):
            if self.cache[i][0] == key:
                tmp = self.cache.pop(i)
                self.cache.append(tmp)
                return tmp[1]
        return -1

    def put(self, key: int, value: int) -> None:
        for i in range(len(self.cache)):
            if self.cache[i][0] == key:
                tmp = self.cache.pop(i)
                tmp[1] = value
                self.cache.append(tmp)
                return

        if self.capacity == len(self.cache):
            self.cache.pop(0)

        self.cache.append([key, value])
```

#### Complexity Analysis

*   **Time complexity**: $O(n)$ for each `put()` and `get()` operation.
*   **Space complexity**: $O(n)$

---

### Approach 2: Doubly Linked List + Hash Map

We combine a **Hash Map** (for O(1) lookup) with a **Doubly Linked List** (for O(1) insertion/removal and ordering).

*   The **most recently used** node is kept near the `right` dummy.
*   The **least recently used** node is kept near the `left` dummy.

**Key Operations:**

*   `get(key)` — Move the node to the right end (most recently used) and return its value.
*   `put(key, value)` — If key exists, update and move to right. If new and at capacity, remove the leftmost node (LRU) and insert the new node at the right.

Dummy `left` and `right` nodes simplify insert/remove logic.

```python
class Node:
    def __init__(self, key, val):
        self.key, self.val = key, val
        self.prev = self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.cap = capacity
        self.cache = {}  # map key to node

        self.left, self.right = Node(0, 0), Node(0, 0)
        self.left.next, self.right.prev = self.right, self.left

    def remove(self, node):
        prev, nxt = node.prev, node.next
        prev.next, nxt.prev = nxt, prev

    def insert(self, node):
        prev, nxt = self.right.prev, self.right
        prev.next = nxt.prev = node
        node.next, node.prev = nxt, prev

    def get(self, key: int) -> int:
        if key in self.cache:
            self.remove(self.cache[key])
            self.insert(self.cache[key])
            return self.cache[key].val
        return -1

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.remove(self.cache[key])
        self.cache[key] = Node(key, value)
        self.insert(self.cache[key])

        if len(self.cache) > self.cap:
            lru = self.left.next
            self.remove(lru)
            del self.cache[lru.key]
```

#### Complexity Analysis

*   **Time complexity**: $O(1)$ for each `put()` and `get()` operation.
*   **Space complexity**: $O(n)$

---

### Approach 3: Built-In OrderedDict

Python's `OrderedDict` maintains insertion order and provides `move_to_end()` and `popitem()` for O(1) reordering and eviction — a perfect fit for LRU.

```python
from collections import OrderedDict

class LRUCache:

    def __init__(self, capacity: int):
        self.cache = OrderedDict()
        self.cap = capacity

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value

        if len(self.cache) > self.cap:
            self.cache.popitem(last=False)
```

#### Complexity Analysis

*   **Time complexity**: $O(1)$ for each `put()` and `get()` operation.
*   **Space complexity**: $O(n)$

## Common Pitfalls

*   **Forgetting to Update on Get Operations** — `get()` must also update recency. Missing this breaks the LRU invariant and causes wrong evictions.
*   **Incorrect Doubly Linked List Pointer Updates** — When removing a node, you must update both `prev.next` and `next.prev`. When inserting, update all four pointers. Missing any corrupts the list.
*   **Not Storing Keys in List Nodes** — During eviction, you need to delete from both the linked list and the hash map. If nodes don't store their key, you can't find the hash map entry to remove. Always store the key in each node.
