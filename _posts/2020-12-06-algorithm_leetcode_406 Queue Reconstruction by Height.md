---
title: "406 Queue Reconstruction by Height"
date: 2020-12-06
categories: algorithm-leetcode
---

### problem definition
people[i] = 11(hi, ki)
    where
        hi is height of ith person
        ki is number of people in front ith person


### 1st solution

for every single input, it checks the previous inputs

 M + (M - 1) + (M - 2) ... = M(M + 1)/2
  = O(M^2)


```python
def reconstructQueue(self, people: [[int]]) -> [[int]]:
    people.sort(key=lambda p: p[1], reverse=False)
    seq = []

    for hi, ki in people:
        i = 0
        temp_ki = ki

        while seq and i < len(seq):
            if seq[i][0] < hi:
                i += 1
            elif temp_ki:
                temp_ki -= 1
                i += 1
            else:
                break

        seq.insert(i, [hi, ki])

    return seq
```

&nbsp;

### optimized one

> fill the value from the most highest height and the lowest number of people in front

```python
def reconstructQueue(self, people: List[List[int]]) -> List[List[int]]:
    q = []
    for hi, ki in people:
        heapq.heappush(q, (-hi, ki))

    seq = []
    while q:
        hi, ki = heapq.heappop(q)
        seq.insert(ki, (-hi, ki))

    return seq
```

&nbsp;

## TC
```python
print([[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]] == Solution().reconstructQueue([[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]))
```
