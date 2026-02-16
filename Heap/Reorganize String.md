# Reorganize String

[LeetCode Problem Link](https://leetcode.com/problems/reorganize-string/description/)

## Problem Statement

Given a string `s`, rearrange the characters of `s` so that any two **adjacent characters are not the same**.

Return any possible rearrangement of `s` or return `""` if not possible.

## Examples

### Example 1:

```plaintext
Input: s = "aab"
Output: "aba"
```

### Example 2:

```plaintext
Input: s = "aaab"
Output: ""
```

### Example 3:

```plaintext
Input: s = "axyy"
Output: "xyay"
```

### Example 4:

```plaintext
Input: s = "abbccdd"
Output: "abcdbcd"
```

### Example 5:

```plaintext
Input: s = "ccccd"
Output: ""
```

## Constraints

*   `1 <= s.length <= 500`
*   `s` consists of lowercase English letters.

## Prerequisites

*   **Frequency Counting** — Using arrays or hash maps to count character occurrences in a string
*   **Greedy Algorithms** — Making locally optimal choices (placing most frequent characters first) to achieve a global solution
*   **Heap / Priority Queue** — Efficiently retrieving the maximum frequency element for character placement

## Key Insight

> If any character appears more than `(n + 1) / 2` times (where `n = len(s)`), it is **impossible** to reorganize the string without adjacent duplicates. Otherwise, a valid rearrangement always exists.

## Solutions

### Approach 1: Frequency Count (Brute Force Greedy)

**Intuition:**
To avoid adjacent duplicates, we should always place the **most frequent** remaining character, then the **second most frequent**. This greedy approach works because alternating between the two most common characters maximizes our ability to separate identical characters.

**Algorithm:**
1.  Count the frequency of each character in a `freq` array of size 26.
2.  If the maximum frequency exceeds `(n + 1) // 2`, return an empty string — reorganization is impossible.
3.  While building the result:
    *   Find the character with the **highest frequency** (`maxIdx`) and append it to `res`.
    *   Decrement its count.
    *   If it still has remaining count, temporarily hide it (set to `-inf`) and find the **next highest frequency** character.
    *   Append the second character and restore the first character's count.
4.  Return the result string.

```python
class Solution:
    def reorganizeString(self, s: str) -> str:
        freq = [0] * 26
        for char in s:
            freq[ord(char) - ord('a')] += 1

        max_freq = max(freq)
        if max_freq > (len(s) + 1) // 2:
            return ""

        res = []
        while len(res) < len(s):
            maxIdx = freq.index(max(freq))
            char = chr(maxIdx + ord('a'))
            res.append(char)
            freq[maxIdx] -= 1
            if freq[maxIdx] == 0:
                continue

            tmp = freq[maxIdx]
            freq[maxIdx] = float("-inf")
            nextMaxIdx = freq.index(max(freq))
            char = chr(nextMaxIdx + ord('a'))
            res.append(char)
            freq[maxIdx] = tmp
            freq[nextMaxIdx] -= 1

        return ''.join(res)
```

#### Complexity Analysis

*   **Time complexity**: $O(n)$ — Each character is placed once; finding the max in a fixed-size array of 26 is $O(1)$.
*   **Space complexity**: $O(1)$ extra space (fixed 26-size array), $O(n)$ for the output string.

---

### Approach 2: Frequency Count with Max-Heap

**Intuition:**
A **max-heap** efficiently gives us the most frequent character at any time. We pop the top character, add it to the result, and then push it back with a decremented count — but only **after** processing the next character. This delay ensures we never place the same character twice in a row.

If the heap is empty but we still have a `prev` pending character (with remaining count), reorganization has failed.

**Algorithm:**
1.  Count character frequencies using `Counter` and build a max-heap of `(-count, character)` pairs (negated for max-heap behavior with Python's min-heap).
2.  Track a `prev` element that was just used and cannot be immediately reused.
3.  While the heap is not empty or `prev` exists:
    *   If `prev` exists but the heap is empty → return `""` (impossible).
    *   Pop the top element, append its character to `res`, and decrement its count.
    *   Push `prev` back into the heap if it exists (it's now safe to reuse).
    *   Set `prev` to the current element if its count is still > 0.
4.  Return the result string.

```python
from collections import Counter
import heapq

class Solution:
    def reorganizeString(self, s: str) -> str:
        count = Counter(s)
        maxHeap = [[-cnt, char] for char, cnt in count.items()]
        heapq.heapify(maxHeap)

        prev = None
        res = ""
        while maxHeap or prev:
            if prev and not maxHeap:
                return ""

            cnt, char = heapq.heappop(maxHeap)
            res += char
            cnt += 1  # cnt is negative, so +1 decrements the actual count

            if prev:
                heapq.heappush(maxHeap, prev)
                prev = None

            if cnt != 0:
                prev = [cnt, char]

        return res
```

#### Why the `prev` Trick Works

The `prev` variable acts as a one-step delay buffer:
*   After placing character `A`, we hold it in `prev` instead of re-inserting it.
*   On the next iteration, we place character `B` and *then* push `A` back.
*   This guarantees `A` can never be popped on two consecutive iterations.

#### Complexity Analysis

*   **Time complexity**: $O(n)$ — At most 26 unique characters in the heap, so each heap operation is $O(\log 26) = O(1)$. We process $n$ characters total.
*   **Space complexity**: $O(1)$ extra space (heap has at most 26 entries), $O(n)$ for the output string.

---

### Approach 3: Frequency Count (Index-Based Greedy)

**Intuition:**
Instead of building the string character by character, we can **place characters at alternating indices**. First, fill all **even indices** (0, 2, 4, ...) with the most frequent character. This guarantees no two identical characters are adjacent since they are separated by at least one position. Then fill the remaining positions with other characters, wrapping to **odd indices** (1, 3, 5, ...) when even slots are exhausted.

**Algorithm:**
1.  Count frequencies in `freq` and find the most frequent character at `max_idx`.
2.  If the max frequency exceeds `(n + 1) // 2`, return an empty string.
3.  Place the most frequent character at even indices `0, 2, 4, ...` until its count is exhausted.
4.  For all remaining characters:
    *   Place them at the current `idx`, incrementing by 2 each time.
    *   When `idx >= len(s)`, wrap to `idx = 1` and continue with odd positions.
5.  Return the result string.

**Visualization:**

```plaintext
s = "aabbc" → freq: a=2, b=2, c=1 → most frequent: 'a' (count=2)

Step 1: Place 'a' at even indices
  Index:  0   1   2   3   4
  Char:  [a] [ ] [a] [ ] [ ]   ← idx now at 4

Step 2: Place 'b' (count=2)
  Index:  0   1   2   3   4
  Char:  [a] [ ] [a] [ ] [b]   ← idx = 4 → 6 (≥5, wrap to 1)
  Char:  [a] [b] [a] [ ] [b]   ← idx = 1 → 3

Step 3: Place 'c' (count=1)
  Char:  [a] [b] [a] [c] [b]   ← idx = 3 → 5

Result: "abacb" ✓ (no adjacent duplicates)
```

```python
class Solution:
    def reorganizeString(self, s: str) -> str:
        freq = [0] * 26
        for char in s:
            freq[ord(char) - ord('a')] += 1

        max_idx = freq.index(max(freq))
        max_freq = freq[max_idx]
        if max_freq > (len(s) + 1) // 2:
            return ""

        res = [''] * len(s)
        idx = 0
        max_char = chr(max_idx + ord('a'))

        # Place the most frequent character at even indices first
        while freq[max_idx] > 0:
            res[idx] = max_char
            idx += 2
            freq[max_idx] -= 1

        # Place all remaining characters
        for i in range(26):
            while freq[i] > 0:
                if idx >= len(s):
                    idx = 1  # Wrap to odd indices
                res[idx] = chr(i + ord('a'))
                idx += 2
                freq[i] -= 1

        return ''.join(res)
```

#### Why Place the Most Frequent Character First?

By placing the most frequent character at even indices first, we guarantee it gets the maximum spacing. If we placed a less frequent character first, the most frequent one might not have enough spread-out positions and could end up adjacent to itself.

#### Complexity Analysis

*   **Time complexity**: $O(n)$ — Single pass to count, single pass to place.
*   **Space complexity**: $O(1)$ extra space (fixed 26-size array), $O(n)$ for the output string.

## Common Pitfalls

*   **Incorrect Impossibility Check** — The condition for impossibility is when the most frequent character appears more than `(n + 1) // 2` times. Using `n // 2` or forgetting the ceiling division leads to incorrect rejection of valid inputs (e.g., `"aab"` has `n=3` and max freq `2`, which equals `(3+1)//2 = 2` — valid!).

*   **Placing Characters at Wrong Indices in Greedy Approach** — Characters must be placed at even indices first (0, 2, 4, ...) before wrapping to odd indices (1, 3, 5, ...). Failing to reset the index to `1` after exceeding the array length results in index out-of-bounds errors or incorrect placements.

*   **Re-inserting the Same Character into the Heap Immediately** — In the heap-based approach, the just-used character must be **held back for one iteration** before reinsertion. Pushing it back immediately allows it to be selected again on the next pop, creating adjacent duplicates:

    ```python
    # WRONG: Adjacent duplicates possible
    cnt, char = heapq.heappop(maxHeap)
    res += char
    heapq.heappush(maxHeap, [cnt + 1, char])  # Can be popped again immediately!

    # CORRECT: Delay reinsertion
    if prev:
        heapq.heappush(maxHeap, prev)  # Push previous, not current
    prev = [cnt, char]  # Hold current back
    ```

*   **Not Handling the Single Character Remaining Case** — When the heap is empty but `prev` still has a character with remaining count, the reorganization is impossible. Forgetting this check leads to silently dropping characters from the output.
