---
title: "301: Remove Invalid Parentheses"
excerpt: Palindromic Substrings
problem_id: 301
categories:
  - Leet Code
tags:
  - DP
---

## Problem

**Problem ID**: [301](https://leetcode.com/problems/remove-invalid-parentheses/)

**Title**: Palindromic Substrings

**Difficulty**: Hard

**Description**:
Given a string s that contains parentheses and letters, remove the minimum number of invalid parentheses to make the input string valid.

Return all the possible results. You may return the answer in any order.

## Thoughts

This is your typical leetcode hard parentheses problem. I have previously done
a similar problem ([Longest Valid Parentheses](https://leetcode.com/problems/longest-valid-parentheses/))
and applied the same idea of using DP for the problem. To my surprise, all of the
solutions in the solution tab and discussion utilised backtracking solution.

## Solution

**DP State**:
`dp[i][j]`: Stores a list of valid parentheses from index `i` (inclusive) to `j` (inclusive).

* ie: `dp[i,j] = {"()()","(())"}`

**DP transition**
For all transitions we the values as candidates first then filter out the values with the maximum length

1. When `s[i]` and `s[j]` are valid parentheses and the inner substring (`i+1...j-1`) is also a valid
  parentheses
  * We will add all valid inner substrings with surrounding `(` and `)` to the candidates
  * `candidates` append `"(" + inner_substr + ")"`
2. When `s[i+1 ... j]`,`s[i ... j-1]`, `s[i+1, ..., j-1]` is a valid parentheses
  * We can think of this when either remove `s[i]` or `s[j]` and use the valid parentheses computed previously
  * `candidates` append all candidate from `dp[i][j]`, `dp[i][j-1]`, `dp[i+1][j]`
3. When we merge two adjacent valid parentheses (ie `(())<-merge->()`)
  * As we utilise a bottom up DP approach, it is guaranteed that if `s[i...j]` is a valid parentheses that is made
  up completely of two smaller adjacent parentheses, the longest length for the two smaller parentheses has been computed.
  * We will have an additional pointer `i<=k<j` that represents the right pointer to the left smaller parentheses.
  * `candidates` append all cross product of `dp[i][k]` and `dp[k+1][j]`
4. `dp[i][j]` is the list of all candidates with the maximum length

**Complexity**:
I believe that the complexity of the solution is still exponential as we will still need to perform a cross product
between the values at two different smaller state.

### Implementation

```cpp
class Solution {
public:
    vector<string> removeInvalidParentheses(string s) {
        int n = s.size();
        // dp[i][j]: a vector of longest valid parenthesis between i and j inclusive
        vector<vector<vector<string>>> dp(n, vector<vector<string>>(n, vector<string>(1, "")));
        for (int i = 0; i < n; i++) {
            if (s[i] != '(' && s[i] != ')') {
                std::string curr{s[i]};
                dp[i][i] = {curr};
            }
            if (i < n-1 && s[i] == '(' && s[i+1] == ')') dp[i][i+1] = {"()"};
        }
        

        for (int len = 1; len <= n;len++) {
            for (int i = 0; i + len <= n; i++) {
                int j = i + len - 1;
                unordered_set<string> candidates;
                if (s[i] == '(' && s[j] == ')') {
                    for (const string& inner_valid : dp[i+1][j-1]) {
                        string candidate = "(" + inner_valid + ")";
                        candidates.insert(candidate);
                    }
                } 
                for (int k = i; k < j; k++) {
                    for (const string& left_valid : dp[i][k]) {
                        for (const string& right_valid : dp[k+1][j]) {
                            if (left_valid.empty() || right_valid.empty()) continue;
                            candidates.insert(left_valid+right_valid);
                        }
                    }
                }
                
                vector<string>& full_inner = dp[i][j];
                candidates.insert(full_inner.begin(), full_inner.end());
                if (j-1 >= 0) candidates.insert(dp[i][j-1].begin(), dp[i][j-1].end());
                if (i+1 < n) candidates.insert(dp[i+1][j].begin(), dp[i+1][j].end());
                int max_length = 0;
                for (const string& curr_valid : candidates) max_length = max(max_length, static_cast<int>(curr_valid.size()));
                vector<string> processed_curr_valid;
                for (const string& curr_valid : candidates) {
                    if(curr_valid.size() == max_length) processed_curr_valid.push_back(curr_valid);
                }
                dp[i][j] = processed_curr_valid;
            }
        }

        return dp[0][n-1];

    }
};
```
