---
title: "621 Task Scheduler"
date: 2020-12-06
categories: algorithm-leetcode
---

### problem definition

for each time, 
    the CPU need to do task or being idle
task could be done in any order.

동일 task를 수행한 이후에는 적어도 n의 시간을 쉬어야 함

1. 
tasks = ["A","A","A","B","B","B"], n = 2

A -> B -> idle -> A -> B -> idle -> A -> B
 = 8

A:3, B:3

최대 빈출 task의 수는 3
즉, 3 - 1의 interval 존재 

2 x 2 = 4의 idle padding 필요

B:3 이고 이는 interval 수 2 보다 1 크니, 
B 2개만 idle로 사용 가능

즉, 4 - 2 = 2개의 idle 추가 필요


2. 
["A","A","A","A","A","A","B","C","D","E","F","G"], n = 2

A -> B -> C -> A -> D -> E -> A -> F -> G -> A -> idle -> idle -> A -> idle -> idle -> A

즉, 가장 빈번한 task의 수 - 1이 필요 interval 수
각 interval은 n의 pad를 필요
위 경우
A:6 이니, 

(6 - 1)x2 = 10의 추가 idle 필요

그러나 B~G가 각각 1씩 존재하기에, 
10 - 6 = 4의 pad 필요


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
