---
title: "173: Binary Search Tree Iterator"
excerpt: Binary Search Tree Iterator
problem_id: 173 
categories:
  - Leet Code
tags:
  - Tree 
---

## Problem

**Problem ID**: [173](https://leetcode.com/problems/binary-search-tree-iterator/)

**Title**: Binary Search Tree Iterator

**Difficulty**: Medium

**Description**:
Implement the BSTIterator class that represents an iterator over the in-order traversal of a binary search tree (BST):

BSTIterator(TreeNode root) Initializes an object of the BSTIterator class. The root of the BST is given as part of the constructor. The pointer should be initialized to a non-existent number smaller than any element in the BST.
boolean hasNext() Returns true if there exists a number in the traversal to the right of the pointer, otherwise returns false.
int next() Moves the pointer to the right, then returns the number at the pointer.
Notice that by initializing the pointer to a non-existent smallest number, the first call to next() will return the smallest element in the BST.

You may assume that next() calls will always be valid. That is, there will be at least a next number in the in-order traversal when next() is called.


## Thoughts

This is a very interesting problem. This problem teaches you how to traverse a BST and
simulate a call stack without recursion.

## Solution

To iterate through a BST, we will need to carry out inorder traversal of the tree. However,
the we cannot use the traditional way relying on the call stack to perform inorder traversal.

We can simulate the call stack with the following operations:

1. Try to traverse a node as far left as possible
2. Once there does not exist a left subtree, we will print the value of the current node.
3. We no longer need the node and will only require the right subtree
4. We will then proceed to process the right subtree.
5. We will also maintain a visited hashset to make sure that we do not process the same left subtree

### Implementation

```cpp
class BSTIterator {
public:
    stack<TreeNode*> call_stack;
    unordered_set<TreeNode*> visited;
    
    BSTIterator(TreeNode* root) {
        call_stack.push(root);    
    }
    
    int next() {
        while (
                !call_stack.empty()
                && call_stack.top()->left
                && visited.count(call_stack.top()->left) == 0
              ) {
            call_stack.push(call_stack.top()->left);
        }
        int ret = call_stack.top()->val;
        visited.insert(call_stack.top());
        TreeNode* right = call_stack.top()->right;
        call_stack.pop();
        if (right) call_stack.push(right);
        return ret;
    }
    
    bool hasNext() {
       return !call_stack.empty();
    }
};
```
