---
title: "Product of Array Except Self: Both Ends at Once"
date: 2026-08-31 09:00:00 +0900
categories: [Algorithms]
tags: [leetcode, arrays, prefix-product, in-place, complexity]
---

[LC238 Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/) asks for an array where each cell holds the product of every element *except* the one at that index — without division, in `O(n)`.

The structure that makes this tractable:

```
answer[i] = (product of everything left of i) × (product of everything right of i)
```

No cell needs to know the total. It needs a **prefix** and a **suffix**.

## The usual shape

The standard write-up runs two passes over the output array:

```python
# pass 1 — left to right, fill in prefixes
prefix = 1
for i in range(n):
    answer[i] = prefix
    prefix *= nums[i]

# pass 2 — right to left, multiply in suffixes
suffix = 1
for i in range(n - 1, -1, -1):
    answer[i] *= suffix
    suffix *= nums[i]
```

This is already optimal on the counts that matter: `O(n)` time, and `O(1)` extra space if the output array is excluded, which the problem allows.

## Collapsing it into one loop

The two passes never contend for the same accumulator — `prefix` only moves forward, `suffix` only moves backward. So they can share a loop:

```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        answer, suffix, prefix = [1] * len(nums), 1, 1
        for i in range(len(nums)):
            answer[i] *= prefix
            prefix *= nums[i]
            answer[-i - 1] *= suffix
            suffix *= nums[-i - 1]
        return answer
```

Each iteration writes to two cells: `answer[i]` from the left, `answer[-i-1]` from the right.

## Why writing both ends is safe

The part worth pausing on: the two writes can land on the *same* cell, and it still works.

For index `j`, across the whole loop:

- the forward write happens exactly once, at `i = j`, contributing the product of `nums[0..j-1]`
- the backward write happens exactly once, at `i = n-1-j`, contributing the product of `nums[j+1..n-1]`

Every cell is touched exactly twice, once by each direction. When `n` is odd, the middle cell takes both writes in the same iteration — and since multiplication is commutative and the two accumulators are independent, the order does not matter. The cell ends up as `prefix × suffix` either way.

The `answer` array is initialized to `1` precisely so both writes can be `*=` rather than one assignment and one multiply.

## What this does and does not buy

It is worth being honest about the win. The loop body doubled, so this performs the same `2n` multiply-and-write operations as the two-pass version. **The complexity is identical — `O(n)` time, `O(1)` extra space.** Nothing was saved asymptotically.

What changes is that the array is traversed once instead of twice, so both ends stay warm in cache, and there is one loop to read instead of two. That is a constant-factor and readability argument, not a complexity one. On LeetCode's timing noise it is not a claim worth making at all.

## The takeaway

The transferable idea is not the trick. It is the check that made the trick safe: **two accumulators can share a loop when neither reads what the other writes.** `prefix` only ever reads `nums` going forward, `suffix` only ever reads `nums` going backward, and `answer` is write-only for both. Once that independence is established, merging the passes is mechanical.

When a solution has two passes over the same array, it is worth asking whether they are actually coupled — often they only look sequential because that is how the explanation was written.
