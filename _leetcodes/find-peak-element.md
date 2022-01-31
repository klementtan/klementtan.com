---
title: "162. Find Peak Element"
excerpt: Find Peak Element
problem_id: 162 
categories:
  - Leet Code
tags:
  - Binary Search 
---

## Problem

**Problem ID**: [162](https://leetcode.com/problems/find-peak-element/)

**Title**: Find Peak Element

**Difficulty**: Medium

**Description**:

A peak element is an element that is strictly greater than its neighbors.

Given an integer array nums, find a peak element, and return its index. If the array contains multiple peaks, return the index to any of the peaks.

You may imagine that nums[-1] = nums[n] = -∞.

You must write an algorithm that runs in O(log n) time.

## Thoughts

Even though this problem has a lot of down votes, the method of using binary
search to solve this problem is quite ingenius

## Solution

**General Idea**

A peak element is a point where the left side of the point is ascending while the right hand side is descending. 

As the left of the left most element `[-1]` and right of the right most element `[n]` are `-INF`, the left most element will always be ascending while the right most element will always be descending.

Thus starting with the left pointer at the left most element and the right at the right most element, we will keep trying to closen the gap between the right and left pointer while mainting the ascending and descending property. Eventually when the left and right pointer meet at the same element, that element is ascending from the left and descending to the right. Thus making that element the peak.

### Implementation

```cpp
class Solution {
public:
    int findPeakElement(vector<int>& nums) {
        int l = 0;
        int r = nums.size() -1;
        while (l < r) {
            int m = (l+r)/2;
            if (nums[m] > nums[m+1]) {
                r = m;
            } else {
                l = m + 1;
            }
        }
        return l;
    }
};
```
