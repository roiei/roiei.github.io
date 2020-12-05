---
title: "33 Search in Rotated Sorted Array"
date: 2020-12-05
categories: algorithm-leetcode
---

### my solution
```python
class Solution:
    def search(self, nums: [int], target: int) -> int:
        if not nums:
            return -1
        
        def find_pivot(nums, l, r):
            if l > r:
                return -1
            
            m = (l + r)//2
            if m > 0 and nums[m - 1] > nums[m]:
                return m - 1
            if m + 1 <= r and nums[m] > nums[m + 1]:
                return m
            
            idx = find_pivot(nums, l, m - 1)
            if -1 != idx:
                return idx
            
            return find_pivot(nums, m + 1, r)
            
        
        def _search(nums, l, r, target):
            while l <= r:
                m = (l + r)//2
                if nums[m] == target:
                    return m
                if nums[m] < target:
                    l = m + 1
                else:
                    r = m - 1
            return -1
    
        idx = find_pivot(nums, 0, len(nums) - 1)
        if -1 == idx:
            return _search(nums, 0, len(nums) - 1, target)
        else:
            idx1, idx2 = _search(nums, 0, idx, target), \
                _search(nums, idx + 1, len(nums) - 1, target)
            return idx1 if idx1 != -1 else idx2
```

