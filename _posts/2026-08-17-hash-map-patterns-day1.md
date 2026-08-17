---
title: "[TIL] Hash Maps: Trading Memory for Lookup Speed"
date: 2026-08-17 09:00:00 +0900
categories: [TIL, Algorithms]
tags: [leetcode, hash-map, arrays, data-structures]
---

Restarting my daily study log. Day 1: three warm-up problems from the Arrays & Hashing pattern — and they all lean on the exact same trick underneath their different phrasing.

## Problems

Each problem's naive solution rescans the array (or a slice of it) for every element, giving O(n²) time. In every case, a hash map (or hash set) removes the rescan by remembering what's already been seen:

- **Set membership** — [LC217 Contains Duplicate](https://leetcode.com/problems/contains-duplicate/): instead of checking each element against every other element, insert into a set as you go and check membership before inserting. One pass, O(n). Sorting would be O(n log n) for no benefit here.
- **Frequency counting** — [LC242 Valid Anagram](https://leetcode.com/problems/valid-anagram/): instead of sorting both strings to compare, count character frequencies for one string, then decrement while scanning the other. Two strings are anagrams iff all counts return to zero. `Counter(s) == Counter(t)` is the one-liner, but writing the 26-slot count array by hand once is worth it to remember *why* it works.
- **Complement lookup** — [LC1 Two Sum](https://leetcode.com/problems/two-sum/): instead of checking every pair, store each visited value's index in a map and, for each new element, look up `target - current` directly.

```python
# Two Sum's complement lookup, as the canonical shape
seen = {}
for i, num in enumerate(nums):
    complement = target - num
    if complement in seen:
        return [seen[complement], i]
    seen[num] = i
```

## The generalized insight

All three collapse to: **"remember what you've passed so you never rescan it."** The hash map converts an O(n) linear search into an O(1) average lookup by paying O(n) extra space. That's the real lesson — not the individual problems, but recognizing that any "have I seen this before / does this exist elsewhere in the data" question is a candidate for this space-for-time trade, before reaching for nested loops or sorting.

## Notes to self

- `dict.get(key, default)` keeps Two Sum branch-free.
- Anagram edge case: length check first — cheap early exit.
