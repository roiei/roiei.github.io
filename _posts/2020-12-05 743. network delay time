---
title: "743 Network Delay Time"
date: 2020-12-05
categories: algorithm-leetcode
---

### my solution
```python
    def networkDelayTime(self, times: List[List[int]], N: int, K: int) -> int:
        g = collections.defaultdict(dict)
        for u, v, w in times:
            g[u][v] = w
        
        q = [(0, K)]
        visited = set()
        mn = 0
        
        while q:
            inc, u = heapq.heappop(q)
            if u not in visited:
                mn = inc
            
            visited.add(u)
            
            for v, w in g[u].items():
                if v in visited:
                    continue
                
                heapq.heappush(q, (inc + w, v))
        
        return mn if len(visited) == N else -1
```

* considerables
  * multiple path in a vertex
  * more faster way with the more edges


&nbsp;
### TC
```python
print(2 == Solution().networkDelayTime([[1,2,1],[2,3,2],[1,3,2]], 3, 1))
print(-1 == Solution().networkDelayTime([[1,2,1],[2,3,2],[1,3,1]], 3, 2))
```
