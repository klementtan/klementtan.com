---
title: "239: Sliding Window Maximum"
excerpt: Sliding Window Maximum
problem_id: 239 
categories:
  - Leet Code
tags:
  - Leet Code
---

## Problem

**Problem ID**: [239](https://leetcode.com/problems/sliding-window-maximum/)

**Title**: Sliding Window Maximum

**Difficulty**: Hard

**Description**:

<p>You are given an array of integers&nbsp;<code>nums</code>, there is a sliding window of size <code>k</code> which is moving from the very left of the array to the very right. You can only see the <code>k</code> numbers in the window. Each time the sliding window moves right by one position.</p>

<p>Return <em>the max sliding window</em>.</p>

<p>&nbsp;</p>
<p><strong>Example 1:</strong></p>

<pre>
<strong>Input:</strong> nums = [1,3,-1,-3,5,3,6,7], k = 3
<strong>Output:</strong> [3,3,5,5,6,7]
<strong>Explanation:</strong> 
Window position                Max
---------------               -----
[1  3  -1] -3  5  3  6  7       <strong>3</strong>
 1 [3  -1  -3] 5  3  6  7       <strong>3</strong>
 1  3 [-1  -3  5] 3  6  7      <strong> 5</strong>
 1  3  -1 [-3  5  3] 6  7       <strong>5</strong>
 1  3  -1  -3 [5  3  6] 7       <strong>6</strong>
 1  3  -1  -3  5 [3  6  7]      <strong>7</strong>
</pre>

<p><strong>Example 2:</strong></p>

<pre>
<strong>Input:</strong> nums = [1], k = 1
<strong>Output:</strong> [1]
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= nums.length &lt;= 10<sup>5</sup></code></li>
	<li><code>-10<sup>4</sup> &lt;= nums[i] &lt;= 10<sup>4</sup></code></li>
	<li><code>1 &lt;= k &lt;= nums.length</code></li>
</ul>


## Thoughts

Finding the O(N) solution to this question is quite hard and it was obvious that there is a O(N) solution.

## Solution

**General Idea**

To solve this problem you will need to make the following key observations:
1. When adding number at index `i` into the sliding window, any number, `j`, with index `j < i` and smaller than
the new number `nums[j] < nums[i]` will never be the maximum number for the window.
  * The reason for this is that all subsequent window that has `j` will also have `i`
  and `nums[i]` will be the maximum number

Intuition:
1. Carry out normal sliding window where we iterate through each element and add it to the
queue. If the queue size is bigger than the window size pop from the front of the queue.
2. Instead of having a FIFO queue, the queue will represent all the possible largest
element for now and all other future iteration.
    1. This is constructed by poping from the back of the queue if the back element
    is smaller than the current element. Represent a previous candidate being
    replaced by the current element.


### Implementation

```cpp
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        
        deque<int> d;
        vector<int> ret;
        for (auto i = size_t{0}; i < nums.size(); i++) {
            while (!d.empty() && i - d.front() >= k) d.pop_front();
            while (!d.empty() && nums[d.back()] < nums[i]) d.pop_back();
            d.push_back(i);
            if (i >= k -1) ret.push_back(nums[d.front()]);
        }
        return ret;
    }
};
```
