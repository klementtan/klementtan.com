---
title: "41: First Missing Positive"
excerpt: First Missing Positive
problem_id: 41 
categories:
  - Leet Code
tags:
  - Leet Code
---

## Problem

**Problem ID**: [41](https://leetcode.com/problems/first-missing-positive/)

**Title**: First Missing Positive

**Difficulty**: Hard

**Description**:

<p>Given an unsorted integer array <code>nums</code>, return the smallest missing positive integer.</p>

<p>You must implement an algorithm that runs in <code>O(n)</code> time and uses constant extra space.</p>

<p>&nbsp;</p>
<p><strong>Example 1:</strong></p>
<pre><strong>Input:</strong> nums = [1,2,0]
<strong>Output:</strong> 3
</pre><p><strong>Example 2:</strong></p>
<pre><strong>Input:</strong> nums = [3,4,-1,1]
<strong>Output:</strong> 2
</pre><p><strong>Example 3:</strong></p>
<pre><strong>Input:</strong> nums = [7,8,9,11,12]
<strong>Output:</strong> 1
</pre>
<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= nums.length &lt;= 5 * 10<sup>5</sup></code></li>
	<li><code>-2<sup>31</sup> &lt;= nums[i] &lt;= 2<sup>31</sup> - 1</code></li>
</ul>


## Thoughts

This is an interesting problem that utilises discreet math.

## Solution

**General Idea**

Key Observations:
* By pigeon hole principal, the answer must be `1 <= Ans <= N + 1`. Each positive
number is assigned the element `num - 1`. In the worst case scenario all elements
in the vector are assigned a number and `Ans = N+1` otherwise the answer is the
first element that does not have a corresponding number.
* All non-positive numbers will not affect the answer and can be modified to any number
that is non-positive or already in the set (can set as a flag).

Intuition: each valid number (`1 <= num <= N`) will be assigned to the element
`num - 1`. For each element, if there is a corresponding number then the value is `< 0`.
Finally iterate through all the elements, if any of the element is not `< 0` then return the index
as the answer.

Algorithm:
1. Sanitize non-positive number: set all non-positive number to `0` so that they
would not falsely represent an element with corresponding number.
2. For each absolute value of the number that is valid (`!= 0 && <= N`), set the
value to negative.
  * Absolute: the number could be negative when a separate number that corresponding to that
  element it is on is present
3. Iterate through all elements and find the first element that is negative. That index represent the answer.

### Implementation

```cpp
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        for (int& num : nums) {
            num = max(num, 0);
        }
        int n = nums.size();
        for (int i = 0; i < n; i++) {
            int num = abs(nums[i]);
            if (num <= 0 || num > n) continue;
            if (nums[num - 1] == 0) {
                nums[num - 1] = -(n+1);
            } else {
                nums[num - 1] = -abs(nums[num - 1]);
            }
        }
        for (int i = 0; i < n; i++) {
            if (nums[i] >= 0) return i + 1;
        }
        return n + 1;
    }
};
```
