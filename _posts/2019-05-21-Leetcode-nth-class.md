---
layout: post
title:  nth problems
category: sample-post
tags: [LEETCODE,Algorithms]
imagefeature:
comments: true
share: true
---
<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>


# **19. Remove Nth Node From End of List**
## problem discription:
### Given a linked list, remove the n-th node from the end of list and return its head.

## Solution:
### Firstly, have two pointers $$p_1$$, $$p_2$$. let p1,p2 = head, head. Let p2 goes nth before p1, then, move p1,p2 simutanlously. When p2 reaches the end. p1 points to the nth node from the end of the list.


~~~ python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def removeNthFromEnd(self, head: ListNode, n: int) -> ListNode:
        p1,p2 = head,head
        for i in range(0, n):
            if p2 == None:
                break
            p2 = p2.next

        if p2 == None:
            head = head.next
            return head
        while True:

            p2 = p2.next
            if p2 == None:
                break
            p1 = p1.next


        p1.next = p1.next.next

        return head
~~~


# **400. Nth Digit**
## problem discription:
### Find the nth digit of the infinite integer sequence 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, ...

## Solution:
### If n is smaller than 10, just return. then, we seperate the digits to 2 bits(10), 3 bits(1000) blocks. We find which blocks the result nuber in and then calcualte the offset in that block.

~~~python

class Solution:
    def findNthDigit(self,n):
        if n < 10:
            return n

        accum = 9

        for i in range(1, n):
            all = (10 ** (i+1)-(10**i)) * (i+1)
            if accum + all > n:
                break
            accum += all
        gap = n- accum
        offset = gap%(i+1)
        which = (gap - offset)//(i+1)

        if offset == 0:
            number = 10 ** (i) + which-1
            val = str(number)[-1]
        else:
            number = 10**(i) + which-1
            found = number+1
            val = str(found)[offset-1]

        return int(val)
~~~
# **878. Nth Magical Number**
## problem discription:
### A positive integer is magical if it is divisible by either A or B. Return the N-th magical number.  Since the answer may be very large, return it modulo $$10^9 + 7$$.

### Solution:

### It A is a factor of B. return N*A elsewise, find the LCM of A,B.  and the the LCM also indicate the number of elements in the period list(in program it is base list). For instance, with 5 and 8
### base = [5,8,10,15,16,20,25,28,30,35,36,40] and the element in base is LCM/A +LCM/B-1

### we calculate the repetions of the list for the given N and the offset in that repetion and return.
~~~python
class Solution:

    def dowork(self,N,A,B):
        if B % A == 0:
            return N * A

        lcm = A*B//math.gcd(A,B)
        elementlen = lcm//A + lcm//B-1

        i=1
        j=1
        counter = 0
        base = []
        while True:
            if counter == elementlen:
                break
            av = i*A
            bv = j*B
            if av < bv:
                base.append(av)
                i+=1
                counter +=1
            else:
                base.append(bv)
                j+=1
                counter +=1


        block = N//elementlen
        offset = N%elementlen

        if offset == 0:
            return lcm*block

        else:
            val = lcm*block
            val += base[offset-1]

        return val



    def nthMagicalNumber(self, N: int, A: int, B: int) -> int:
        modval = 1000000000+7
        if A > B:
            A,B = B,A

        ret = self.dowork(N,A,B)
        return ret%modval
~~~
