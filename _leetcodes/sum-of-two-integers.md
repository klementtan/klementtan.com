---
title: "371: Sum of Two Integers"
excerpt: Add two integers without +/-
problem_id: 371
categories:
  - Leet Code
tags:
  - Bit
---

## Problem

**Problem ID**: [371](https://leetcode.com/problems/sum-of-two-integers/)

**Title**: Sum of Two Integers

**Description**: Given two integers a and b, return the sum of the two integers
without using the operators + and -.


## Thoughts

This problem is a good revision for bitwise arithmetic.


## Solution

### Explanation

Instead of manually evaluating the individual bit, we can recursively calculate
the new sum without carry and swap the other operand with the carry.

For example:

Adding:
```
a: 0 0 1 0 1 1 0 1
b: 0 0 1 1 0 1 1 0
```

New sum without carry (`a^b`):
```
0 0 0 1 1 0 1 0
```
Carry(`a&b << 1`):
```
0 1 0 0 1 0 0 0
```
Now the answer becomes `sum(a^b, a&b <<1)`

**Implementation**

```cpp
class Solution {
public:
    int getSum(int x, int y) {
        
    // Iterate till there is no carry
    while (y != 0)
    {
        unsigned carry = x & y;

        x = x ^ y;

        y = carry << 1;
    }
    return x;
        
    }
};
```

