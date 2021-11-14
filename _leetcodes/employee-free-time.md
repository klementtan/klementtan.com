---
title: "759: Employee Free Time"
excerpt: Employee Free Time
problem_id: 759
categories:
  - Leet Code
---

## Problem

**Problem ID**: [759](https://leetcode.com/problems/employee-free-time/)

**Title**: Employee Free Time

**Difficulty**: Hard

**Description**:
We are given a list schedule of employees, which represents the working time for each employee.

Each employee has a list of non-overlapping Intervals, and these intervals are in sorted order.

Return the list of finite intervals representing common, positive-length free time for all employees, also in sorted order.

(Even though we are representing Intervals in the form [x, y], the objects inside are Intervals, not lists or arrays. For example, schedule[0][0].start = 1, schedule[0][0].end = 2, and schedule[0][0][0] is not defined).  Also, we wouldn't include intervals like [5, 5] in our answer, as they have zero length.

## Thoughts

For this problem, I utilised a technique used for [Hackerrank's Sprint Training](https://leetcode.com/discuss/interview-question/1377265/hackerrank-question) question.
The technique of storing the delta of intervals is very useful when you want to compute the number of overlapped intervals.

## Solution

**General Idea**: For each time (start/end) we will store the change in number of employees that will become busy. This will
let us easily simulate moving through time and detecting any time intervals when none of the employees are busy (same as all employees free).

Data Structure: `map<int, int> busy_times` stores the `time`: delta in number of busy employee.
We use ordered map so that we can iterate through the busy times in ascending order. 

Algorithm:
* Populating `busy_times`
    * As each interval is disjoint, this means that at `Interval.start` an 
    employee becomes busy (`busy_times[Interval.start]++`) and at `Interval.end` 
     an employee becomes free (`busy_times[Interval.end]--`).
* Getting free intervals:
  * We will store the `prev_time` that stores the previous processed time and `busy_count` that tracks the number of
  busy employee
  * Iterate through `busy_times`
      * if the previous `busy_count=0`, it signifies that it is the end of a all employee free interval.
      * Update the `prev_time` with the current time and `busy_count` with the current delta.


### Implementation

```cpp
class Solution {
public:
    vector<Interval> employeeFreeTime(vector<vector<Interval>> schedule) {
        map<int, int> busy_times;
        int n = schedule.size();
        for (const auto& emp_sch : schedule) {
            for (const auto& interval : emp_sch) {
                busy_times[interval.start]++;
                busy_times[interval.end]--;
            }
        }
        
        vector<Interval> ret;
        int prev_time = -1;
        int busy_count = -1;
        for (const auto& [time, busy_delta] : busy_times) {
            if (prev_time == -1) {
                prev_time = time;
                busy_count = busy_delta;
                continue;
            }
            if (busy_count == 0) {
                ret.emplace_back(prev_time, time);
            }
            busy_count += busy_delta;
            prev_time = time;
        }
        return ret;
    }
};
```
