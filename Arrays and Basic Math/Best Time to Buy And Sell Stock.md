# Best Time to Buy and Sell Stock

[LeetCode Problem Link](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

## Problem Statement

You are given an array `prices` where `prices[i]` is the price of a given stock on the `i`th day.

You want to maximize your profit by choosing a **single day** to buy one stock and choosing a **different day in the future** to sell that stock.

Return *the maximum profit you can achieve from this transaction*. If you cannot achieve any profit, return `0`.

## Examples

### Example 1:

```plaintext
Input: prices = [7,1,5,3,6,4]
Output: 5
Explanation: Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6-1 = 5.
Note that buying on day 2 and selling on day 1 is not allowed because you must buy before you sell.
```

### Example 2:

```plaintext
Input: prices = [7,6,4,3,1]
Output: 0
Explanation: In this case, no transactions are done and the max profit = 0.
```

## Constraints

*   `1 <= prices.length <= 10^5`
*   `0 <= prices[i] <= 10^4`

## Solutions

### Approach 1: Brute Force

The brute force approach involves iterating through each day `i` to buy stock and trying to sell it on every subsequent day `j` (where `j > i`). We calculate the profit for every valid pair `(i, j)` and assume the maximum profit found.

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        max_profit = 0
        n = len(prices)
        for i in range(n):
            for j in range(i + 1, n):
                profit = prices[j] - prices[i]
                if profit > max_profit:
                    max_profit = profit
        return max_profit
```

#### Complexity Analysis

*   **Time complexity**: $O(n^2)$. We have two nested loops. The inner loop runs $n-1$, $n-2$, ..., $1$ times. The sum of the first $n-1$ integers is $\frac{n(n-1)}{2}$, which is $O(n^2)$.
*   **Space complexity**: $O(1)$. We only use a constant amount of extra space (`max_profit`).

### Approach 2: One Pass

We can solve this problem in a single pass. As we iterate through the array, we need to keep track of the minimum price encountered so far (`min_price`) and the maximum profit we can get (`max_profit`).

For each price in the array:
1.  If the current price is lower than our `min_price`, we update `min_price`. This essentially means we've found a better day to buy.
2.  If the current price is not lower, we calculate the profit if we were to sell today (`price - min_price`) and update `max_profit` if this is higher than what we've seen so far.

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        min_price = float('inf')
        max_profit = 0
        
        for price in prices:
            if price < min_price:
                min_price = price
            elif price - min_price > max_profit:
                max_profit = price - min_price
                
        return max_profit
```

#### Complexity Analysis

*   **Time complexity**: $O(n)$. We only iterate through the `prices` array once.
*   **Space complexity**: $O(1)$. We only need two variables (`min_price` and `max_profit`) to store the state.
