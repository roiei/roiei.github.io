---
title: "226 Invert Binary Tree"
date: 2020-12-05
categories: algorithm-leetcode
---

### my solution
```python
    def invertTree(self, root: TreeNode) -> TreeNode:
        def dfs(node):
            if not node:
                return None
            
            node.left, node.right = \
                dfs(node.right), dfs(node.left)

            return node
    
        return dfs(root)
```

* considerables
  * multiple path in a vertex
  * more faster way with the more edges


&nbsp;
### TC
```python
print(tree_is_same(deserialize('[4,7,2,9,6,3,1]'), Solution().invertTree(deserialize('[4,2,7,1,3,6,9]'))))
```
