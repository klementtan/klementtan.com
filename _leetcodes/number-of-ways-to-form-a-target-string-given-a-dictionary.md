---
title: "1744: Number of Ways to Form a Target String Given a Dictionary"
excerpt: Number of Ways to Form a Target String Given a Dictionary
problem_id: 1744 
categories:
  - Leet Code
tags:
  - Leet Code
---

## Problem

**Problem ID**: [1744](https://leetcode.com/problems/number-of-ways-to-form-a-target-string-given-a-dictionary/)

**Title**: Number of Ways to Form a Target String Given a Dictionary

**Difficulty**: Hard

**Description**:

<p>You are given a list of strings of the <strong>same length</strong> <code>words</code> and a string <code>target</code>.</p>

<p>Your task is to form <code>target</code> using the given <code>words</code> under the following rules:</p>

<ul>
	<li><code>target</code> should be formed from left to right.</li>
	<li>To form the <code>i<sup>th</sup></code> character (<strong>0-indexed</strong>) of <code>target</code>, you can choose the <code>k<sup>th</sup></code> character of the <code>j<sup>th</sup></code> string in <code>words</code> if <code>target[i] = words[j][k]</code>.</li>
	<li>Once you use the <code>k<sup>th</sup></code> character of the <code>j<sup>th</sup></code> string of <code>words</code>, you <strong>can no longer</strong> use the <code>x<sup>th</sup></code> character of any string in <code>words</code> where <code>x &lt;= k</code>. In other words, all characters to the left of or at index <code>k</code> become unusuable for every string.</li>
	<li>Repeat the process until you form the string <code>target</code>.</li>
</ul>

<p><strong>Notice</strong> that you can use <strong>multiple characters</strong> from the <strong>same string</strong> in <code>words</code> provided the conditions above are met.</p>

<p>Return <em>the number of ways to form <code>target</code> from <code>words</code></em>. Since the answer may be too large, return it <strong>modulo</strong> <code>10<sup>9</sup> + 7</code>.</p>

<p>&nbsp;</p>
<p><strong>Example 1:</strong></p>

<pre>
<strong>Input:</strong> words = [&quot;acca&quot;,&quot;bbbb&quot;,&quot;caca&quot;], target = &quot;aba&quot;
<strong>Output:</strong> 6
<strong>Explanation:</strong> There are 6 ways to form target.
&quot;aba&quot; -&gt; index 0 (&quot;<u>a</u>cca&quot;), index 1 (&quot;b<u>b</u>bb&quot;), index 3 (&quot;cac<u>a</u>&quot;)
&quot;aba&quot; -&gt; index 0 (&quot;<u>a</u>cca&quot;), index 2 (&quot;bb<u>b</u>b&quot;), index 3 (&quot;cac<u>a</u>&quot;)
&quot;aba&quot; -&gt; index 0 (&quot;<u>a</u>cca&quot;), index 1 (&quot;b<u>b</u>bb&quot;), index 3 (&quot;acc<u>a</u>&quot;)
&quot;aba&quot; -&gt; index 0 (&quot;<u>a</u>cca&quot;), index 2 (&quot;bb<u>b</u>b&quot;), index 3 (&quot;acc<u>a</u>&quot;)
&quot;aba&quot; -&gt; index 1 (&quot;c<u>a</u>ca&quot;), index 2 (&quot;bb<u>b</u>b&quot;), index 3 (&quot;acc<u>a</u>&quot;)
&quot;aba&quot; -&gt; index 1 (&quot;c<u>a</u>ca&quot;), index 2 (&quot;bb<u>b</u>b&quot;), index 3 (&quot;cac<u>a</u>&quot;)
</pre>

<p><strong>Example 2:</strong></p>

<pre>
<strong>Input:</strong> words = [&quot;abba&quot;,&quot;baab&quot;], target = &quot;bab&quot;
<strong>Output:</strong> 4
<strong>Explanation:</strong> There are 4 ways to form target.
&quot;bab&quot; -&gt; index 0 (&quot;<u>b</u>aab&quot;), index 1 (&quot;b<u>a</u>ab&quot;), index 2 (&quot;ab<u>b</u>a&quot;)
&quot;bab&quot; -&gt; index 0 (&quot;<u>b</u>aab&quot;), index 1 (&quot;b<u>a</u>ab&quot;), index 3 (&quot;baa<u>b</u>&quot;)
&quot;bab&quot; -&gt; index 0 (&quot;<u>b</u>aab&quot;), index 2 (&quot;ba<u>a</u>b&quot;), index 3 (&quot;baa<u>b</u>&quot;)
&quot;bab&quot; -&gt; index 1 (&quot;a<u>b</u>ba&quot;), index 2 (&quot;ba<u>a</u>b&quot;), index 3 (&quot;baa<u>b</u>&quot;)
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= words.length &lt;= 1000</code></li>
	<li><code>1 &lt;= words[i].length &lt;= 1000</code></li>
	<li>All strings in <code>words</code> have the same length.</li>
	<li><code>1 &lt;= target.length &lt;= 1000</code></li>
	<li><code>words[i]</code> and <code>target</code> contain only lowercase English letters.</li>
</ul>


## Thoughts

I found this problem very hard. Took around and hour to solve the problem. It is
quite obvious that this is a DP problem as it involves counting. However, I was
initially tricked into thinking that DP state involves the word index.

I could quickly develop the correct DP state and DP transition but I found huge
difficulty optimising it from a O(N^3) solution to a O(N^2) solution.

My solution led to flaky TLE but eventhough the time complexity is exactly the
same as the [model answer](https://leetcode.com/problems/number-of-ways-to-form-a-target-string-given-a-dictionary/discuss/917779/JavaC%2B%2BPython-Space-O(N))
solution in the discuss.


## Solution

**General Idea**

For each characters in the target, find the possible indexes it can match to in
the `words`. This simplifies the problem to finding the number of ways to
select the associated indexes for each character in the target such that the
indexes are increasing. This could solved using DP.

DP State | `dp[i][j]`: represents the total number of ways to match `target[0,...,i]` characters
with the first `words[k][0,...,j]` characters (for all `k`).

DP Transition:
* Do not use the `j`th character of `words` to match `target[i]`. 
  * Instead Use the `j<`th character of `words` to match `target[i]`
  * `dp[i][j] += dp[i][j-1]`
* Use the `j`th character of `words` to match `target[i]`
  * For each instance of `j`th character of `words` that matches `target[i]`, extend it
  from the previously calculated ways for the `j <`th character of `words` to match `target[0,..., i-1]`
  * `dp[i][j] += dp[i-1][j-1]*freq`

### Implementation

```cpp
class Solution {
public:
    int numWays(vector<string>& words, string target) {
        unordered_set<char> target_chars(target.begin(), target.end());
        unordered_map<char, map<int,int> > char_idxes;
        
        auto m = size_t{0};
        
        for (const auto& word : words) {
            m = max(m, word.size());
            for (size_t i = 0; i < word.size(); i++) {
                if (target_chars.count(word[i]) == 0) continue;
                char_idxes[word[i]][i] += 1;
            }
        }
        
        auto n = target.size();
        vector<vector<int>> dp(n, vector<int>(m));
        auto print = [&dp, &char_idxes]() {
            cout << "Char Index" << endl;
            for (auto& [c, m] : char_idxes) {
                cout << c << endl;
                for(auto& [i, frq] : m) {
                    cout << "  " << i << "*" << frq << endl;
                }
            }
            cout << "DP" << endl;
            for (const auto& row : dp) {
                for (const auto i : row) cout << i << " ";
                cout << endl;
            }
        };
        for (auto [j, freq] : char_idxes[target[0]]) dp[0][j] += freq;
        for (int j = 1; j < m; j++) dp[0][j] += dp[0][j-1];
        m = char_idxes[target.back()].rbegin()->first;
        
        for (int i = 1; i < n; i++) {
            char target_c = target[i];
            for (int j = 1; j <= m; j++) {
                int freq = char_idxes[target_c].count(j) ? char_idxes[target_c][j] : 0;
                if (freq > 0) dp[i][j] = (dp[i][j] + dp[i-1][j-1]) % static_cast<int>(1e9+7);
                int tmp = 0;
                for (int k = 0; k < freq; k++) {
                    tmp = (tmp + dp[i][j])%static_cast<int>(1e9+7);
                }
                dp[i][j] = tmp;
                dp[i][j] = (dp[i][j] + dp[i][j-1]) % static_cast<int>(1e9+7); // dont select the current char
            }
        }
        return dp[n-1][m];
    }
};
```
