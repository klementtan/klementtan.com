---
title: "136: Single Number"
excerpt: Single Number
problem_id: 136 
categories:
  - Leet Code
tags:
  - Bit 
---
## Problem

**Problem ID**: [136](https://leetcode.com/problems/single-number/)

**Title**: Single Number

**Difficulty**: Easy

**Description**:

Given a non-empty array of integers nums, every element appears twice except for one. Find that single one.

You must implement a solution with a linear runtime complexity and use only constant extra space.

## Thoughts

Solving this problem with linear space is easy but to do it in constant space is tricky. Using XOR technique
to remove identical numbers is a good technique

## Solution

**General Idea**

As `k ^ k = 0`, we will just need to continuously XOR across all numbers.

### Implementation

```cpp
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int ret = 0;
        for (auto num : nums) ret ^= num;
        return ret;
    }
};
```

