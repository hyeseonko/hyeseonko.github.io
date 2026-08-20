---
title: "Top K Frequent Elements: Why Eviction Is Safe"
date: 2026-08-20 09:00:00 +0900
categories: [Algorithms]
tags: [leetcode, heap, bucket-sort, streaming, complexity]
---

[LC347 Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) asks for the `k` most frequent values in an array.

## The real question

The problem splits into two steps, and only one of them is interesting:

1. Count how often each number appears.
2. Pick the top `k` by count.

Step 1 is a `Counter`. There is nothing to decide. The whole problem is step 2 — and the follow-up note pins down what's wanted:

> Your algorithm's time complexity must be better than O(n log n).

So "count, then sort by count" is accepted but isn't the intended answer. Sorting builds a total order over every distinct value when we only asked for the top `k`. There are two ways to stop paying for that.

**Keep a bounded structure.** Maintain a min-heap of size `k`; each new candidate either displaces the weakest member or is dropped. `O(n log k)`.

```python
import heapq
from collections import Counter

def topKFrequent(nums, k):
    count = Counter(nums)
    return heapq.nlargest(k, count, key=count.get)
```

**Remove the sort entirely.** How large can a frequency be? At least 1, at most `len(nums)`. Frequencies are small bounded integers — which means they can be array *indices* rather than sort keys. Bucket the values by their count and sweep from the top.

```python
def topKFrequent(nums, k):
    count = Counter(nums)
    buckets = [[] for _ in range(len(nums) + 1)]   # index = frequency
    for value, freq in count.items():
        buckets[freq].append(value)

    out = []
    for freq in range(len(nums), 0, -1):
        out.extend(buckets[freq])
        if len(out) >= k:
            return out[:k]
    return out
```

`O(n)` time and space. The `+ 1` is load-bearing: a value can appear `n` times, so `buckets[n]` has to exist.

## The doubt worth having

The heap version bothered me. If I only ever keep `k` candidates, what stops a value I threw away from deserving a place later, once more of the array has gone by?

Nothing — *if* counts are allowed to change after a candidate has been judged. What makes it safe here is the two-phase structure: **counting finishes before selecting starts.** By the time a number enters the heap it carries its final frequency, and it enters exactly once. There is no "later" in which it could grow.

That, plus a property of the heap itself: in a size-`k` min-heap, the root only ever moves up. Take `k = 2` and candidates `(5,a) (3,b) (1,c) (4,d)`:

| step | heap | root |
|---|---|---|
| `a=5` | `{a:5}` | 5 |
| `b=3` | `{a:5, b:3}` | 3 |
| `c=1` | `1 < 3` → dropped | 3 |
| `d=4` | `4 > 3` → evicts `b` | **4** |

When `c` was dropped, `k` candidates already beat it. Every future root is at least the current one, so `c` can never clear the bar again. Eviction is a decision that never needs revisiting.

## Where the doubt is right

Fold the counting into the selecting and it becomes a real bug. Update the heap while walking the raw array, with `k = 1` and `nums = [1, 2, 2, 1, 1]`:

- `1` sits at count 1 when `2` reaches count 2, so `1` is evicted
- `1` finishes at count 3 — wrong answer

Same heap, same eviction rule, broken. The invariant that made eviction safe was never "heaps are correct." It was "a candidate's key is final when it is judged."

## Streaming is a different problem

"Keep only `k` things" sounds like it should be a memory win on an unbounded stream. For exact answers it isn't: counts keep moving, so you need a counter for every distinct key — `O(distinct)` memory, with the heap reduced to a final selection step. Exact top-`k` in `O(k)` space isn't on the menu.

What sublinear space buys you is an approximation: Count-Min Sketch for frequency estimates, or Space-Saving / Misra-Gries for heavy hitters. Both accept a bounded error in exchange for memory that doesn't grow with the number of distinct keys.

## Takeaway

Bucketing is the answer to the constraint as written. The heap is the better answer when `k` is far smaller than the number of distinct values, or when memory rather than time is the binding constraint — and it degrades toward `O(n log n)` as `k` approaches `n`, which is precisely where bucketing wins.

The part worth keeping isn't which one is faster. It's that dropping a candidate is only safe when its score can no longer change, and that the two-phase split is what buys that guarantee. I nearly reached for the heap without being able to say why the eviction was sound.
