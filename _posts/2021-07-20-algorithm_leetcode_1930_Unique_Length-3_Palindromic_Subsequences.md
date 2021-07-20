---
title: "243 Shortest Word Distance"
date: 2020-12-04 10:41:00 -0400
categories: algorithm-leetcode
---


주어진 string 내에 길이가 3인 palindrome이 몇 개나 존재하는지를 찾는 문제이다. 
길이가 3으로 고정 되어 있다.  

즉, 양끝의 2개는 같아야 하고, 중간에 뭐가되었던 한개만 존재해도 이는 palindrome이다.
"aabca"가 주어질 시, 
양끝의 a 사이에는 abc가 존재한다.
이 중 하나를 고르면 palindrome이 되는데 a, b, c 3개의 서로 다른 palindrome을 만들 수 있다.

이 idea로 구현한 코드가 다음의 코드이다.


### my solution
```python
class Solution:
    def countPalindromicSubsequence(self, s: str) -> int:
        res = 0
        for c in set(s):
            i, j = s.find(c), s.rfind(c)
            if i > -1:
                res += len(set(s[i + 1: j]))
        return res
```

&nbsp;
### TC
```python
print(3 == Solution().countPalindromicSubsequence("aabca"))
```
