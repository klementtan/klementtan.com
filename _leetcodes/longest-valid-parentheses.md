---
title: "32: Longest Valid Parentheses"
excerpt: Longest Valid Parentheses
problem_id: 32 
categories:
  - Leet Code
tags:
  - Leet Code
---

## Problem

**Problem ID**: [32](https://leetcode.com/problems/longest-valid-parentheses/)

**Title**: Longest Valid Parentheses

**Difficulty**: Hard

**Description**:

<p>Given a string containing just the characters <code>&#39;(&#39;</code> and <code>&#39;)&#39;</code>, find the length of the longest valid (well-formed) parentheses substring.</p>

<p>&nbsp;</p>
<p><strong>Example 1:</strong></p>

<pre>
<strong>Input:</strong> s = &quot;(()&quot;
<strong>Output:</strong> 2
<strong>Explanation:</strong> The longest valid parentheses substring is &quot;()&quot;.
</pre>

<p><strong>Example 2:</strong></p>

<pre>
<strong>Input:</strong> s = &quot;)()())&quot;
<strong>Output:</strong> 4
<strong>Explanation:</strong> The longest valid parentheses substring is &quot;()()&quot;.
</pre>

<p><strong>Example 3:</strong></p>

<pre>
<strong>Input:</strong> s = &quot;&quot;
<strong>Output:</strong> 0
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>0 &lt;= s.length &lt;= 3 * 10<sup>4</sup></code></li>
	<li><code>s[i]</code> is <code>&#39;(&#39;</code>, or <code>&#39;)&#39;</code>.</li>
</ul>


## Thoughts

This is hard problem that requires strong observation skills. I re-did this problem
a couple times but always have trouble realising the key observations.

## Solution | DP

**General Idea**

The longest parentheses can either consist of a single component parentheses (cannot be split into two
component of valid parentheses) or two component parentheses. Thus, the longest single
valid parentheses is trying to get the longest single component parentheses ending at `i`
and combining it with any adjacent single component parentheses.

### Implementation

DP State:
* `dp[i]`: represents the length of longest valid parentheses ending at `s[i]`
* All entries initialize to `0`

DP transition:
* Longest single component:
  * `dp[i] += dp[i-1] + 2` if `s[i] == ')'` and `s[i-dp[i-1] -1] == '('`
    * Try to wrap the valid parentheses ending at `i-1` with `(` and `)` (at `i`)
* Concat adjacent valid parentheses
  * `dp[i] += (dp[i - dp[i-1] - 2])`
    * Try to concat the valid parentheses with the newly found longest single component

```cpp
class Solution {
public:
    int longestValidParentheses(string s) {
        auto n = s.size();
        
        // dp[i] = longest paran ending at char i
        vector<size_t> dp(n, 0);
        auto print = [&] { for (auto num : dp) cout << num << " "; };
        size_t ret = 0;
        for (size_t r = 0; r < n; r++) {
            size_t l = r-1;
            if (r > 0) l = r - dp[r-1] - 1;
            if (l >= n) continue;
            if (!(s[l] == '(' && s[r] == ')')) continue;
            dp[r] = (r - l + 1);
            
            if (l > 0) dp[r] += dp[l-1];
            ret = max(ret, dp[r]);
        }
        print();
        return ret;
    }
};
```

## Solution | Stack

**General Idea**

Observation:
Even though the entire string might not be a valid parentheses, performing the normal
parentheses matching algorithm will match any valid parentheses substring.

### Implementation

Algorithm:
1. Perform the normal parentheses matching algorithm with a stack
2. At the end of each iteration, the top element in the stack represents an unmatched
parentheses
  * The substring between the top element of stack (non-inclusive) and the current element
  (inclusive) is a valid parentheses
  * We will just need to max on this substring length to get the answer.

```cpp
class Solution {
public:
    int longestValidParentheses(string s) {
        auto n = s.size();
        
        // dp[i] = longest paran ending at char i
        vector<size_t> dp(n, 0);
        auto print = [&] { for (auto num : dp) cout << num << " "; };
        size_t ret = 0;
        for (size_t r = 0; r < n; r++) {
            size_t l = r-1;
            if (r > 0) l = r - dp[r-1] - 1;
            if (l >= n) continue;
            if (!(s[l] == '(' && s[r] == ')')) continue;
            dp[r] = (r - l + 1);
            
            if (l > 0) dp[r] += dp[l-1];
            ret = max(ret, dp[r]);
        }
        print();
        return ret;
    }
};
```

