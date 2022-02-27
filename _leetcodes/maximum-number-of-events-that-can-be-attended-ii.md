---
title: "1851: Maximum Number of Events That Can Be Attended II"
excerpt: Maximum Number of Events That Can Be Attended II
problem_id: 1851 
categories:
  - Leet Code
tags:
  - Leet Code
---

## Problem

**Problem ID**: [1851](https://leetcode.com/problems/maximum-number-of-events-that-can-be-attended-ii/)

**Title**: Maximum Number of Events That Can Be Attended II

**Difficulty**: Hard

**Description**:

<p>You are given an array of <code>events</code> where <code>events[i] = [startDay<sub>i</sub>, endDay<sub>i</sub>, value<sub>i</sub>]</code>. The <code>i<sup>th</sup></code> event starts at <code>startDay<sub>i</sub></code><sub> </sub>and ends at <code>endDay<sub>i</sub></code>, and if you attend this event, you will receive a value of <code>value<sub>i</sub></code>. You are also given an integer <code>k</code> which represents the maximum number of events you can attend.</p>

<p>You can only attend one event at a time. If you choose to attend an event, you must attend the <strong>entire</strong> event. Note that the end day is <strong>inclusive</strong>: that is, you cannot attend two events where one of them starts and the other ends on the same day.</p>

<p>Return <em>the <strong>maximum sum</strong> of values that you can receive by attending events.</em></p>

<p>&nbsp;</p>
<p><strong>Example 1:</strong></p>

<p><img alt="" src="https://assets.leetcode.com/uploads/2021/01/10/screenshot-2021-01-11-at-60048-pm.png" style="width: 400px; height: 103px;" /></p>

<pre>
<strong>Input:</strong> events = [[1,2,4],[3,4,3],[2,3,1]], k = 2
<strong>Output:</strong> 7
<strong>Explanation: </strong>Choose the green events, 0 and 1 (0-indexed) for a total value of 4 + 3 = 7.</pre>

<p><strong>Example 2:</strong></p>

<p><img alt="" src="https://assets.leetcode.com/uploads/2021/01/10/screenshot-2021-01-11-at-60150-pm.png" style="width: 400px; height: 103px;" /></p>

<pre>
<strong>Input:</strong> events = [[1,2,4],[3,4,3],[2,3,10]], k = 2
<strong>Output:</strong> 10
<strong>Explanation:</strong> Choose event 2 for a total value of 10.
Notice that you cannot attend any other event as they overlap, and that you do <strong>not</strong> have to attend k events.</pre>

<p><strong>Example 3:</strong></p>

<p><strong><img alt="" src="https://assets.leetcode.com/uploads/2021/01/10/screenshot-2021-01-11-at-60703-pm.png" style="width: 400px; height: 126px;" /></strong></p>

<pre>
<strong>Input:</strong> events = [[1,1,1],[2,2,2],[3,3,3],[4,4,4]], k = 3
<strong>Output:</strong> 9
<strong>Explanation:</strong> Although the events do not overlap, you can only attend 3 events. Pick the highest valued three.</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= k &lt;= events.length</code></li>
	<li><code>1 &lt;= k * events.length &lt;= 10<sup>6</sup></code></li>
	<li><code>1 &lt;= startDay<sub>i</sub> &lt;= endDay<sub>i</sub> &lt;= 10<sup>9</sup></code></li>
	<li><code>1 &lt;= value<sub>i</sub> &lt;= 10<sup>6</sup></code></li>
</ul>


## Thoughts

I had difficulty solving this problem as initially I was not sure how to optimise for `value` and end time for the DP transition.
Knowing that `k*events.length < 1e6` was a big hint as it implies that the problem should be solved using a 2DP of `n` times
`k`.

## Solution

**General Idea**
We will use bottom up DP to solve this problem with the following state and transitions.
The reason we have to use DP instead of greedy is because choosing event `i` could result
in later overlapping events with large value(`INF`) being passed on but passing on event `i` could
result in a wrong answer if event `i` has (`INF`) value.

**DP State** - `dp[i][j]`
* Represents the maximum score at the end of `events[i]`'s end time with `j` events selected.

**DP Transition** - when processing event `i` we could:
* Choosing current event:
  * Logically, this means choosing the current events with the maximum valid combination of previous
  events that do not overlap with the current event.
  * The value will be equals to `events[i]`'s value sum with `dp[k][j-1]` where `k` is the event with
  end time strictly less than the current event start time.
  * `dp[k][j-1]`: represents a combination of `j-1` previous events that has the maximum value and
  do not overlap with the current event
* Passing on the current event:
  * Value Equals to `dp[i-1][j]` as it represents the combination of previous `j` events
* Final dp transition: `dp[i][j] = max(dp[i-1][j], events[i][2] + dp[k][j-1])`


### Implementation

```cpp
class Solution {
public:
    int maxValue(vector<vector<int>>& events, int k) {
        events.push_back({0,0,0});
        int n = events.size();
        sort(events.begin(), events.end(), [](const auto& a, const auto& b){return a[1] < b[1];});
        vector<int> end_times(n,0);
        for (int i = 0; i < n; i++) end_times[i] = events[i][1];

        vector<vector<int>> dp(n, vector<int>(k+1, 0));
        int ret = 0;
        for (int i = 1; i < n; i++) {
            for (int j = 1; j < k+1; j++) {
                auto it = lower_bound(end_times.begin(), end_times.end(), events[i][0]);
                assert(it != end_times.end());
                assert(it != end_times.begin());
                it--;
                int k = std::distance(end_times.begin(), it);
                dp[i][j] = max(events[i][2] + dp[k][j-1], dp[i-1][j]);
                ret = max(ret, dp[i][j]);
            }
        }
        return ret;
    }
};
```
