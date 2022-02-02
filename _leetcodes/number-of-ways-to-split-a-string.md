---
title: "1573 Number of Ways to Split a String"
excerpt: Number of Ways to Split a String
problem_id: 1573
categories:
  - Leet Code
tags:
  - Math
---

## Problem 

**Problem ID**: [1573](https://leetcode.com/problems/number-of-ways-to-split-a-string/)

**Title**: Number of Ways to Split a String

**Difficulty**: Medium

**Description**:

Given a binary string s, you can split s into 3 non-empty strings s1, s2, and s3 where s1 + s2 + s3 = s.

Return the number of ways s can be split such that the number of ones is the same in s1, s2, and s3. Since the answer may be too large, return it modulo 109 + 7.

## Solution

**General Idea**

We can solve this by finding the bounds for the first and second segment. However, we will need to handle different cases:
1. Number of ones not divisible by 3
  * Return `0`
2. All zeroes
  * No clear difference between the left and right bound
  * Return `(n-1)(n-2)/2`
3. Everything else
  * Return number of left bounds * number of right bounds

### Implementation

```cpp
class Solution {
public:
    int numWays(string s) {
        size_t n = s.size();
        vector<size_t> pre_ones(n, 0);
        size_t n_ones{0};
        
        for (size_t i = 0; i < n; i++) {
            pre_ones[i] = s[i] - '0';
            if (i > 0) pre_ones[i] += pre_ones[i-1];
            if (s[i] - '0') n_ones++;
        }
        if (n_ones % 3 != 0) return 0;
        size_t l_ones = n_ones/3;
        size_t r_ones = l_ones*2;
        size_t l_n{0};
        size_t r_n{0};
        for (auto bound : pre_ones) {
            if (bound == l_ones) l_n++;
            if (bound == r_ones) r_n++;
        }
        size_t ret_64{0};
        l_n = l_n % static_cast<size_t>(1e9+7);
        r_n = r_n % static_cast<size_t>(1e9+7);
        if (l_ones == r_ones) {
            ret_64 = ((l_n-1)*(l_n -2))/2;
        } else {
            ret_64 = (l_n)*(r_n);
        }
      
        return static_cast<int>(ret_64 % static_cast<size_t>(1e9+7));
    }
};
```
