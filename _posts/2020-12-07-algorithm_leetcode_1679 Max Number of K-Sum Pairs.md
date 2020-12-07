---
title: "126 Word Ladder II"
date: 2020-12-07
categories: algorithm-leetcode
---

### problem definition

> input
beginWord = "hit",
endWord = "cog",
wordList = ["hot","dot","dog","lot","log","cog"]

> output
[
  ["hit","hot","dot","dog","cog"],
  ["hit","hot","lot","log","cog"]
]

### my solution

for every single input, it checks the previous inputs

 M + (M - 1) + (M - 2) ... = M(M + 1)/2
  = O(M^2)


```python
def findLadders(self, beginWord: str, endWord: str, wordList: List[str]) -> List[List[str]]:
    g = collections.defaultdict(list)
    visited = set()

    for word in wordList:
        for i in range(len(word)):
            key = word[:i] + '@' + word[i + 1:]
            g[key] += word,

    q = [(beginWord, [beginWord])]
    res = set()

    while q:
        word, trace = q.pop(0)
        visited.add(word)
        if word == endWord:
            res.add(tuple(trace))

        for i in range(len(word)):
            key = word[:i] + '@' + word[i + 1:]
            for v in g[key]:
                if v in visited:
                    continue

                q += (v, trace + [v]),

    if not res:
        return None

    res = sorted(res, key=len)
    ret = [list(res.pop(0))]
    for i in range(len(res)):
        if len(res[i]) != len(ret[-1]):
            break
        ret += list(res[i]),

    return ret
```

&nbsp;


## TC
```python
print([
  ["hit","hot","dot","dog","cog"],
  ["hit","hot","lot","log","cog"]
] == Solution().findLadders("hit", "cog", ["hot","dot","dog","lot","log","cog"]))
```
