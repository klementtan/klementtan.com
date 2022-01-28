---
title: "300: Longest Increasing Subsequence"
excerpt: Longest Increasing Subsequence
problem_id: 300
categories:
  - Leet Code
tags:
  - Binary Search
---

## Problem 

**Problem ID**: [300](https://leetcode.com/problems/longest-increasing-subsequence/)

**Title**: Longest Increasing Subsequence

**Difficulty**: Medium

**Description**:

Given an integer array nums, return the length of the longest strictly increasing subsequence.

A subsequence is a sequence that can be derived from an array by deleting some or no elements without changing the order of the remaining elements. For example, [3,6,2,7] is a subsequence of the array [0,3,1,6,2,2,7].

## Thoughts

This is an interesting problem that does not have an intuitive solution to it.

## Solution

**General Idea**

The key to solving this problem is observing that:
* The longest increasing subsequence will contain as many small numbers as possible
* We only need to keep track of the last number (largest so far) of when constructing the increasing subsequence

We solve this problem by tracking multiple subsequence in a single vector 
* If a number is greater than the last number in the subsequence, we will just append it to the end
* If a number is smaller than the last number, this means a new subsequence can begin
  * This new subsequence can continue from all previous number that are small than it
  * As all the numbers are increasing (sorted), we can use binary search to find the most element to start the new subsequence to allow the number.
* It is always guaranteed that the last element is part of a valid subsequence
* As we "greedily" add elements into the subsequence, the size of the subsequence is answer eventhough the elements
in it might not represent the exact valid subsequence

### Implementation

```cpp
class Solution {
public:
    int b_search(vector<int>& nums, int target) {
        int l = 0;
        int r = nums.size() -1;
        while (l < r) {
            int m = (l+r)/2;
            if (nums[m] >= target) {
                r = m;
            } else {
                l = m+1;
            }
        }
        return r;
    }
    int lengthOfLIS(vector<int>& nums) {
        vector<int> seq;
        for (int num : nums) {
            cout << endl;
            if (seq.empty() || seq.back() < num) {
                seq.push_back(num);
                continue;
            }
            auto i = b_search(seq, num);
            if (seq[i] == num) continue;
            seq[i] = num;
        }
        return seq.size();
    }
};
```
