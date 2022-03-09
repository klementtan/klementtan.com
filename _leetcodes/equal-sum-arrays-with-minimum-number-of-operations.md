---
title: "1775: Equal Sum Arrays With Minimum Number of Operations"
excerpt: Equal Sum Arrays With Minimum Number of Operations
problem_id: 1175 
categories:
  - Leet Code
tags:
  - Greedy
---

## Problem 

**Problem ID**: [1775](https://leetcode.com/problems/equal-sum-arrays-with-minimum-number-of-operations/)

**Title**: Equal Sum Arrays With Minimum Number of Operations

**Difficulty**: Medium

**Description**:

You are given two arrays of integers nums1 and nums2, possibly of different lengths. The values in the arrays are between 1 and 6, inclusive.

In one operation, you can change any integer's value in any of the arrays to any value between 1 and 6, inclusive.

Return the minimum number of operations required to make the sum of values in nums1 equal to the sum of values in nums2. Return -1 if it is not possible to make the sum of the two arrays equal.

## Thoughts

This was a hard problem for me and I had to look at the solution.

## Solution

**General Idea**

Find the bigger and smaller numbers from `nums1` and `nums2`. The num `6` on the bigger and `1` on the smaller. Both of them have same impact on making the sums equal as they both are able to reduce the difference by `5`. This also applies to `5` and `2` and so on and so forth.

We can then greedily reduce the difference in sum by changing the pair with the highest impact (`1` and `6`). If converting all the nums of a `pair` is insufficient we will then move on the next pair (`2` and `5`) else we would use all the necessary current pair and if there are extra add extra to it.

```cpp
class Solution {
public:
    int minOperations(vector<int>& nums1, vector<int>& nums2) {
        int sum1 = 0;
        int sum2 = 0;
        for (int num : nums1) sum1 += num;
        for (int num : nums2) sum2 += num;
        
        std::array<int, 6> contributions = {0};
        
        vector<int>& bigger = sum1 > sum2 ? nums1 : nums2;
        vector<int>& smaller = sum1 > sum2 ? nums2 : nums1;
        
        for (auto num : bigger) contributions[num - 1]++;
        for (auto num : smaller) contributions[6 - num]++;
        
        int delta = abs(sum1 - sum2);
        int ret = 0;
        for (int contrib = 5; delta > 0 && contrib >= 1; contrib--) {
            int take = min(contributions[contrib], delta/contrib + (delta % contrib != 0));
            bool should_break = take == delta/contrib + (delta % contrib != 0);
            delta -= take*contrib;
            ret += take;
            if (should_break) break;
        }

        return delta > 0 ? -1 : ret;
    }
};
```
