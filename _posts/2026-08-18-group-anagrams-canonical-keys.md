---
title: "Group Anagrams: Choosing a Canonical Key"
date: 2026-08-18 09:00:00 +0900
categories: [Algorithms]
tags: [leetcode, hash-map, strings, complexity]
---

[LC49 Group Anagrams](https://leetcode.com/problems/group-anagrams/) asks you to group words that are permutations of each other.

## The real question

Grouping is trivial once you have the right key: build a `dict` from key to list, append, done. The entire problem is **"what makes two anagrams collide and everything else not?"** That's a canonical form question — pick any representation that is identical for all members of a group and unique across groups.

Two candidates:

**Sorted string.** `"eat" → "aet"`. One line, obviously correct.

```python
key = "".join(sorted(word))
```

**Character counts.** Anagrams have identical letter frequencies, so serialize the multiset instead:

```python
key = "".join(c + str(word.count(c)) for c in sorted(set(word)))  # "eat" → "a1e1t1"
```

I went with the count-based key. Both accept, so the interesting part is *why* you'd prefer one.

## Complexity, and where I got it wrong

For `n` words of length `k`:

- Sorting keys: `O(n · k log k)`
- Counting keys: `O(n · k)` if you count in a single pass

The counting version is asymptotically better — but **my implementation threw that away**. `word.count(c)` scans the whole word once *per distinct character*, so with an alphabet of size `a` it's `O(k · a)` per word, worse than sorting for short strings. The fix is one pass with a counter:

```python
from collections import Counter, defaultdict

def groupAnagrams(strs):
    groups = defaultdict(list)
    for word in strs:
        counts = [0] * 26
        for ch in word:
            counts[ord(ch) - ord("a")] += 1
        groups[tuple(counts)].append(word)   # tuple is hashable; list is not
    return list(groups.values())
```

Two details worth remembering: the key must be **hashable**, so `tuple(counts)` rather than the list itself; and a fixed-size 26-slot vector makes the key comparison cost constant instead of proportional to the number of distinct characters.

## Takeaway

Sorting is the better *first* answer in an interview: shorter, no bugs, easy to explain. Counting is the better answer when `k` is large or the alphabet is small and fixed — and only if you actually count in one pass. Knowing which one you picked *and why* matters more than picking the clever one.

Writing the key generator badly cost more than choosing the "wrong" strategy would have.
