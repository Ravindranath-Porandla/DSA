# Two Sum

[LeetCode Problem Link](https://leetcode.com/problems/two-sum/description/)

## Problem Statement

Given an array of integers `nums` and an integer `target`, return the indices `i` and `j` such that `nums[i] + nums[j] == target` and `i != j`.

You may assume that every input has **exactly one** pair of indices `i` and `j` that satisfy the condition.

Return the answer with the smaller index first.

## Examples

### Example 1:

```plaintext
Input: nums = [3,4,5,6], target = 7
Output: [0,1]
Explanation: nums[0] + nums[1] == 7, so we return [0, 1].
```

### Example 2:

```plaintext
Input: nums = [4,5,6], target = 10
Output: [0,2]
```

### Example 3:

```plaintext
Input: nums = [5,5], target = 10
Output: [0,1]
```

### Example 4:

```plaintext
Input: nums = [2,7,11,15], target = 9
Output: [0,1]
Explanation: Because nums[0] + nums[1] == 9, we return [0, 1].
```

## Constraints

*   `2 <= nums.length <= 10^4`
*   `-10^9 <= nums[i] <= 10^9`
*   `-10^9 <= target <= 10^9`
*   Only one valid answer exists.

## Prerequisites

*   **Hash Maps** — Using dictionaries to store values and their indices for O(1) lookup of complements
*   **Two Pointers** — Understanding how sorting enables the two-pointer approach for finding pairs
*   **Array Traversal** — Iterating through arrays while tracking indices and values

## Solutions

### Approach 1: Brute Force

**Intuition:**
We can check every pair of different elements in the array and return the first pair that sums up to the target. This is the most intuitive approach but not the most efficient.

**Algorithm:**
1.  Iterate through the array with two nested loops using indices `i` and `j` to check every pair of different elements.
2.  If the sum of the pair equals the target, return the indices of the pair.
3.  If no such pair is found, return an empty array.
4.  There is guaranteed to be exactly one solution, so we will never return an empty array.

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        for i in range(len(nums)):
            for j in range(i + 1, len(nums)):
                if nums[i] + nums[j] == target:
                    return [i, j]
        return []
```

#### Complexity Analysis

*   **Time complexity**: $O(n^2)$ — For each element, we loop through the rest of the array to find its complement.
*   **Space complexity**: $O(1)$ — No extra space is used beyond a few variables.

---

### Approach 2: Sorting + Two Pointers

**Intuition:**
We can sort the array and use two pointers to find the two numbers that sum up to the target. After sorting, if the current sum is too small we move the left pointer right, and if it's too large we move the right pointer left. This is more efficient than brute force.

Since sorting changes element positions, we need to store the **original indices** alongside the values before sorting.

**Algorithm:**
1.  Create a copy of the array with each element paired with its original index.
2.  Sort this array by value in ascending order.
3.  Initialize two pointers: `i` at the beginning and `j` at the end.
4.  Compute the current sum:
    *   If equal to target → return the original indices (smaller index first).
    *   If less than target → move `i` right to increase the sum.
    *   If greater than target → move `j` left to decrease the sum.

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        A = []
        for i, num in enumerate(nums):
            A.append([num, i])

        A.sort()
        i, j = 0, len(nums) - 1
        while i < j:
            cur = A[i][0] + A[j][0]
            if cur == target:
                return [min(A[i][1], A[j][1]),
                        max(A[i][1], A[j][1])]
            elif cur < target:
                i += 1
            else:
                j -= 1
        return []
```

#### Complexity Analysis

*   **Time complexity**: $O(n \log n)$ — Dominated by the sorting step. The two-pointer scan itself is $O(n)$.
*   **Space complexity**: $O(n)$ — For storing the value-index pairs.

---

### Approach 3: Hash Map (Two Pass)

**Intuition:**
We can use a hash map to store the value and index of each element in the array. Then, we iterate through the array and check if the **complement** (`target - nums[i]`) exists in the hash map. The complement must be at a different index, because we can't use the same element twice.

By using a hash map, we achieve $O(1)$ lookup time for each complement check.

**Algorithm:**
1.  **First pass:** Build a hash map mapping each value to its index.
2.  **Second pass:** For each element, compute the complement `target - nums[i]`.
3.  Check if the complement exists in the hash map and has a different index.
4.  If found, return both indices.

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        indices = {}  # val -> index

        for i, n in enumerate(nums):
            indices[n] = i

        for i, n in enumerate(nums):
            diff = target - n
            if diff in indices and indices[diff] != i:
                return [i, indices[diff]]
        return []
```

#### Complexity Analysis

*   **Time complexity**: $O(n)$ — We traverse the array twice. Hash map lookups are $O(1)$.
*   **Space complexity**: $O(n)$ — The hash map stores at most $n$ elements.

---

### Approach 4: Hash Map (One Pass)

**Intuition:**
We can solve the problem in a **single pass** by checking if the complement exists in the hash map *before* inserting the current element. If it does, we return the indices immediately. If not, we store the current element and move on.

This guarantees we never use the same element twice (since we only search among previously seen elements) and we still check every possible pair.

**Algorithm:**
1.  Create an empty hash map to store values and their indices.
2.  Iterate through the array with index `i`:
    *   Compute the complement: `diff = target - nums[i]`.
    *   If `diff` exists in the hash map, return `[prevMap[diff], i]`.
    *   Otherwise, store `nums[i] -> i` in the hash map.

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        prevMap = {}  # val -> index

        for i, n in enumerate(nums):
            diff = target - n
            if diff in prevMap:
                return [prevMap[diff], i]
            prevMap[n] = i
```

#### Why One Pass Works

By inserting elements *after* checking for their complement, we ensure:
*   We never match an element with itself.
*   The returned indices are always in ascending order (`prevMap[diff] < i`).
*   We find the answer as soon as the second element of the pair is encountered.

#### Complexity Analysis

*   **Time complexity**: $O(n)$ — Single pass through the array. Each hash map lookup is $O(1)$.
*   **Space complexity**: $O(n)$ — The hash map stores at most $n$ elements.

## Common Pitfalls

*   **Using the Same Element Twice** — You cannot use the same element twice to form a pair. When using a hash map, ensure the found index differs from the current index, or use the one-pass approach where you only look at previously seen elements.

    ```python
    # WRONG: might return [i, i] when nums[i] * 2 == target
    if diff in indices:
        return [i, indices[diff]]

    # CORRECT: ensure different indices
    if diff in indices and indices[diff] != i:
        return [i, indices[diff]]
    ```

*   **Returning Values Instead of Indices** — The problem asks for **indices**, not the values themselves. A common mistake is returning the two numbers that sum to the target rather than their positions in the array.

*   **Handling Duplicate Values** — When building a hash map with values as keys, duplicate values overwrite earlier indices. In the two-pass approach, this still works because you check `indices[diff] != i`. In the one-pass approach, you check for the complement **before** inserting the current element.

*   **Wrong Complement Calculation** — The complement should be `target - nums[i]`, not `nums[i] - target`. Getting this backwards will search for the wrong value.

    ```python
    # WRONG: searching for the wrong complement
    diff = nums[i] - target   # e.g., nums[i]=3, target=7 → diff=-4 (wrong!)

    # CORRECT
    diff = target - nums[i]   # e.g., nums[i]=3, target=7 → diff=4 (correct!)
    ```
