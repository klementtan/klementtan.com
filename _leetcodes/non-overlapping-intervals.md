---
title: "435: Non-overlapping Intervals"
excerpt: Minimal intervals to remove to get non-overlapping intervals
problem_id: 435 
categories:
  - Leet Code
tags:
  - Greedy
  - Intervals
---

## Problem

**Problem ID**: [435](https://leetcode.com/problems/non-overlapping-intervals/)

**Title**: Non-overlapping Intervals

**Difficulty**: Medium

**Description**:
Given an array of intervals intervals where `intervals[i] = [start_i, end_i]`,
return the minimum number of intervals you need to remove to make the rest of 
the intervals non-overlapping.

## Thoughts

This problem can be reduced to finding longest non-overlapping interval which
could be solved greedily.

I like to use the following framework to determine if the problem can be solved
greedily. When trying to solve a problem of a smaller size, do we need to choose
a less optimal solution to allow for an optimal solution in the future?

If no, the problem can be solved greedily. For the longest non-overlapping interval
example, choosing longest non-overlapping interval for the first `i` interval will not
affect the solution for all the intervals.


## Solution

### Implementation

```cpp
class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        
        vector<pair<int,int>> ordered_intervals; // <end, start>
        for (const vector<int>& interval : intervals) {
            ordered_intervals.push_back({interval[1], interval[0]});
        }
        sort(ordered_intervals.begin(), ordered_intervals.end());
        int next_avail = -INT_MAX;
        vector<pair<int,int>> nonoverlap_interval;
        for (auto [end, start] : ordered_intervals) {
            if (start >= next_avail) {
                nonoverlap_interval.push_back({start, end});
                next_avail = end;
            }
        }
        return intervals.size() - nonoverlap_interval.size();
    }
};
```
