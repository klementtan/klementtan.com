---
title: "259: 3Sum Smaller"
excerpt: 3Sum Smaller
problem_id: 259 
categories:
  - Leet Code
tags:
  - Leet Code
---

## Problem

**Problem ID**: [259](https://leetcode.com/problems/3sum-smaller/)

**Title**: 3Sum Smaller

**Difficulty**: Medium

**Description**:



## Thoughts


## Solution

**General Idea**

The main observation: for 2 sum smaller, if there exists two pointer such that the sum
is less than the target, moving the right pointer all the way until right before the left
pointer will give the smaller.

ie:
```
target = 9
[1, 2, 3, 5, 8]
 ↑        ↑
left    right
```
* (1,5), (1,3), (1,2) are valid combinations and we can easily calculate this 
by right - left


### Implementation

```cpp
class Solution {
public:
    int threeSumSmaller(vector<int>& nums, int target) {
        
        auto n = nums.size();
        int ret = 0;
        sort(nums.begin(), nums.end());
        
        for (auto i = size_t{0}; i < n; i++) {
            auto j = i + 1;
            auto k = n - 1;
            while (j < k) {
                int sum = nums[i] + nums[j] + nums[k];
                if (sum < target) {
                    ret += (k - j);
                    j++;
                } else {
                   k--; 
                }
            }
        }
        return ret;
    }
};
```
