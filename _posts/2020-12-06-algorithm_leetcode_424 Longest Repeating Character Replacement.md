---
title: "424 Longest Repeating Character Replacement"
date: 2020-12-06
categories: algorithm-leetcode
---

### my solution
```python
def characterReplacement(self, s: str, k: int) -> int:
    if not s:
        return 0

    r = l = 0
    freq = collections.Counter()

    for r, ch in enumerate(s):
        freq[ch] += 1
        mx = freq.most_common(1)[0][1]

        if r - l + 1 - mx > k:
            freq[s[l]] -= 1
            l += 1

    return r - l + 1
```

&nbsp;

### optimized one

```python
def characterReplacement(self, s, k):
    if not s:
        return 0

    freq = collections.defaultdict(int)
    mx = 0
    l = 0
    
    for r, ch in enumerate(s):
        freq[ch] += 1
        mx = max(mx, freq[ch])
        
        while r - l + 1 - mx > k:
            freq[s[l]] -= 1
            l += 1
        
    return r - l + 1
```

&nbsp;

## TC
```python
print(4 == Solution().characterReplacement(s = "ABAB", k = 2))
print(4 == Solution().characterReplacement(s = "AABABBA", k = 1))
print(4 == Solution().characterReplacement("AAAA", 0))
print(3 == Solution().characterReplacement("AAAB", 0))
```
