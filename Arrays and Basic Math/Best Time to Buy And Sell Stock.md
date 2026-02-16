# Best Time to Buy and Sell Stock

[LeetCode Problem Link](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/description/)

## Problem Statement

You are given an integer array `prices` where `prices[i]` is the price of a given stock on the `i`th day.

You may choose a **single day** to buy one stock and choose a **different day in the future** to sell it.

Return the **maximum profit** you can achieve. You may choose to not make any transactions, in which case the profit would be `0`.

## Examples

### Example 1:

```plaintext
Input: prices = [10,1,5,6,7,1]
Output: 6
Explanation: Buy prices[1] and sell prices[4], profit = 7 - 1 = 6.
```

### Example 2:

```plaintext
Input: prices = [10,8,7,5,2]
Output: 0
Explanation: No profitable transactions can be made, thus the max profit is 0.
```

### Example 3:

```plaintext
Input: prices = [7,1,5,3,6,4]
Output: 5
Explanation: Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6 - 1 = 5.
Note that buying on day 2 and selling on day 1 is not allowed because you must buy before you sell.
```

### Example 4:

```plaintext
Input: prices = [7,6,4,3,1]
Output: 0
Explanation: In this case, no transactions are done and the max profit = 0.
```

## Constraints

*   `1 <= prices.length <= 10^5`
*   `0 <= prices[i] <= 10^4`

## Prerequisites

*   **Arrays** — Iterating through elements and tracking values
*   **Two Pointers** — Using multiple pointers to track buy and sell positions
*   **Greedy Algorithms** — Making locally optimal choices (tracking minimum buy price) for global optimization

## Solutions

### Approach 1: Brute Force

**Intuition:**
The brute-force approach checks every possible buy–sell pair. For each day, we pretend to buy the stock, and then we look at all the future days to see what the best selling price would be. Among all these profits, we keep the highest one.

**Algorithm:**
1.  Initialize `res = 0` to store the maximum profit.
2.  Loop through each day `i` as the buy day.
3.  For each buy day, loop through each day `j > i` as the sell day.
4.  Calculate the profit `prices[j] - prices[i]` and update `res`.
5.  Return `res` after checking all pairs.

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        res = 0
        for i in range(len(prices)):
            buy = prices[i]
            for j in range(i + 1, len(prices)):
                sell = prices[j]
                res = max(res, sell - buy)
        return res
```

#### Complexity Analysis

*   **Time complexity**: $O(n^2)$ — Two nested loops checking all buy-sell pairs.
*   **Space complexity**: $O(1)$ — Only a constant amount of extra space.

---

### Approach 2: Two Pointers

**Intuition:**
We want to buy at a low price and sell at a higher price that comes after it. Using two pointers helps us track this efficiently:

*   `l` is the **buy day** (looking for the lowest price)
*   `r` is the **sell day** (looking for a higher price)

If the price at `r` is higher than at `l`, we can make a profit — so we update the maximum. If the price at `r` is lower, then `r` becomes the new `l` because a cheaper buying price is always better.

By moving the pointers this way, we scan the list once and always keep the best buying opportunity.

**Algorithm:**
1.  Set two pointers: `l = 0` (buy day), `r = 1` (sell day).
2.  Initialize `maxP = 0` to track maximum profit.
3.  While `r` is within the array:
    *   If `prices[l] < prices[r]`, compute the profit and update `maxP`.
    *   Otherwise, move `l` to `r` (we found a cheaper buy price).
    *   Move `r` to the next day.
4.  Return `maxP`.

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        l, r = 0, 1
        maxP = 0

        while r < len(prices):
            if prices[l] < prices[r]:
                profit = prices[r] - prices[l]
                maxP = max(maxP, profit)
            else:
                l = r
            r += 1
        return maxP
```

#### Complexity Analysis

*   **Time complexity**: $O(n)$ — Single pass through the array.
*   **Space complexity**: $O(1)$ — Only pointer variables and profit tracker.

---

### Approach 3: Dynamic Programming (Greedy)

**Intuition:**
As we scan through the prices, we keep track of two things:

*   The **lowest price so far** → this is the best day to buy.
*   The **best profit so far** → selling today minus the lowest buy price seen earlier.

At each price, we imagine selling on that day. The profit would be: `current price – lowest price seen so far`.

We then update:
*   The **maximum profit** if we found a better one.
*   The **lowest price** if we find a cheaper one.

This way, we make the optimal buy–sell decision in one simple pass.

**Algorithm:**
1.  Initialize `minBuy` as the first price, `maxP = 0` for the best profit.
2.  Loop through each price `sell`:
    *   Update `maxP` with `sell - minBuy`.
    *   Update `minBuy` if we find a smaller price.
3.  Return `maxP` after scanning all days.

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        maxP = 0
        minBuy = prices[0]

        for sell in prices:
            maxP = max(maxP, sell - minBuy)
            minBuy = min(minBuy, sell)
        return maxP
```

#### Complexity Analysis

*   **Time complexity**: $O(n)$ — Single pass through the array.
*   **Space complexity**: $O(1)$ — Only two variables to track state.

## Common Pitfalls

*   **Selling Before Buying** — The sell day must come **after** the buy day. Calculating `prices[i] - prices[j]` where `j > i` means you're selling in the past, which is invalid.

    ```python
    # WRONG: selling before buying
    for i in range(len(prices)):
        for j in range(i):  # j < i means selling earlier
            profit = prices[j] - prices[i]  # Backwards

    # CORRECT: sell after buy
    for i in range(len(prices)):
        for j in range(i + 1, len(prices)):  # j > i
            profit = prices[j] - prices[i]
    ```

*   **Returning Negative Profit** — If prices only decrease, the maximum profit is `0` (don't trade), not a negative number. Always ensure the result is at least `0`.

    ```python
    # WRONG: can return negative
    return maxPrice - minPrice  # Could be negative if maxPrice found before minPrice

    # CORRECT: initialize to 0, only update when profit is positive
    maxP = 0
    maxP = max(maxP, sell - minBuy)
    ```
