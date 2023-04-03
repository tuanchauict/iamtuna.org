---
layout: post
title:  "Leetcode 148: Sort list - with Radix sort"
author: "Tuna"
comments: false
category: Leetcode
tags: leetcode python radix-sort
excerpt_separator: <!--more-->
sticky: false
hidden: false
draft: false
---

The question is simple: 

> Given the `head` of a linked list, return the list after sorting it in **ascending order**<!--more-->.
> <br> *([Leetcode 148](https://leetcode.com/problems/sort-list/))*


### Simple solutions
A simple solution is:

1. Create an array (or list) from the linked list.
2. Apply the sort algorithm to the array (or list).
3. Recreate the linked list or update linked list's node value

The time and memory complexity are O(n logn) and O(n) respectively.

But this solution is boring because we actually don't do anything.

More interested in solution is to use merge sort. A simple solution of merge sort is to use top-down recursion.

1. Cut the linked list in half (by couting or by fast-slow pointers)
2. For each left and right sublist
   1. Repeat step 1 with the sublist has more than 1 item
3. Merge left and right sublists in ascending order
4. Return the merged list

This approach has O(n logn) for time complexity and O(logn) for space complexity for the backtracking.

The follow up of this question is more interesting.

{: .box-note}
> Can you sort the linked list in `O(n logn)` time and `O(1)` memory (i.e. constant space)?

We can achieve `O(1)` memory with bottom up merge sort approach, you can see this on the Leetcode's discusson or solution. However, the code is quite complicated.

My solution for the follow up question is simpler: use [Radix sort](https://en.wikipedia.org/wiki/Radix_sort).

### Radix sort

Different from traditional radix sort on array, we don't need an extra memory for reordering the linked list. All we need is a similar array for counting.

```
┌───┐┌───┐┌───┐┌───┐┌───┐┌───┐┌───┐┌───┐┌───┐┌───┐
│ 0 ││ 1 ││ 2 ││ 3 ││ 4 ││ 5 ││ 6 ││ 7 ││ 8 ││ 9 │
└─▲─┘└───┘└─▲─┘└───┘└───┘└─▲─┘└───┘└───┘└───┘└─▲─┘
  │         │              │                   │  
┌─┴─┐     ┌─┴─┐          ┌─┴─┐               ┌─┴─┐
│10 │     │52 │          │35 │               │ 9 │
└─▲─┘     └─▲─┘          └─▲─┘               └───┘
  │         │              │                      
┌─┴─┐     ┌─┴─┐          ┌─┴─┐                    
│20 │     │ 2 │          │ 5 │                    
└─▲─┘     └───┘          └─▲─┘                    
  │                        │                      
┌─┴─┐                    ┌─┴─┐                    
│40 │                    │25 │                    
└───┘                    └───┘         
               Memory image of nodes           
```

For simple, here I use `10^exp` base for the radix sort. We just need to run 5 times to cover all possible values from the test cases

Steps:

- For each exp from 0 to 5
    - Create an array `tails` with size 20 (0-9 for negative, 10-19 for positive)
    - Loop over all nodes on the original list
      - Calculate index of the node based on index
      - Append new node to the linked list (`tails[index].next = node`)
    - Merge all sub-list on the tail array

One note for merge step, because we store the sub-list in reverse order, we need to reverve the list when merge. Merge runs in `O(n)` but we can improve to `O(1)` by keeping both `head` and `tail` of the sub-list.

Code:

{% highlight python linenos%}
def radix_sort(head):
    def reverse(node):
        cur, nex, tail = None, node, node
        while nex:
            tmp = nex.next
            nex.next = cur
            cur, nex = nex, tmp
        return cur, tail

    def radix(head, exp):
        # From exp = 1, the tail looks like: 4 -> 3 -> 2 -> 1, 
        # therefore, we need to reverse when merging
        tails = [ListNode(0) for _ in range(20)]
        div = 10 ** exp
        cur = head
        while cur:
            abs_index = (abs(cur.val) // div) % 10
            index = 10 + abs_index * (1 if cur.val >= 0 else -1)
            tmp = cur.next
            cur.next = tails[index].next
            tails[index].next = cur
            cur = tmp

        dummy = ListNode(0)
        cur = dummy
        # Merge. If we store both head and tail, the merge is 
        # faster with O(1)
        for tail in tails:
            if tail.next:
                h, t = reverse(tail.next)
                cur.next = h
                cur = t
        return dummy.next

    # Run
    for i in range(0, 5):
        head = radix(head, i)
    return head
{% endhighlight %}

#### Improvement
In addition to merge step improvement, there is 1 more place we can improve the run time.
At the `#Run` step, we can also do ealry break if we check there is only 1 tail in the `tails` array has valid value.

```python
        # Merge
        count = 0
        for tail in tails:
            if tail.next:
                h, t = reverse(tail.next)
                cur.next = h
                cur = t
				count += 1
        return dummy.next, count == 1
```

then

```python
    # Run
    for i in range(0, 5):
        head, is_merged = radix(head, i)
		if is_merged: break
```

#### Complexity
We need `O(n)` for breaking the list into sublist and also `O(n)` for merging, therefore `radix`'s time complexity is `O(n)`. 

`Run` is fixed to 5 for this problem (*`-10^5 <= Node.val <= 10^5`*) but it's actually `O(log n)` in general cases. Therefore, overall, the time complexity is `O(n log n)`.
We just need an extra array with fixed size, therefore, space complexity is `O(1)`.
