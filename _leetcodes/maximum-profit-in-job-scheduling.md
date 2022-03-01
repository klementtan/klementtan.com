---
title: "1352: Maximum Profit in Job Scheduling"
excerpt: Maximum Profit in Job Scheduling
problem_id: 1352 
categories:
  - Leet Code
tags:
  - DP
---

## Problem

**Problem ID**: [1352](https://leetcode.com/problems/maximum-profit-in-job-scheduling/)

**Title**: Maximum Profit in Job Scheduling

**Difficulty**: Hard

**Description**:

<p>We have <code>n</code> jobs, where every job is scheduled to be done from <code>startTime[i]</code> to <code>endTime[i]</code>, obtaining a profit of <code>profit[i]</code>.</p>

<p>You&#39;re given the <code>startTime</code>, <code>endTime</code> and <code>profit</code> arrays, return the maximum profit you can take such that there are no two jobs in the subset with overlapping time range.</p>

<p>If you choose a job that ends at time <code>X</code> you will be able to start another job that starts at time <code>X</code>.</p>

<p>&nbsp;</p>
<p><strong>Example 1:</strong></p>

<p><strong><img alt="" src="https://assets.leetcode.com/uploads/2019/10/10/sample1_1584.png" style="width: 380px; height: 154px;" /></strong></p>

<pre>
<strong>Input:</strong> startTime = [1,2,3,3], endTime = [3,4,5,6], profit = [50,10,40,70]
<strong>Output:</strong> 120
<strong>Explanation:</strong> The subset chosen is the first and fourth job. 
Time range [1-3]+[3-6] , we get profit of 120 = 50 + 70.
</pre>

<p><strong>Example 2:</strong></p>

<p><strong><img alt="" src="https://assets.leetcode.com/uploads/2019/10/10/sample22_1584.png" style="width: 600px; height: 112px;" /> </strong></p>

<pre>
<strong>Input:</strong> startTime = [1,2,3,4,6], endTime = [3,5,10,6,9], profit = [20,20,100,70,60]
<strong>Output:</strong> 150
<strong>Explanation:</strong> The subset chosen is the first, fourth and fifth job. 
Profit obtained 150 = 20 + 70 + 60.
</pre>

<p><strong>Example 3:</strong></p>

<p><strong><img alt="" src="https://assets.leetcode.com/uploads/2019/10/10/sample3_1584.png" style="width: 400px; height: 112px;" /></strong></p>

<pre>
<strong>Input:</strong> startTime = [1,1,1], endTime = [2,3,4], profit = [5,6,4]
<strong>Output:</strong> 6
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= startTime.length == endTime.length == profit.length &lt;= 5 * 10<sup>4</sup></code></li>
	<li><code>1 &lt;= startTime[i] &lt; endTime[i] &lt;= 10<sup>9</sup></code></li>
	<li><code>1 &lt;= profit[i] &lt;= 10<sup>4</sup></code></li>
</ul>


## Thoughts

This question is similar to [1851](https://leetcode.com/problems/maximum-number-of-events-that-can-be-attended-ii/) where you need to use DP to optimise for both value and end time. Tried to use `size_t` for all indexes but it leads to integer underflow and had to add an underflow check before `-1`. Not sure what is the cleaner way to do binary search with `size_t`.

## Solution

**General Idea**

We treat all `endTime`s as the only possible times in the problem. When deciding on whether or not to choose a job, we can either choose it with the best possible combinations of jobs with `endTime < starTime` or not choose it and have the profit set to the previously calculated best combinations of jobs.


**DP State**: `dp[i]` - represents the best profit as of `jobs[i].endTime`

**DP transitions**: We can either
* Drop the current job: `dp[i] = dp[i-1]`
* Choose the current job: `dp[i] = dp[k] + jobs[i].profit` where `k` is the largest index such that `jobs[k].endTime <= jobs[i].startTime`

### Implementation

```cpp
struct Job {
    int start;
    int end;
    int profit;
};
class Solution {
public:
    int jobScheduling(vector<int>& startTime, vector<int>& endTime, vector<int>& profit) {
        auto n = startTime.size();
        vector<Job> jobs(n);
        for (size_t i = 0; i < n; i++) {
            jobs[i] = {startTime[i], endTime[i], profit[i]};
        }
        
        sort(jobs.begin(), jobs.end(), [](const auto& a, const auto& b) { return a.end < b.end; });
        vector<int> dp(n, 0);
        
        for (auto i = 0; i < n; i++) {
            int prev_profit = i > 0 ? dp[i-1] : 0;
            const auto [start, end, profit] = jobs[i];
            size_t l = 0;
            size_t r = n-1;
            while (l < r) {
                size_t m = (l+r + 1)/2;
                if (jobs[m].end <= start) {
                    l = m;
                } else {
                    r = m;
                    if (r != 0) r--;
                }
            }
            int curr_profit = profit;
            if (jobs[l].end <= jobs[i].start) curr_profit += dp[l];
            dp[i] = max(curr_profit, prev_profit);
        }
        return dp[n-1];
    }
};
```
