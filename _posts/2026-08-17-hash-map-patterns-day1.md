---
title: "[TIL] Hash Map Patterns — Contains Duplicate, Valid Anagram, Two Sum"
date: 2026-08-17 09:00:00 +0900
categories: [TIL, Algorithms]
tags: [leetcode, hash-map, arrays]
---

Restarting my daily study log. Day 1: three warm-up problems from the Arrays & Hashing pattern.

## Problems

- [LC217 Contains Duplicate](https://leetcode.com/problems/contains-duplicate/) — a set gives O(n) membership checks; sorting would be O(n log n) for no benefit here.
- [LC242 Valid Anagram](https://leetcode.com/problems/valid-anagram/) — two frequency counts must match. `Counter(s) == Counter(t)` is the one-liner, but writing the count array by hand (26 slots) is worth doing once to remember *why* it works.
- [LC1 Two Sum](https://leetcode.com/problems/two-sum/) — the classic "store what you've seen, look up what you need". One pass: for each `x`, check if `target - x` was already seen.

## The pattern

All three are the same idea wearing different clothes: **trade memory for lookup speed**. When a brute-force solution re-scans the array for every element (O(n²)), ask what a hash map could remember instead — the answer is usually "everything I've already walked past".

## Notes to self

- `dict.get(key, default)` keeps Two Sum branch-free.
- Anagram edge case: length check first — cheap early exit.
