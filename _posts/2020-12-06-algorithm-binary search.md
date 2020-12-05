---
title: "binary search"
date: 2020-12-06
categories: algorithm
---

## binary search type 1

searching for an element, single index
 - comparision of elements is determined by only condition

```python
l = 0
r = len(nums) - 1

while l <= r:
    m = (l + r)//2
    if nums[m] == target:
        return m
    elif nums[m] < target:
        l = m + 1
    else:
        r = m - 1

return l # or -1
```
&nbsp;

## binary search type 2

requried to access the current index and its neighbor's index
at the end of the code, there could be one element not handled.

```python
l = 0
r = len(nums)

while l < r:
    m = (l + r)//2
    if nums[m] == target:
        return m
    elif nums[m] > target:
        r = m
    else:
        l = m + 1

if l != len(nums) and nums[l] == target:
    return l

return -1
```
&nbsp;

## binary search type 3

accessing the current index and its left and right indices

```python
l = 0
r = len(nums) - 1

while l + 1 < r:
    m = (l + r)//2
    if nums[m] == target:
        return m
    elif nums[m] > target:
        r = m
    else:
        l = m

if nums[l] == target:
    return l

if nums[r] == target:
    return r

return -1
```
&nbsp;

### application for the type 3
* 34. Find First and Last Position of Element in Sorted Array
```python
    def searchRange(self, nums, target):
        def _search(nums, target, left):
            l = 0
            r = len(nums)

            while l < r:
                m = (l + r)//2
                if nums[m] > target or (left and nums[m] == target):
                    r = m
                else:
                    l = m + 1

            return l

        l_idx = _search(nums, target, True)
        if l_idx == len(nums) or nums[l_idx] != target:
            return [-1, -1]

        return [l_idx, _search(nums, target, False) - 1]
```

but, it's quite tricky. 
the following is more understandable
&nbsp;

```python
    def search(self, nums, val):
        l, r = 0, len(nums) - 1

        while l <= r:
            m = (l + r)//2
            if nums[m] == val:
                s = e = m
                j = m - 1
                while j >= 0:
                    if nums[j] == val:
                        s = j
                    j -= 1
                j = m + 1
                while j <= len(nums) - 1:
                    if nums[j] == val:
                        e = j
                    j += 1
                return [s, e]
            if nums[m] > val:
                r = m - 1
            else:
                l = m + 1

        return [-1, -1]

    def searchRange(self, nums, target):
        if not nums:
            return [-1, -1]
        return self.search(nums, target)
```
