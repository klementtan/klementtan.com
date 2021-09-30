---
title: "647: Palindromic Substrings"
excerpt: Palindromic Substrings
problem_id: 647 
categories:
  - Leet Code
tags:
  - DP
---

## Problem

**Problem ID**: [647](https://leetcode.com/problems/palindromic-substrings/)

**Title**: Palindromic Substrings

**Difficulty**: Medium

**Description**:
Given a string s, return the number of palindromic substrings in it.

A string is a palindrome when it reads the same backward as forward.

A substring is a contiguous sequence of characters within the string.

## Thoughts

I initially went with the solution of maintaining a forward and backward deque. For each
index we will treat it as the head and keep traversing the characters and check if it is palindrome
but checking if both deque are equal. However, I failed to realise that comparing deque
is a O(N) operation and my solution would result in O(N^3) time complexity.

**Take aways**:

* Comparing equality of data structure is O(N) operation
* A string is a palindrome if and only if the inner substring is a palindrome
* When dealing with a palindrome, a string of different parity (even/odd) in length
should be dealt differently.

## Solution

If a string is a palindrome, then its inner string(exclude start and end) must be a palindrome as well.
A tricky edge case to take note is that the base case is when the string is of size 2.

### Implementation

```cpp
class Solution {
public:
    int countSubstrings(string s) {
        int ans = 0;
        int n = s.size();
        vector<vector<bool>> dp(n, vector<bool>(n+1, false));
        for (int i = 0; i < n ;i++) {

            dp[i][1] = true;
            ans++;
        }
        for (int i = 0; i < n-1; i++) {
            if (s[i] == s[i+1]) {
                ans++;
                dp[i][2] = true;
            }

        }
        for (int length = 3; length <= n; length++) {
            for (int start = 0; start < n; start++) {
                if (start + length > n) continue;
                int end = start + length -1;
                int inner_start = start+1;
                int inner_length = length -2;
                if (s[start] != s[end]) continue;
                if (!dp[inner_start][inner_length]) continue;
                ans++;
                dp[start][length] = true;
            }
        }
        return ans;
    }
};
```
