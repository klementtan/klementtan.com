---
title: "347: Top K Frequent Elements"
excerpt: Get the K most frequent element
problem_id: 347
categories:
  - Leet Code
tags:
  - DP
---

## Problem

**Problem ID**: [347](https://leetcode.com/problems/top-k-frequent-elements/)

**Title**: Top K Frequent Elements

**Difficulty**: Medium

**Description**:
Given an integer array nums and an integer `k`, return the `k` most frequent
elements. You may return the answer in any order.

## Thoughts

This problem is relatively straight forward with C++'s ordered set.

## Solution

Find the frequency for each number `O(N)`. Insert the number and frequency into
an ordered set that orders the number by the frequency. Whenever the size of the
set exceeds `k` remove the number with the lowest frequency. 

### Implementation

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int,int> num_freq;
        for (const int num : nums) num_freq[num]++;
        
        set<pair<int,int>> most_freq;
        for (const auto& [num, freq] : num_freq) {
            most_freq.insert({freq, num});
            if (most_freq.size() > k) {
                most_freq.erase(most_freq.begin());
            }
        }

        vector<int> ans;
        for (const auto& [freq, num] : most_freq) {
            ans.push_back(num);            
        }
        return ans;
    }
};
```
