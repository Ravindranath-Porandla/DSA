# Number of Islands

[LeetCode Problem Link](https://leetcode.com/problems/number-of-islands/description/)

## Problem Statement

Given an `m x n` 2D binary grid `grid` which represents a map of `'1'`s (land) and `'0'`s (water), return the **number of islands**.

An island is surrounded by water and is formed by connecting adjacent lands **horizontally or vertically**. You may assume all four edges of the grid are surrounded by water.

## Examples

### Example 1:

```plaintext
Input: grid = [
  ["1","1","1","1","0"],
  ["1","1","0","1","0"],
  ["1","1","0","0","0"],
  ["0","0","0","0","0"]
]
Output: 1
```

### Example 2:

```plaintext
Input: grid = [
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
Output: 3
```

### Example 3:

```plaintext
Input: grid = [
  ["1","1","0","0","1"],
  ["1","1","0","0","1"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
Output: 4
```

## Constraints

*   `m == grid.length`
*   `n == grid[i].length`
*   `1 <= m, n <= 300`
*   `grid[i][j]` is `'0'` or `'1'`.

## Prerequisites

*   **2D Arrays/Matrices** — Navigating and manipulating grid-based data structures
*   **Depth First Search (DFS)** — Recursive traversal to explore all connected components
*   **Breadth First Search (BFS)** — Level-by-level traversal using a queue
*   **Union-Find (Disjoint Set Union)** — Data structure for efficiently tracking connected components

## Solutions

### Approach 1: Depth First Search (DFS)

Think of the grid as a map where `'1'` is land and `'0'` is water. An island is a group of connected land cells (up, down, left, right). Whenever we find a land cell that hasn't been visited, we start a DFS to **sink** the entire island by marking all its connected land as `'0'`. Each DFS call corresponds to one island.

**Algorithm:**
1.  Iterate through every cell in the grid.
2.  When a cell with value `'1'` is found, increment the island count and run DFS from that cell.
3.  In DFS: if the cell is out of bounds or is `'0'`, return. Otherwise, mark it as `'0'` and recursively explore all 4 directions.

```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        directions = [[1, 0], [-1, 0], [0, 1], [0, -1]]
        ROWS, COLS = len(grid), len(grid[0])
        islands = 0

        def dfs(r, c):
            if (r < 0 or c < 0 or r >= ROWS or
                c >= COLS or grid[r][c] == "0"
            ):
                return

            grid[r][c] = "0"
            for dr, dc in directions:
                dfs(r + dr, c + dc)

        for r in range(ROWS):
            for c in range(COLS):
                if grid[r][c] == "1":
                    dfs(r, c)
                    islands += 1

        return islands
```

#### Complexity Analysis

*   **Time complexity**: $O(m \times n)$ — Each cell is visited at most once.
*   **Space complexity**: $O(m \times n)$ — Recursion stack in worst case (all land).

---

### Approach 2: Breadth First Search (BFS)

When we encounter a land cell, we use BFS to visit all connected land cells and mark them as water, ensuring the same island is not counted again.

**Algorithm:**
1.  Traverse every cell in the grid.
2.  When a `'1'` cell is found, increment the island count and start BFS from that cell.
3.  In BFS: push the starting cell into a queue and mark it as `'0'`. While the queue is not empty, pop a cell, explore its 4 neighbors, and enqueue any land neighbors (marking them `'0'`).

```python
from collections import deque

class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        directions = [[1, 0], [-1, 0], [0, 1], [0, -1]]
        ROWS, COLS = len(grid), len(grid[0])
        islands = 0

        def bfs(r, c):
            q = deque()
            grid[r][c] = "0"
            q.append((r, c))

            while q:
                row, col = q.popleft()
                for dr, dc in directions:
                    nr, nc = dr + row, dc + col
                    if (nr < 0 or nc < 0 or nr >= ROWS or
                        nc >= COLS or grid[nr][nc] == "0"
                    ):
                        continue
                    q.append((nr, nc))
                    grid[nr][nc] = "0"

        for r in range(ROWS):
            for c in range(COLS):
                if grid[r][c] == "1":
                    bfs(r, c)
                    islands += 1

        return islands
```

#### Complexity Analysis

*   **Time complexity**: $O(m \times n)$ — Each cell is visited at most once.
*   **Space complexity**: $O(m \times n)$ — Queue size in worst case.

---

### Approach 3: Disjoint Set Union (Union-Find)

Think of every land cell (`'1'`) as its own separate island initially. When two land cells are adjacent, they belong to the same island, so we merge them. Each successful merge reduces the total island count by 1.

**Algorithm:**
1.  Initialize DSU for all cells. For each land cell, increment island count.
2.  Traverse the grid and for each land cell, check its 4 neighbors.
3.  If a neighbor is also land, union the two cells. If the union actually merges two different components, decrement island count.
4.  Return the remaining island count.

```python
class DSU:
    def __init__(self, n):
        self.Parent = list(range(n + 1))
        self.Size = [1] * (n + 1)

    def find(self, node):
        if self.Parent[node] != node:
            self.Parent[node] = self.find(self.Parent[node])
        return self.Parent[node]

    def union(self, u, v):
        pu = self.find(u)
        pv = self.find(v)
        if pu == pv:
            return False
        if self.Size[pu] >= self.Size[pv]:
            self.Size[pu] += self.Size[pv]
            self.Parent[pv] = pu
        else:
            self.Size[pv] += self.Size[pu]
            self.Parent[pu] = pv
        return True

class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        ROWS, COLS = len(grid), len(grid[0])
        dsu = DSU(ROWS * COLS)

        def index(r, c):
            return r * COLS + c

        directions = [(1, 0), (-1, 0), (0, 1), (0, -1)]
        islands = 0

        for r in range(ROWS):
            for c in range(COLS):
                if grid[r][c] == '1':
                    islands += 1
                    for dr, dc in directions:
                        nr, nc = r + dr, c + dc
                        if (nr < 0 or nc < 0 or nr >= ROWS or
                            nc >= COLS or grid[nr][nc] == "0"
                        ):
                            continue

                        if dsu.union(index(r, c), index(nr, nc)):
                            islands -= 1

        return islands
```

#### Complexity Analysis

*   **Time complexity**: $O(m \times n)$ — Nearly O(1) per union/find with path compression and union by size.
*   **Space complexity**: $O(m \times n)$ — For the DSU parent and size arrays.

## Common Pitfalls

*   **Not Marking Cells as Visited** — Forgetting to mark cells as `'0'` (or using a visited set) causes infinite loops as DFS/BFS keeps revisiting the same cells.
*   **Counting Every Land Cell Instead of Islands** — Each island should be counted once when you *first* encounter it, not for every `'1'` cell.
*   **Incorrect Boundary Checks** — Always verify indices are within bounds *before* accessing the grid. Check bounds first, then check the cell value.
*   **Diagonal Connections** — This problem only considers horizontal and vertical connections (4-directional). Including diagonal neighbors incorrectly merges separate islands.
