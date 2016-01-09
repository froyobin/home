---
layout: post
title:  Minimum Depth of Binary Tree
category: sample-post
tags: [LEETCODE,Algorithms]
imagefeature: 
comments: true
share: true
---
**Problem:**


Given a binary tree, find its minimum depth.

The minimum depth is the number of nodes along the shortest path from the root node down to the nearest leaf node.





#### **Solution**

~~~ python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    
    def minf(self,a,b):
        if a<b:
            return a
        else:
            return b
    
    def minDepth(self, root):
        """
        :type root: TreeNode
        :rtype: int
        """
        if root ==None:
            return 0
        lmin = self.minDepth(root.left)
        rmin = self.minDepth(root.right)
        
        if lmin==0 and rmin==0:
            return 1
        if lmin==0:
            lmin=INT_MAX
        if rmin==0:
            rmin=INT_MAX
        return self.minf(lmin,rmin)+1
~~~
