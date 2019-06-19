---
layout: post
title:  Longest Increasing Subsequence
category: Leetcode
tags: [LEETCODE,Algorithms, list, Medium]
imagefeature:
comments: true
share: true
---
<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>


# **430. Flatten a Multilevel Doubly Linked List**
You are given a doubly linked list which in addition to the next and previous pointers, it could have a child pointer, which may or may not point to a separate doubly linked list. These child lists may have one or more children of their own, and so on, to produce a multilevel data structure, as shown in the example below.

Flatten the list so that all the nodes appear in a single-level, doubly linked list. You are given the head of the first level of the list.



**Example**
Input:
 1---2---3---4---5---6--NULL
         |
         7---8---9---10--NULL
             |
             11--12--NULL

Output:
1-2-3-7-8-11-12-9-10-4-5-6-NULL


A typical recursive function can do the work.
~~~ python

Definition for a Node.
"""
class Node:
    def __init__(self, val, prev, next, child):
        self.val = val
        self.prev = prev
        self.next = next
        self.child = child
"""
class Solution:

    def addnode(self,  current):

        while True:

            if current.next is None and current.child is None:
                return current

            if current.child != None:
                current.child.prev = current
                comesback =  current.next
                current.next = current.child
                end = self.addnode(current.child)
                current.child = None
                end.next = comesback
                if comesback is not None:
                    comesback.prev = end

            current = current.next


    def flatten(self, head: 'Node') -> 'Node':
        if head is None:
            return head
        self.newhead  = head
        self.currentnode = head
        self.addnode(self.currentnode)


        return self.newhead

~~~
