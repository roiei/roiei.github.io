---
title: "617 Merge Two Binary Trees.md"
date: 2020-12-05
categories: algorithm-leetcode
---

### my solution
```python
class Solution:
    def mergeTrees(self, t1: TreeNode, t2: TreeNode) -> TreeNode:
        
        def dfs(node1, node2):
            if not node1 or not node2:
                return node1 if node1 else node2

            node = TreeNode(node1.val + node2.val)
            node.left = dfs(node1.left, node2.left)
            node.right = dfs(node1.right, node2.right)
            return node
    
        node = dfs(t1, t2)
        return node
```
