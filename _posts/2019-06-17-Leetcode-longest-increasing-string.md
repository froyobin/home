---
layout: post
title:  Longest Increasing Subsequence
category: sample-post
tags: [LEETCODE,Algorithms]
imagefeature:
comments: true
share: true
---
<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>


# **300. Longest Increasing Subsequence**
Given an unsorted array of integers, find the length of longest increasing subsequence.

Example:

Input: [10,9,2,5,3,7,101,18]
Output: 4
Explanation: The longest increasing subsequence is [2,3,7,101], therefore the length is 4.

Note:

    There may be more than one LIS combination, it is only necessary for you to return the length.
    Your algorithm should run in O(n2) complexity.

Follow up: Could you improve it to O(n log n) time complexity?


Here is the DP version
~~~ python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

300. Longest Increasing Subsequence
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:

        if len(nums)==0:
            return 0

        Len = [0]*len(nums)


        for j in range(0,len(nums)):
            maxvalue = [0]
            for i in range(0, j):
                if nums[i]< nums[j]:
                    maxvalue.append(Len[i])


            output = max(maxvalue) +1
            Len[j] = output

        return max(Len)

~~~


binary search version. However, here I just implement the linear search version. change range(1,len(nums)) to the binary search to achieve n(logn) time complexity.


~~~ python
        class Solution:
            def lengthOfLIS(self, nums: List[int]) -> int:

                if len(nums) == 0:
                    return 0

                lastpos = [0]
                lastpos.append(nums[0])
                length = 1
                for i in range(1, len(nums)):
                    currentval = nums[i]
                    if currentval <= lastpos[-1]:

                        for i in range(0, len(lastpos)):
                            if currentval <= lastpos[i]:
                                break

                        lastpos[i] = currentval
                    else:
                        lastpos.append(currentval)


                print(lastpos)

                return len(lastpos)-1
~~~
