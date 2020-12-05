---
title: "783 Minimum Distance Between BST Nodes"
date: 2020-12-05
categories: algorithm-leetcode
---

### my solution
```python
class Solution:
    def minDiffInBST(self, root: TreeNode) -> int:
        def dfs(node):
            if not node.left and not node.right:
                return [node.val]

            l, r = [], []

            if node.left:
                l += dfs(node.left)

            if node.right:
                r += dfs(node.right)

            return l + [node.val] + r

        nums = dfs(root)
        mn = float('inf')
        for i in range(1, len(nums)):
            mn = min(mn, nums[i] - nums[i - 1])

        return mn
```


&nbsp;
### TC
```python
print(6 == Solution().minDiffInBST(deserialize('[27,null,34,null,58,50,null,44,null,null,null]')))
```
