---
title: "1679  Max Number of K-sum Pairs"
date: 2020-12-07
categories: algorithm-leetcode
---

### problem definition

> input
nums = [1,2,3,4], k = 5

> output
2

* pick two values that sum up k
* repeat until it is possible
* return the number of pick-up

&nbsp;

### my solution 1


```python
def maxOperations(self, nums: List[int], k: int) -> int:
    cnt = 0
    idxs = collections.defaultdict(int)

    for i, num in enumerate(nums):
        idxs[num] += 1

    for num in idxs:
        left = k - num

        if left == num:
            cnt += idxs[num]//2
            idxs[num] = 0
        elif left in idxs:
            cnt += min(idxs[left], idxs[num])
            idxs[left] = 0
            idxs[num] = 0

    return cnt
```

&nbsp;


### my solution 2

```python
def maxOperations(self, nums: List[int], k: int) -> int:
    cnt = 0
    if not nums or len(nums) == 1:
        return cnt

    nums.sort()
    l = 0
    r = len(nums) - 1

    while l < r:
        s = nums[l] + nums[r]
        if s == k:
            cnt += 1
            l +=1
            r -=1
        elif s > k:
            r -= 1
        else:
            l += 1

    return cnt
```

&nbsp;
