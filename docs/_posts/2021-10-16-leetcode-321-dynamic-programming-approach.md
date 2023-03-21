---
layout: post
title:  "LeetCode 321: Create maximum number — a dynamic programming approach"
author: "Tuna"
comments: false
category: Leetcode
tags: leetcode dynamic-programming python monotonic-stack
excerpt_separator: <!--more-->
sticky: false
hidden: false
draft: false
---

{: .box-italic}
>You are given two integer arrays `nums1` and `nums2` of lengths `m` and `n` respectively. `nums1` and `nums2` represent the digits of two numbers. You are also given an integer `k`<!--more-->.
> 
> Create the maximum number of length `k <= m + n` from digits of the two numbers. The relative order of the digits from the same array must be preserved.
> 
> Return an array of the `k` digits representing the answer.<br>
> ([Leetcode 321: Create Maximum Number](https://leetcode.com/problems/create-maximum-number/))

## Approach

Instead of applying the [Monotonic stack](https://labuladong.gitbook.io/algo-en/ii.-data-structure/monotonicstack) as used in the algorithm proposed by [dietpepsi](https://leetcode.com/problems/create-maximum-number/discuss/77285/Share-my-greedy-solution), my solution uses Dynamic Programming to have better runtime. Let’s break the problem into smaller problems.

## Extremely easy version #0

{: .box-note}
> #0.1. Give one array of numbers, find maximum number of the array and its 1st appearance index.

Yes, I’m serious. In python, we can solve this easily as:

```python
def max_and_index(array):
    max_value = max(array)
    index = array.index(max_value)
    return max_value, index
```

Okay, let’s increase the “complexness” a little bit.

{: .box-note}
> #0.2. Give two arrays of numbers, find the maximum number of the two and its first appearance index.

Super easy, right? Let get back to this later.

## Easy version #1
#1. Given one array of length **n**, create the maximum number of length **k** (k ≤ n).

Okay, this version is similar to what [dietpepsi mentioned in his article](https://web.archive.org/web/20160120093629/http://algobox.org/create-maximum-number/). However, this time, we don’t use the monotonic stack but use recursion to find the maximum digit of each position on the final array from left to right based on facts:

- The final value is maximum if and only if the 1st number is the maximum of the possible options
- There are `n-k+1` options for selecting the 1st number

By using these facts, we can construct a recursive algorithm as:

- Find the `max_value` of subarray `array[0:n-k+1]` for the 1st number
- Get the index of the `max_value` in the `array`
- Continue with subarray `array[index+1:]` and `k-1`
- The algorithm stops when the length of the array is equal to `k`. This time, we use the leftover of the array for the final value as there is only 1 option for each number.

Python code:

```python
def max_array(nums, k):
    if len(nums) == k:
        return nums
    max_value = max(nums[:len(nums)-k+1])
    index = nums.index(max_value)
    return [candidate] + max_array(nums[index+1:], k - 1)
```

In the worst case, this algorithm runs with `O(n*k)` (not counting list operations), which must be worse than with the mono stack approach.

## Easy version #2

{: .box-note}
> #2. Given two arrays of length **m** and **n**, create maximum number of length **k = m+n**

This time, we can apply [dietpepsi’s solution (Easy version №2)](https://web.archive.org/web/20160120093629/http://algobox.org/create-maximum-number/), I only implement his idea with Python code:

```python
def is_greater(nums1, idx1, nums2, idx2):
    while idx1 < len(nums1) and idx2 < len(nums2) and nums1[idx1] == nums2[idx2]:
        idx1 += 1
        idx2 += 1
    return idx2 == len(nums2) or idx1 < len(nums1) and nums1[idx1] > nums2[idx2]

def merge(nums1, nums2):
    l1, l2 = len(nums1), len(nums2)
    i1, i2 = 0, 0
    ans = []
    while len(ans) < l1 + l2:
        if is_greater(nums1, i1, nums2, i2):
            ans.append(nums1[i1])
            i1 += 1
        else:
            ans.append(nums2[i2])
            i2 += 1
    return ans
```

## Final solution
Okay, come back to the real problem. Recall problem #0.2, the real problem can be described as:

- Get the maximum digit of _**selectable options**_ from both arrays
- Use the max index to skip the number of options left in the array containing the max value.
- Continue until we fill all k-digit array
- When the number of options left is equal to the number of unfilled places on the final array, use `merge()` mentioned in the above code to construct the solution.

### What is “selectable options”?
Before having the final solution, we must answer the question: What is the correct “selectable options”?

As shown in problem **#1**, for a single array, it’s `n-k+1`. For 2 arrays, we can choose up to `n1+n2-k+1` numbers from each array.

Why? For a short answer, because for each round, only one array is chosen, therefore, if the max lays at `(n1+n2-k)`-th of the array, we can fill the rest with the leftover of that array plus all of the other array.

### Simple pseudocode
```
max_number(nums1, nums2, k):
  option_count = len(nums1) + len(nums2) - k + 1
  
  if option_count == 1:
    -> merge(nums1, nums2, k)
  candidate1, index1 = max_and_index(nums1[:option_count])
  candidate2, index2 = max_and_index(nums2[:option_count])

  if candidate1 > candidate2:
    -> [candidate1] + max_number(nums1[index1+1:], nums2, k-1)
  else if candidate2 > candidate1:
    -> [candidate2] + max_number(nums1, nums2[index2+1:], k-1)
  else:
    -> Let's talk about this
```

When the two candidates are equals, it’s harder because we need to check both options to find out which branch will produce the correct answer and it’s double every time we face the equal numbers.

```
else:
  try1 = max_number(nums1[index1+1:], nums2, k-1)
  try2 = max_number(nums1, nums2[index2+1:], k-1)
  if try1 > try2:
    -> [candidate1] + try1
  else:
    -> [candidate2] + try2
```

### Improvement
Denote a, b, c, d are the maximum numbers of each round on both arrays:

```
#1: --a-------b----c----d--->
#2: -a-----b---c-----d------>
```

When we run both checks to select a, we are likely to meet the checks for b, c, and d on smaller runs. This is the chance to apply dynamic programming to improve the runtime. On each run, we store the maximum value found for the input for later use.

The final solution with DP is not so different from the above pseudocode

```
dp = {}

max_number(nums1, nums2, k):
  if (nums1, nums2) in dp:
    return dp[(nums1, nums2)]

  option_count = len(nums1) + len(nums2) - k + 1

  if option_count == 1:
    -> merge(nums1, nums2, k)

  candidate1, index1 = max_and_index(nums1[:option_count])
  candidate2, index2 = max_and_index(nums2[:option_count])

  if candidate1 > candidate2:
    array = [candidate1] + max_number(nums1[index1+1:], nums2, k-1)
  else if candidate2 > candidate1:
    array = [candidate2] + max_number(nums1, nums2[index2+1:], k-1)
  else:
    try1 = [candidate1] + max_number(nums1[index1+1:], nums2, k-1)
    try2 = [candidate2] + max_number(nums1, nums2[index2+1:], k-1)
    array = try1 if try1 > try2 else try2

  dp[(nums1, nums2)] = ret
  return ret
```

### Why this is fast?
Compared to the Monotonic stack approach, as pointed out in Easy version #1, recursion runs slower, but why is it faster here?

To be fair, the two algorithms run almost the same with `k` ~ total numbers as `merge()` is called most of the time. However, when `k` is much less than the number count, this approach jumps faster and `merge()` runs with smaller `k`.

### Final code
In the solution code submitted to Leetcode, I used indexes instead of arrays for each run, therefore, it’s a little bit messive than the pseudocode. Please check out [this gist for the code](https://gist.github.com/tuanchauict/210ed7a2315999bb2d650e6587f6498a).