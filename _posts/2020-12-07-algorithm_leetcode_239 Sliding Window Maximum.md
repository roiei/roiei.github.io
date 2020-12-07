---
title: "239 Sliding Window Maximum"
date: 2020-12-07
categories: algorithm-leetcode
---

### problem definition

> input
nums = [1,3,-1,-3,5,3,6,7], k = 3

> output
[3,3,5,5,6,7]

&nbsp;

### my solution

for every single input, it checks the previous inputs

 M + (M - 1) + (M - 2) ... = M(M + 1)/2
  = O(M^2)


```python
def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:
    if not nums or len(nums) < k:
        return None

    sorted_wnd = sorted(nums[:k])
    res = [max(sorted_wnd)]

    for i in range(k, len(nums)):
        left = nums[i - k]
        idx = bisect.bisect_left(sorted_wnd, left)
        sorted_wnd.pop(idx)

        idx = bisect.bisect_left(sorted_wnd, nums[i])
        sorted_wnd.insert(idx, nums[i])
        res += sorted_wnd[-1],

    return res
```

nlogn + N x (logn x 2) = O(nlogn)

&nbsp;

