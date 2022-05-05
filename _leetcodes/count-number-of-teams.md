---
title: "1511: Count Number of Teams"
excerpt: Count Number of Teams
problem_id: 1511 
categories:
  - Leet Code
tags:
  - Leet Code
---

## Problem

**Problem ID**: [1511](https://leetcode.com/problems/count-number-of-teams/)

**Title**: Count Number of Teams

**Difficulty**: Medium

**Description**:

<p>There are <code>n</code> soldiers standing in a line. Each soldier is assigned a <strong>unique</strong> <code>rating</code> value.</p>

<p>You have to form a team of 3 soldiers amongst them under the following rules:</p>

<ul>
	<li>Choose 3 soldiers with index (<code>i</code>, <code>j</code>, <code>k</code>) with rating (<code>rating[i]</code>, <code>rating[j]</code>, <code>rating[k]</code>).</li>
	<li>A team is valid if: (<code>rating[i] &lt; rating[j] &lt; rating[k]</code>) or (<code>rating[i] &gt; rating[j] &gt; rating[k]</code>) where (<code>0 &lt;= i &lt; j &lt; k &lt; n</code>).</li>
</ul>

<p>Return the number of teams you can form given the conditions. (soldiers can be part of multiple teams).</p>

<p>&nbsp;</p>
<p><strong>Example 1:</strong></p>

<pre>
<strong>Input:</strong> rating = [2,5,3,4,1]
<strong>Output:</strong> 3
<strong>Explanation:</strong> We can form three teams given the conditions. (2,3,4), (5,4,1), (5,3,1). 
</pre>

<p><strong>Example 2:</strong></p>

<pre>
<strong>Input:</strong> rating = [2,1,3]
<strong>Output:</strong> 0
<strong>Explanation:</strong> We can&#39;t form any team given the conditions.
</pre>

<p><strong>Example 3:</strong></p>

<pre>
<strong>Input:</strong> rating = [1,2,3,4]
<strong>Output:</strong> 4
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>n == rating.length</code></li>
	<li><code>3 &lt;= n &lt;= 1000</code></li>
	<li><code>1 &lt;= rating[i] &lt;= 10<sup>5</sup></code></li>
	<li>All the integers in <code>rating</code> are <strong>unique</strong>.</li>
</ul>


## Thoughts

As usual, I didn't take a step back to think about the most optimal solution before attempting to solve this problem. The initial approach I took was the usual DP counting problem where the number of ways to form a team of 2 is add the number of ways soldier `i` can join a team of 1 and the number of ways to form a team of 3 is to add the number of ways soldier `i` can join a team of 2.

Initial solution (O(N^2)):
```cpp
class Solution {
public:
    int helper(const vector<int>& rating) {
        const auto n = rating.size();
        vector<int> dp(n,1);
        for (size_t soldier = 1; soldier < 3; soldier++) {
            vector<int> next_dp(n, 0);
            for (size_t j = 0; j < n; j++) {
                for (size_t i = 0; i < j; i++) {
                    if (rating[i] >= rating[j]) continue;
                    next_dp[j] += dp[i];
                }
            }
            dp = move(next_dp);
        }
        int ret = 0;
        for (const auto count : dp) {
            ret += count;
        }
        return ret;
    }
    int numTeams(vector<int>& rating) {
       int forward = helper(rating);
        reverse(rating.begin(), rating.end());
        int backward = helper(rating);
        return forward + backward;
    }
};
```

## Solution

**General Idea**

*Intuition*: Treat each solider as the middle `j` solider and find the number of possible of `i` (left) and number of possible `k` (right).

To find the number of possible `i` and `k`:
* Two multiset (sorted)
    * One multiset represents the soldiers to the left of `j` 
    * One represents the soldiers to the right of `j`.
* Finding the number of possible `i` is the number of elements in the left multiset that are strictly less than `rating[j]`
    * This can be easily achieved by using `lower_bound` (the first element that `>= rating[j]`) and moving it back by `1`
    * Use `std::distance` to find the distance between the start and the new iterator
* Finding the number of possible `j` is the number of elements in the right multiset that are strictly greater than `rating[j]`
    * This can be easily achieved by using `upper_bound` (the first element that `> rating[j]`)
    * Use `std::distance` to find the distance between the new iterator and the end
* Complexity: O(log(N))

Multiplying the number of possible `i` and `k` will give the total groups with solider `j` in the middle.

### Implementation

```cpp
class Solution {
public:
    int solve(const vector<int>& ratings) {
        const auto n = ratings.size();
        multiset<int> left;
        multiset<int> right;
        for (const auto rating : ratings) {
           right.insert(rating);
        }
        int ret = 0;
        for (size_t i = 0; i < n; i++) {
            auto rating = ratings[i];
            right.erase(right.find(rating));
            
            auto l_it = lower_bound(left.begin(), left.end(), rating);
            if (l_it !=  left.begin()) {
                // the largest left that is strictly less than rating
                l_it--;
                auto left_count = distance(left.begin(), l_it) +1;
                // the smallest right that is strictly more than rating
                auto r_it = upper_bound(right.begin(), right.end(), rating);
                auto right_count = distance(r_it, right.end());
                ret += (left_count*right_count);
            }
            left.insert(rating);
        }
        return ret;
    }
    int numTeams(vector<int>& rating) {
        int forward = solve(rating);
        reverse(rating.begin(), rating.end());
        int backward = solve(rating);
        return forward + backward;
        
    }
};
```
