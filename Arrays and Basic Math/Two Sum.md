# Two Sum

[LeetCode Problem Link](https://leetcode.com/problems/two-sum/description/)

## Problem Statement

Given an array of integers `nums` and an integer `target`, return indices of the two numbers such that they add up to `target`.

You may assume that each input would have **exactly one solution**, and you may not use the same element twice.

You can return the answer in any order.

## Examples

### Example 1:

```plaintext
Input: nums = [2,7,11,15], target = 9
Output: [0,1]
Explanation: Because nums[0] + nums[1] == 9, we return [0, 1].
```

### Example 2:

```plaintext
Input: nums = [3,2,4], target = 6
Output: [1,2]
```

### Example 3:

```plaintext
Input: nums = [3,3], target = 6
Output: [0,1]
```

## Constraints

*   `2 <= nums.length <= 10^4`
*   `-10^9 <= nums[i] <= 10^9`
*   `-10^9 <= target <= 10^9`
*   Only one valid answer exists.

## Solutions

### Approach 1: Brute Force

The brute force approach involves iterating through each element `x` and checking if there is another value that equals to `target - x`.

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        n = len(nums)
        for i in range(n - 1):
            for j in range(i + 1, n):
                if nums[i] + nums[j] == target:
                    return [i, j]
        return []  # No solution found
```

#### Complexity Analysis

*   **Time complexity**: $O(n^2)$. For each element, we try to find its complement by looping through the rest of the array which takes $O(n)$ time. Therefore, the time complexity is $O(n^2)$.
*   **Space complexity**: $O(1)$. The space required does not depend on the size of the input array, so only constant space is used.

### Approach 2: Two-Pass Hash Table

In this approach, we perform two passes. In the first pass, we add each element's value and its index to the hash table. Then, in the second pass, we check if each element's complement (`target - nums[i]`) exists in the table. Beware that the complement must not be `nums[i]` itself.

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        numMap = {}
        n = len(nums)

        # Build the hash table
        for i in range(n):
            numMap[nums[i]] = i

        # Find the complement
        for i in range(n):
            complement = target - nums[i]
            if complement in numMap and numMap[complement] != i:
                return [i, numMap[complement]]

        return []  # No solution found
```

#### Complexity Analysis

*   **Time complexity**: $O(n)$. We traverse the list containing $n$ elements exactly twice. Since the hash table reduces the lookup time to $O(1)$, the overall time complexity is $O(n)$.
*   **Space complexity**: $O(n)$. The extra space required depends on the number of items stored in the hash table, which stores exactly $n$ elements.

### Approach 3: One-Pass Hash Table

It turns out we can do it in one-pass. While we iterate and inserting elements into the table, we also look back to check if current element's complement already exists in the table. If it exists, we have found a solution and return immediately.

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        numMap = {}
        n = len(nums)

        for i in range(n):
            complement = target - nums[i]
            if complement in numMap:
                return [numMap[complement], i]
            numMap[nums[i]] = i

        return []  # No solution found
```

#### Complexity Analysis

*   **Time complexity**: $O(n)$. We traverse the list containing $n$ elements only once. Each lookup in the table costs only $O(1)$ time.
*   **Space complexity**: $O(n)$. The extra space required depends on the number of items stored in the hash table, which stores at most $n$ elements.
