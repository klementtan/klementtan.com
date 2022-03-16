---
title: "1398: Number of Ways to Stay in the Same Place After Some Steps"
excerpt: Number of Ways to Stay in the Same Place After Some Steps
problem_id: 1398 
categories:
  - Leet Code
tags:
  - Leet Code
---

## Problem

**Problem ID**: [1398](https://leetcode.com/problems/number-of-ways-to-stay-in-the-same-place-after-some-steps/)

**Title**: Number of Ways to Stay in the Same Place After Some Steps

**Difficulty**: Hard

**Description**:

<p>You have a pointer at index <code>0</code> in an array of size <code>arrLen</code>. At each step, you can move 1 position to the left, 1 position to the right in the array, or stay in the same place (The pointer should not be placed outside the array at any time).</p>

<p>Given two integers <code>steps</code> and <code>arrLen</code>, return the number of ways such that your pointer still at index <code>0</code> after <strong>exactly</strong> <code>steps</code> steps. Since the answer may be too large, return it <strong>modulo</strong> <code>10<sup>9</sup> + 7</code>.</p>

<p>&nbsp;</p>
<p><strong>Example 1:</strong></p>

<pre>
<strong>Input:</strong> steps = 3, arrLen = 2
<strong>Output:</strong> 4
<strong>Explanation: </strong>There are 4 differents ways to stay at index 0 after 3 steps.
Right, Left, Stay
Stay, Right, Left
Right, Stay, Left
Stay, Stay, Stay
</pre>

<p><strong>Example 2:</strong></p>

<pre>
<strong>Input:</strong> steps = 2, arrLen = 4
<strong>Output:</strong> 2
<strong>Explanation:</strong> There are 2 differents ways to stay at index 0 after 2 steps
Right, Left
Stay, Stay
</pre>

<p><strong>Example 3:</strong></p>

<pre>
<strong>Input:</strong> steps = 4, arrLen = 2
<strong>Output:</strong> 8
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= steps &lt;= 500</code></li>
	<li><code>1 &lt;= arrLen &lt;= 10<sup>6</sup></code></li>
</ul>


## Thoughts

This problem is similar to [1155. Number of Dice Rolls With Target Sum](https://leetcode.com/problems/number-of-dice-rolls-with-target-sum/)
where the number of ways for event `i` is dependent on event `i-1`. However, after looking at the discuss
I learnt that presumably 2D DP problems that replies only on the previous state (`DP[i][j-1]`) can be
simplified to 1D by storing 2 x 1D DP instead where one of the dp acts as the previous and the other
acts as the present. At the end of iteration you will just need to swap both the DP. This will allow
the memory complexity to reduce from O(N^2) to O(N)


## Solution

**General Idea**

DP state:
* `prev_dp[i]`: represent the ways to reach position `i` at `k-1` step from position 0
* `dp[i]`: represent the ways to reach position `i` at `k` step from position 0

DP transition:
* Intuition: to reach position 0 at step `k` you will just need to enumerate over the possible kth step. 
  * If the kth step is left, then the position at step `k-1` is `i+1`. This can be easily extended
  to kth step being stay or right.
  * The total ways is to reach `i` is just the sum of the ways to reach `i-1,i,i+1` at step `k-1`
* `dp[i] = prev_dp[i-1] + prev_dp[i] + prev_dp[i+1]` at the end of the iteration `swap(dp, prev_dp)`


### Implementation

```cpp
class Solution {
public:
    int numWays(int steps, int arrLen) {
        int size = min(arrLen, steps+2);
        vector<int> prev_dp(size, 0);
        vector<int> dp(size, 0);
        prev_dp[0] = 1;
        int mod = 1e9 + 7;
        for (int k = 1; k <= steps; k++) {
            for (int i = 0; i < size; i++) {
                dp[i] = 0;
                if (i > 0) dp[i] = (dp[i] + prev_dp[i-1]) % mod;
                if (i < size - 1) dp[i] = (dp[i] + prev_dp[i+1]) % mod;
                dp[i] = (dp[i] + prev_dp[i]) % mod;
            }
            swap(dp, prev_dp);
        }
        return prev_dp[0];
    }
};
```
