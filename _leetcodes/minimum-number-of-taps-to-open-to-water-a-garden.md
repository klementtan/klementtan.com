---
title: "1326: Minimum Cost to Reach Destination in Time"
excerpt: Minimum Cost to Reach Destination in Time
problem_id: 1326 
categories:
  - Leet Code
tags:
  - Intervals 
---

## Problem

**Problem ID**: [1326](https://leetcode.com/problems/minimum-number-of-taps-to-open-to-water-a-garden/)

**Title**: Minimum Number of Taps to Open to Water a Garden

**Difficulty**: Hard

**Description**:

There is a one-dimensional garden on the x-axis. The garden starts at the point 0 and ends at the point n. (i.e The length of the garden is n).

There are n + 1 taps located at points [0, 1, ..., n] in the garden.

Given an integer n and an integer array ranges of length n + 1 where ranges[i] (0-indexed) means the i-th tap can water the area [i - ranges[i], i + ranges[i]] if it was open.

Return the minimum number of taps that should be open to water the whole garden, If the garden cannot be watered return -1.

## Thoughts

This is quite a tricky problem as it does not use the definition  of an interval. Intervals `(i, j)` and `(j+1, k)`  do not cover the entire range
from `i` to `k`.

The key to solving this question is to identify that it is a greedy problem. A model I like to use is to ask myself,
at point `i`, if I choose the interval that contains `i` and ends the latest, will the answer be wrong when I process
the intervals beyond `i`?

## Implementation

### `O(nlog(n))` solution
**General Idea**: Similar to [LC 56: merge interval](https://leetcode.com/problems/merge-intervals/) questions, we would 
sort the ranges from by the start of intervals.

**Handling edge case**: for `ranges=0` we would artificially set the range to `(-1,-1)` as it would not contribute
to the intervals.


```cpp
class Solution {
public:
    int minTaps(int n, vector<int>& ranges) {
        vector<pair<int,int>> intervals(ranges.size());
        for (int i = 0; i < ranges.size(); i++) {
            if (ranges[i] == 0) {
                // tap will not be able cover its own location so shift it to -1
                intervals[i] = make_pair(-1, -1);
            } else {
                intervals[i] = make_pair(i-ranges[i], i+ranges[i]); 
            }
        }
        sort(intervals.begin(), intervals.end());
        int next_uncovered;
        int count = 0;
        int curr_interval = 0;
        int last_covered = -1;

        for (int i = 0; i < intervals.size(); i++) {
            if (i < last_covered) continue;
            if (i == intervals.size() -1 && i <= last_covered) break;
            while (curr_interval < intervals.size()) {
                int start = intervals[curr_interval].first;
                int end = intervals[curr_interval].second;
                if (start > i) break;
                last_covered = max(last_covered, end);
                curr_interval++;
            }
            if (last_covered < i) return -1;
            count++;
        }
        return count;
    }
};
```
