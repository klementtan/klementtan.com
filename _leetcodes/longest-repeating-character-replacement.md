---
title: "424: Longest Repeating Character Replacement"
excerpt: Longest Repeating Character Replacement
problem_id: 424 
categories:
  - Leet Code
tags:
  - DP
---

## Problem

**Problem ID**: [424](https://leetcode.com/problems/longest-repeating-character-replacement/)

**Title**: Longest Repeating Character Replacement

**Difficulty**: Medium

**Description**:

You are given a string s and an integer k. You can choose any character of the string and change it to any other uppercase English character. You can perform this operation at most k times.

Return the length of the longest substring containing the same letter you can get after performing the above operations.

## Thoughts

I found this question extremely hard. Had to view the related topics before being able to derive
a non-ideal O(N^2) solution and had to view the discussion to derive the optimal O(N) solution.

The main idea that I missed is that you can determine determine the number of replacement performed in O(1) 
time by keeping a frequency map. Number of replacement = length of curr substring - highest frequency val.
Additionally any map containing characters can be traversed in O(1) time as there are only constant(26) number
of keys.

## Solution

Checking a valid substring with repeating characters and less than `k` replacement
can be done using: `substring_length - max_char_freq <= k`.

We can solve this question by maintain a sliding window that always ensure that
`substring_length - max_char_freq <= k`. We do so by advancing the start of the substring
whenever there is a violation.

### Implementation

```cpp
class Solution {
public:
    int characterReplacement(string s, int k) {
        unordered_map<char,int> char_freq;
        int n = s.length();
        int start = 0;
        int ans = 0;
        for (int i = 0; i < n; i++) {
            char_freq[s[i]]++;
            int max_count = char_freq[GetMax(char_freq)];
            if (i-start+1 -  max_count> k) {
                 
                char_freq[s[start]]--;
                start++;
            }
            ans = max(ans, i-start +1);
        }
        return ans;
    }
    
    char GetMax(const unordered_map<char,int>& m) {
        int max_val = 0;
        char max_c;
        for (const auto [c, val] : m) {
            if (val > max_val) {
                max_c = c;
                max_val = val;
            }
        }
        return max_c;
    }
};
```
