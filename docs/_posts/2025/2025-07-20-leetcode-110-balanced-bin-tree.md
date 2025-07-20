---
layout: post
title:  "Leetcode 110: Balanced Binary Tree - Iterative Solution"
author: "Tuna"
comments: false
category: Leetcode
tags: leetcode python binary-tree iterative stack
excerpt: |
  Exploring an iterative stack-based approach to solve Leetcode problem #110: Balanced Binary Tree. This solution avoids recursion by simulating post-order traversal with a dynamic stack structure...
excerpt_separator: <!--more-->
sticky: false
hidden: false
draft: false
---

The question is straightforward:

> Given a binary tree, determine if it is **height-balanced**.
> 
> For this problem, a height-balanced binary tree is defined as:
> *a binary tree in which the left and right subtrees of every node differ in height by no more than 1.*
> <br> *([Leetcode 110](https://leetcode.com/problems/balanced-binary-tree/))*

### **Simple Solutions**

The most obvious approach is **recursive**:

1. **Calculate height** of left and right subtrees recursively
2. **Check difference** - if `abs(left_height - right_height) > 1`, return false  
3. **Recursively verify** both subtrees are also balanced

```python
def isBalanced(self, root):
    def height(node):
        if not node: return 0
        return max(height(node.left), height(node.right)) + 1
    
    if not root: return True
    return (abs(height(root.left) - height(root.right)) <= 1 and 
            self.isBalanced(root.left) and 
            self.isBalanced(root.right))
```

**Problem**: This has **O(n²)** time complexity because we recalculate heights multiple times.

A **better recursive approach** combines height calculation with balance checking:

```python
def isBalanced(self, root):
    def check(node):
        if not node: return True, 0
        
        left_balanced, left_height = check(node.left)
        if not left_balanced: return False, 0
        
        right_balanced, right_height = check(node.right) 
        if not right_balanced: return False, 0
        
        balanced = abs(left_height - right_height) <= 1
        height = max(left_height, right_height) + 1
        return balanced, height
    
    return check(root)[0]
```

This achieves **O(n)** time and **O(h)** space complexity, where h is the tree height.

But there's a more interesting challenge:

{: .box-note}
> **_Can we solve this iteratively without recursion?_**

### **Iterative Stack Solution**

My solution uses a **dynamic stack approach** that simulates post-order traversal:

```python
def isBalanced(self, root: Optional[TreeNode]) -> bool:
    stack = [[root]]
    while stack:
        item = stack[-1]
        if not item[0] or len(item) == 3:
            left, right = item[1:] if len(item) == 3 else (0, 0)
            if abs(left-right) > 1:
                return False
            stack.pop()
            if stack:
                stack[-1].append(max(left, right)+1)
        elif len(item) == 2:
            stack.append([item[0].right])
        else:
            stack.append([item[0].left])
            
    return True
```

### **How It Works**

The key insight is using a **dynamic stack structure** where each item evolves:

1. **`[node]`** → Initially contains just the node
2. **`[node, left_height]`** → After processing left subtree  
3. **`[node, left_height, right_height]`** → After processing right subtree

```
Stack Evolution Example:
┌─────────────────┐
│ [root]          │ ← Start with root
└─────────────────┘

┌─────────────────┐
│ [left_child]    │ ← Process left subtree first
│ [root]          │
└─────────────────┘

┌─────────────────┐  
│ [right_child]   │ ← Now process right subtree 
│ [root, 2]       │ ← Left subtree height = 2 
└─────────────────┘

┌─────────────────┐
│ [root, 2, 3]    │ ← Both heights available
└─────────────────┘ ← Check balance: |2-3| = 1 ≤ 1 ✓
```

### **Step-by-Step Process**

**For each stack item:**

1. **If `len(item) == 3` or `item[0] is None`**:
   - **Extract** left and right heights
   - **Check balance**: `abs(left - right) <= 1`
   - **Pop** current item
   - **Propagate** height `max(left, right) + 1` to parent
2. **If `len(item) == 1`**: Push left child `[node.left]`
3. **If `len(item) == 2`**: Push right child `[node.right]`

### **Visual Example**

Consider this tree:
```
    1
   / \
  2   3
 /
4
```

**Stack execution:**
```
Step  1: [[1]]                           → Push left child of 1
Step  2: [[1], [2]]                      → Push left child of 2
Step  3: [[1], [2], [4]]                 → Push left child of 4
Step  4: [[1], [2], [4], [None]]         → Node is none -> Height = 0
Step  5: [[1], [2], [4, 1]]              → Update height for 4's left child
Step  6: [[1], [2], [4, 1], [None]]      → Push right child of 4
Step  7: [[1], [2], [4, 1], [None]]      → Node is none -> Height = 0
Step  8: [[1], [2], [4, 1, 1]]           → Update height for 4's right child
Step  9: [[1], [2]]                      → Balance: |1-1|=0 ≤ 1 ✓
Step 10: [[1], [2, 2]]                   → Update height for 2's left child
Step 11: [[1], [2, 2], [None]]           → Push right child of 2
Step 12: [[1], [2, 2], [None]]           → Node is none -> Height = 0
Step 13: [[1], [2, 2, 1]]                → Update height for 2's right child
Step 14: [[1]]                           → Balance: |2-1|=1 ≤ 1 ✓
Step 15: [[1, 3]]                        → Update height for 1's left child
and so on for the right subtree of 1...
Step  n: [[1, 3, 2]]                     → Balance: |3-2|=1 ≤ 1 ✓
Final Step: Stack is empty, all nodes balanced

Finally: return True
```

### **Why This Works**

This approach **mimics post-order traversal** (left → right → root) iteratively:

- **Post-order** ensures we process children before parents
- **Dynamic stack items** store intermediate results (heights)
- **Early termination** on first imbalance detection
- **No recursion** means O(1) extra space beyond the stack

### **Complexity Analysis**

- **Time**: **O(n)** - Each node visited exactly once
- **Space**: **O(h)** - Stack depth equals tree height

The most complicated part of implementing this iterative approach is **tracking the child-to-parent relationship**. Some solutions attempt to use `map[child] = parent` but this approach is incorrect for tree nodes since we cannot use TreeNode as a key in map. Others serialize the tree to an array and use formulas like `parent_index = child_index // 2`, but this requires **O(2^h)** space in the worst case.

Our dynamic stack approach elegantly sidesteps this issue by **storing intermediate results directly in the stack items**, eliminating the need for explicit parent tracking. The child-to-parent relationship is implicitly maintained through the stack and the order of processing, we always know the parent of an item is the item right below it in the stack.
