---
title: "1143: Longest Common Subsequence"
excerpt: Finding longest common subsequence between two strings.
problem_id: 1143
categories:
  - Leet Code
tags:
  - Subsequence
  - DP
---

## Problem

**Problem ID**: [1143](https://leetcode.com/problems/longest-common-subsequence/)

**Title**: Longest Common Subsequence

**Description**:
Given two strings, find the length of the longest common subsequence.

## Thoughts

The first thing that came to my mind after reading this question is DP as it
involved comparing 2 different strings. However, I was having trouble with
deriving the DP state transition as it involved.

My initial approach was to store a 4D DP where `DP[i_l][i_r][j_l][j_r]` will
store the longest length for `text1[i_l][i_r]` and `text[j_l][j_r]` but this
would result in TLE as `text1.length < 1000` and the total run time would be
`O(n^4) = 1e12`. I soon realise that we should greedily add characters to the
subsequence and there is no need to store the left bound in the DP which would
reduce the complexity to `O(n^2)`.

## Solution

### Explanation

**General Idea**: Lets say you are currently at index `i` and `j` of `text1` and `text2`. There
are three scenario you should consider to calculate the longest subsequence up till `i` and `j`.

1. What is the longest subsequence up till `i-1` and `j`. The similarity of
characters `text1[i]` and `text2[j]` will not contribute to longest subsequence
as `text2[j]` has already been used.
2. What is the longest subsequence up till `i` and `j-1`. The similarity of
characters `text1[i]` and `text2[j]` will not contribute to longest subsequence
as `text1[i]` has already been used.
3. What is the longest subsequence up till `i-1` and `j-1`. The similarity of
`text1[i]` and `text2[j]` can contribute to the longest subsequence. 

Using this idea we can formulate the following DP transition.

**DP**

`DP[i][j]`: stores the longest subsequence for the first `i` characters of `text1`
and first `j` characters of `text2`. Even though I prefer to use index of characters
instead of length for DP parameters, I use length for this problem to allow us to not
check for index in range.

**DP transition**:
`DP[i][j] = max(DP[i-1][j-1], DP[i][j-1], DP[i-1][j-1] + text1[i-1] == text2[j-1] ? 1 : 0)`


**Implementation**

```cpp
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int n = text1.size();
        int m = text2.size();
        vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                char c1 = text1[i - 1];
                char c2 = text2[j - 1];
                int exclude_curr = max(dp[i-1][j], dp[i][j-1]);
                int curr_take = dp[i-1][j-1];
                if (c1 == c2) curr_take++;
                dp[i][j] = max(curr_take, exclude_curr);
            }
        }
        return dp[n][m];
    }
};
```

