---
title: "448: Find All Numbers Disappeared in an Array"
excerpt: Find All Numbers Disappeared in an Array
problem_id: 448 
categories:
  - Leet Code
tags:
  - Array
---

## Problem 

**Problem ID**: [448](https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array/)

**Title**: Find All Numbers Disappeared in an Array

**Difficulty**: Easy 

**Description**:

Given an array nums of n integers where nums[i] is in the range [1, n], return an array of all the integers in the range [1, n] that do not appear in nums.

## Thoughts

This is a very simple problem but to solve it in O(1) space and constant time requires a neat trick that could be
reused for other more difficult problems. If you are not allowed to use extra space, you could use the provided input
as an additional space by changing the sign (positive or negative) of each element. With this you will be able to have
an extra boolean array without incurring more space.

Admittedly, I was not able to think of this and had to refer to the solution.

## Solution

### Implementation

```cpp
class Solution {
public:
    vector<int> findDisappearedNumbers(vector<int>& nums) {
       for (auto num : nums) {
          if(nums[abs(num)-1] > 0) nums[abs(num)-1] *= -1;
       } 
        vector<int> ret;
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] > 0) ret.push_back(i+1);
        }
        return ret;
    }
};
```
