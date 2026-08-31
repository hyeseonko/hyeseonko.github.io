---
title: "Valid Palindrome: Compare the Filtered String, Not the Original"
date: 2026-09-01 07:54:56 +0900
categories: [Algorithms]
tags: [leetcode, two-pointers, strings, isalnum, complexity]
---

[LC125 Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) asks whether a string reads the same forwards and backwards *after* lowercasing and dropping every non-alphanumeric character. The problem looks trivial, but it hides one bug that is easy to write and hard to see.

## The trap

The natural first move is to build a cleaned list of characters, then walk from both ends:

```python
filtered = [c.lower() for c in s if c.isalnum()]
for i in range(len(filtered) // 2):
    if s[i] != s[-i - 1]:      # BUG: comparing the original string
        return False
return True
```

You filtered into `filtered`, then compared `s` — the raw input, spaces and punctuation still in it. The indices no longer line up, so most non-trivial inputs give the wrong answer. Compare the thing you cleaned:

```python
filtered = [c.lower() for c in s if c.isalnum()]
for i in range(len(filtered) // 2):
    if filtered[i] != filtered[-i - 1]:
        return False
return True
```

## Two details that matter

- **`isalnum()`, not `isalpha()`.** Digits are part of the check. `"0P"` is *not* a palindrome (`0` vs `P`); dropping digits would silently pass it.
- **`lower()` normalizes case.** It returns a new value and leaves the original untouched — fine inside a comprehension, where you use the return value directly.

## O(1) space: two pointers

Building `filtered` costs O(n) extra space. You can skip it entirely by scanning inward and stepping over non-alphanumeric characters in place:

```python
i, j = 0, len(s) - 1
while i < j:
    while i < j and not s[i].isalnum(): i += 1
    while i < j and not s[j].isalnum(): j -= 1
    if s[i].lower() != s[j].lower():
        return False
    i += 1; j -= 1
return True
```

| Approach | Time | Space |
|---|---|---|
| Filter + compare both ends | O(n) | O(n) |
| Two pointers | O(n) | **O(1)** |

The lesson generalizes past this one problem: when you transform data into a cleaned copy, every later step has to reference the copy, not the source it came from.
