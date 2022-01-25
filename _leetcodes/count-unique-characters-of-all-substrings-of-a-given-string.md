---
title: "828: Count Unique Characters of All Substrings of a Given String"
excerpt: Count Unique Characters of All Substrings of a Given String
problem_id: 828
categories:
  - Leet Code
tags:
  - String
---

## Problem 

**Problem ID**: [828](https://leetcode.com/problems/count-unique-characters-of-all-substrings-of-a-given-string/)

**Title**: Count Unique Characters of All Substrings of a Given String

**Difficulty**: Hard 

**Description**:

Let's define a function countUniqueChars(s) that returns the number of unique characters on s.

For example if s = "LEETCODE" then "L", "T", "C", "O", "D" are the unique characters since they appear only once in s, therefore countUniqueChars(s) = 5.
Given a string s, return the sum of countUniqueChars(t) where t is a substring of s.

Notice that some substrings can be repeated so in this case you have to count the repeated ones too.

## Solution

**General Idea**

The key idea to solving this problem is identify the number of substrings a character contribute. A character will only contribute to the substring if all other characters in the substring. 

### Implementation

```cpp
class Solution {
public:
    unordered_map<char, set<int>> m_c_indexes;
    int uniqueLetterString(string s) {
        for (int i = 0; i < s.size(); i++) {
            m_c_indexes[s[i]].insert(i);
        }
        
        int ret = 0;
        
        for (int i = 0; i < s.size(); i++) {
            char c = s[i];
            auto it = m_c_indexes[c].find(i);
            int left_i = 0;
            int right_i = s.size() -1;
            if (it != m_c_indexes[c].begin()) {
                it--;
                left_i = *it + 1;
                it++;
            }
            it++;
            if (it != m_c_indexes[c].end() ) {
                right_i = *it - 1; 
            }
            it--;
            ret += (i - left_i + 1)*(right_i - i + 1);
        }
        return ret;
    }
};
```
