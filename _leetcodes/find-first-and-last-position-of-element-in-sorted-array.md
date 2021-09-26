---
title: "34: Find First and Last Position of Element in Sorted Array"
excerpt: Find the first and last position.
problem_id: 34
categories:
  - Leet Code
tags:
  - Binary Search 
---

## Problem

**Problem ID**: [34](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

**Title**: Find First and Last Position of Element in Sorted Array

**Difficulty**: Medium

**Description**:
Given an array of integers nums sorted in ascending order, find the starting and 
ending position of a given target value.

## Thoughts

I have always struggled with implementing binary search well and found this problem
very challenging (22 failed submissions).

IMO, The main challenges with implementing binary search are:
* Ensuring that the left and right index are inside the range of the array
* Preventing an infinite loop

## Solution

### Explanation

**First position**: We can think of finding the first position of the element as moving the **right**
pointer **as far left as possible** while the value is more than or equal to the target.
We can achieve this with `m = (l+r)/2` and `r = m`. This ensures that the **right**
pointer will always progress towards the left.

**Last Position**: Similarly, the last position of the element is the position of the left pointer
after moving as far right as possible while the value is less than or equal to the
target. We can achieve this with `m = (l+r+1)/2` and `l = m`. This ensures that
the **left** pointer will always progress towards the right.

**Preventing infinite loop**: With `l = m + 1` (first position) and `r = m - 1` (last position) we can
ensure that the search space will always be reduced when the main pointer is not making any progression.


### Implementation

```cpp
class Solution {
public:
    // [1, 2, 3, 3, 3, 4, 5]
    int first(const vector<int>& nums, int target) {
        int l = 0;
        int r = nums.size() - 1;
        while (l < r) {
            int m = (l+r)/2;
            int m_val = nums[m];
            if (m_val >= target) {
                r = m;
            } else {
                l = m + 1;
            }
        }
        return nums[r] == target ? l : -1;
    }
    int last(const vector<int>& nums, int target) {
        int l = 0;
        int r = nums.size() - 1;
        while (l < r) {
            int m = (l+r+1)/2;
            int m_val = nums[m];
            if (m_val <= target) {
                l = m;
            } else {
                r = m -1;
            }
        }
        return nums[l] == target ? l : -1;
    }
    vector<int> searchRange(vector<int>& nums, int target) {
        if (nums.empty()) return {-1, -1};
        return {first(nums, target), last(nums,target)};
    }
};
```
