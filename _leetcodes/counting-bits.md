
---
title: "LC: Counting Bits"
excerpt: Count the number of bits for 0 to n
categories:
  - Leet Code
tags:
  - DP
---

## Problem

[Counting Bits](https://leetcode.com/problems/counting-bits/) Given an integer
`n`, return an array ans of length `n + 1` such that for each `i (0 <= i <= n)`,
`ans[i]` is the number of 1's in the binary representation of `i`.

## Thoughts

Even though this question is LC easy question, I was not able to derive the most
optimal solution without looking at the solution. Admittedly, I was intellectually
lazy to push myself to think of the most optimal solution and went with the naive
solution of counting the number of bits for each number using `while(num > 0){num = num & (num-1);}`
method. 

## Solution

The number of set bit for `i` is either equal to the number of set bit of `i/2` if
`i` is even else it is 1 more than the number of set bit of `i-1`. With this information
we can easily derive the following dp solution.

### Implementation

```cpp
class Solution {
public:

    vector<int> countBits(int n) {
        vector<int>ans;
        ans.push_back(0);
        
        for (int i = 1; i <= n; i++) {
            if (i&1) {
                ans.push_back(ans[i-1] + 1);
            } else {
                ans.push_back(ans[i/2]);
            }
        }
        return ans;
    }
};
```
