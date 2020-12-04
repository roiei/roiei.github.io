---
title: "243. Shortest Word Distance"
date: 2020-12-04 10:41:00 -0400
categories: algorithm-leetcode
---

### my solution
```python
class Solution:
    """
    @param words: a list of words
    @param word1: a string
    @param word2: a string
    @return: the shortest distance between word1 and word2 in the list
    """
    def shortestDistance(self, words, word1, word2):
        tgt_words = {word1, word2}
        positions = []
        mn = float('inf')

        for i, word in enumerate(words):
            if word in tgt_words:
                positions += (i, word),

        for i in range(1, len(positions)):
            if positions[i][1] != positions[i - 1][1]:
                mn = min(mn, positions[i][0] - positions[i - 1][0])

        return mn
```

&nbsp;
### TC
```python
print(3 == Solution().shortestDistance(["practice", "makes", "perfect", "coding", "makes"],"coding","practice"))
print(1 == Solution().shortestDistance(["practice", "makes", "perfect", "coding", "makes"],"makes","coding"))
```
