---
title: "1196: Filling Bookcase Shelves"
excerpt: Filling Bookcase Shelves
problem_id: 1196 
categories:
  - Leet Code
tags:
  - Leet Code
---

## Problem

**Problem ID**: [1196](https://leetcode.com/problems/filling-bookcase-shelves/)

**Title**: Filling Bookcase Shelves

**Difficulty**: Medium

**Description**:

<p>You are given an array <code>books</code> where <code>books[i] = [thickness<sub>i</sub>, height<sub>i</sub>]</code> indicates the thickness and height of the <code>i<sup>th</sup></code> book. You are also given an integer <code>shelfWidth</code>.</p>

<p>We want to place these books in order onto bookcase shelves that have a total width <code>shelfWidth</code>.</p>

<p>We choose some of the books to place on this shelf such that the sum of their thickness is less than or equal to <code>shelfWidth</code>, then build another level of the shelf of the bookcase so that the total height of the bookcase has increased by the maximum height of the books we just put down. We repeat this process until there are no more books to place.</p>

<p>Note that at each step of the above process, the order of the books we place is the same order as the given sequence of books.</p>

<ul>
	<li>For example, if we have an ordered list of <code>5</code> books, we might place the first and second book onto the first shelf, the third book on the second shelf, and the fourth and fifth book on the last shelf.</li>
</ul>

<p>Return <em>the minimum possible height that the total bookshelf can be after placing shelves in this manner</em>.</p>

<p>&nbsp;</p>
<p><strong>Example 1:</strong></p>
<img alt="" src="https://assets.leetcode.com/uploads/2019/06/24/shelves.png" style="height: 500px; width: 337px;" />
<pre>
<strong>Input:</strong> books = [[1,1],[2,3],[2,3],[1,1],[1,1],[1,1],[1,2]], shelf_width = 4
<strong>Output:</strong> 6
<strong>Explanation:</strong>
The sum of the heights of the 3 shelves is 1 + 3 + 2 = 6.
Notice that book number 2 does not have to be on the first shelf.
</pre>

<p><strong>Example 2:</strong></p>

<pre>
<strong>Input:</strong> books = [[1,3],[2,4],[3,2]], shelfWidth = 6
<strong>Output:</strong> 4
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= books.length &lt;= 1000</code></li>
	<li><code>1 &lt;= thickness<sub>i</sub> &lt;= shelfWidth &lt;= 1000</code></li>
	<li><code>1 &lt;= height<sub>i</sub> &lt;= 1000</code></li>
</ul>


## Thoughts

None

## Solution

**General Idea**

Starting from the last book, we  find the current minimum possible height with
the current book at the start of the row.

### Implementation

DP State:
* `dp[i]`: represents the minimum height of books `[i,...,n-1]` with book `i`
at the start of the row

DP Transition:
* For all `j >= i` and sum of `books[i][0] + ... books[j][0] <= shelfWidth`,
`dp[i] = min( dp[i], max(books[i][1], ..., books[j][1]) + dp[j+1])`
* Represents trying to add as many books to the row with book `i` at the start
of the row. For each new book added to the row you can find the minimum height by
adding the minimum of the books in the current row and the minimum height of the shelf
with the next book at a new row.

```cpp
class Solution {
public:
    int minHeightShelves(vector<vector<int>>& books, int shelfWidth) {
        int n = books.size();
        vector<int> dp(n, 1e9);
        for (int i = n-1; i >= 0; i--) {
            int w = 0;
            int h = 0;
            for (int j = i ; j < n; j++) {
                w += books[j][0];
                h = max(h, books[j][1]);
                if (w > shelfWidth) break;
                int additional = 0;
                if (j + 1 < n) additional += dp[j+1];
                dp[i] = min(dp[i], h + additional);
            }
        }
        return dp[0];
    }
};
```
