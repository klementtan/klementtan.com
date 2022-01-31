---
title: "1073. Adding Two Negabinary Numbers"
excerpt: Adding Two Negabinary Numbers
problem_id: 1073 
categories:
  - Leet Code
tags:
  - Math
---

## Problem 

**Problem ID**: [1073](https://leetcode.com/problems/adding-two-negabinary-numbers/)

**Title**: Adding Two Negabinary Numbers

**Difficulty**: Medium

**Description**:

Given two numbers arr1 and arr2 in base -2, return the result of adding them together.

Each number is given in array format:  as an array of 0s and 1s, from most significant bit to least significant bit.  For example, arr = [1,1,0,1] represents the number (-2)^3 + (-2)^2 + (-2)^0 = -3.  A number arr in array, format is also guaranteed to have no leading zeros: either arr == [0] or arr[0] == 1.

Return the result of adding arr1 and arr2 in the same format: as an array of 0s and 1s with no leading zeros.



## Thoughts

I would classify this question as a iykyk kind of question as it hard to encounter for the case for negative number (ie: `-1`)

## Solution

Intuition:
* If both bit are set at the current bit index, it would borrow from the next 
bit. Irregardless if the current $$(-2)^{i}$$ is negative

  $$1*(-2)^i + 1*(-2)^i = -(-2)^{i+1}$$

* This is also similar if both the current bit index are set and there is 
carry from the previous.
* From the previous two points it should be obvious that the carry will minus from the next value instead of borrowing.
* The difficult part is when the bits and the sum adds up to $$-1$$. The 
following identity could be used.

  $$-1 = 1*(-2)^0 + 1*(-2)^2$$

### Implementation

```cpp
class Solution {
public:
    vector<int> addNegabinary(vector<int>& arr1, vector<int>& arr2) {
        vector<int> ret;
        int arr1_i = arr1.size() - 1;
        int arr2_i = arr2.size() - 1;
        int carry = 0;
        
        
        while (arr1_i >= 0 || arr2_i >= 0 || carry != 0) {
            int arr1_bit = arr1_i >= 0 ? arr1[arr1_i] : 0;
            int arr2_bit = arr2_i >= 0 ? arr2[arr2_i] : 0;
            int sum = carry + arr1_bit + arr2_bit;
            int ret_bit = sum & 1;
            carry = -(sum >> 1);
            ret.push_back(ret_bit);
            arr1_i--;
            arr2_i--;
        }
        
        if (carry > 0) ret.push_back(carry);
        while (ret.size() > 1 && ret.back() == 0) ret.pop_back();
        reverse(ret.begin(), ret.end());
        return ret;
    }
};

```
