---
title: "538 Convert BST to Greater Tree"
date: 2020-12-05
categories: algorithm-leetcode
---

### my solution
```python
class Solution:
    def bstToGst(self, root: TreeNode) -> TreeNode:
        nodes = collections.defaultdict(None)
        tot = 0
        
        def dfs(nodes, node):
            nonlocal tot
            
            if not node:
                return
            
            nodes[node.val] = node
            tot += node.val
            
            dfs(nodes, node.left)
            dfs(nodes, node.right)
        
        dfs(nodes, root)
        
        nodes = sorted(nodes.items(), key=lambda p: p[0])

        preval = 0
        for val, node in nodes:
            tot -= preval
            node.val, preval = tot, node.val
        
        return root
    
    def bstToGst(self, root: TreeNode) -> TreeNode:
        def dfs(node, nodes):
            if not node:
                return

            dfs(node.right, nodes)
            nodes += node,
            dfs(node.left, nodes)

        nodes = []
        dfs(root, nodes)

        inc = 0
        for node in nodes:
            inc += node.val
            node.val = inc

        return root
```

* considerables
  * multiple path in a vertex
  * more faster way with the more edges


&nbsp;
### TC
```python
print(traverse_level(deserialize('[30,36,21,36,35,26,15,null,null,null,33,null,null,null,8]')) == traverse_level(Solution().bstToGst(deserialize('[4,1,6,0,2,5,7,null,null,null,3,null,null,null,8]'))))
```
