---
title: "1901: Find a Peak Element II"
excerpt: Find a Peak Element II
problem_id: 1901
categories:
  - Leet Code
tags:
  - Binary Search 
---

## Problem 

**Problem ID**: [1901](https://leetcode.com/problems/find-a-peak-element-ii/)

**Title**: Find a Peak Element II

**Difficulty**: Medium

**Description**:
A peak element in a 2D grid is an element that is strictly greater than all of its adjacent neighbors to the left, right, top, and bottom.

Given a 0-indexed m x n matrix mat where no two adjacent cells are equal, find any peak element mat[i][j] and return the length 2 array [i,j].

You may assume that the entire matrix is surrounded by an outer perimeter with the value -1 in each cell.

You must write an algorithm that runs in O(m log(n)) or O(n log(m)) time.


## Thoughts

This problem is an extension to Find a Peak Element I and you should attempt that problem before this.

My initial solution was iterate through every row of the matrix and recursively
find all the 1D peak elements for that row. If a 1D peak elements is not bigger than all 4 of the adjacent grid, then I will process the next peak element in that role. However, this solution does not provide a `O(mlog(n))` solution.

Contradiction Proof:
```
3 2 1
4 3 2
5 4 3
```

Using my initial solution, I will iterate through every single grid as the peak element is only on the bottom left.

## Solution

Instead of finding the 2D peak element we can flatten the 2D grid into 1D grid
by having each column reduce to the highest largest element on the grid. Thus,
using the initial Find a Peak Element I solution we can treat accessing the `ith` element in the 1D grid as `find_max(i)`.

As we only iterate through every column only during each iteration of the binary search, this will add an additional `*n` to the complexity

### Implementation

```cpp
class Solution {
public:
    vector<pair<int,int>> cache;
    pair<int,int> get_val(vector<vector<int>>& mat, int j) {
        cout << j << endl;
        if (cache[j] != pair<int,int>{-1,-1}) return cache[j];
        auto max_val = -1;
        auto max_i = 0;
        for (int i = 0; i < mat.size(); i++) {
            if (mat[i][j] > max_val) {
                max_i = i;
                max_val = mat[i][j];
            }
        }
        cache[j] = {max_i, max_val};
        return {max_i, max_val};
    }
    
    vector<int> findPeakGrid(vector<vector<int>>& mat) {
        int l = 0;
        int r = mat.front().size() - 1;
        cache = vector<pair<int,int>>(r + 1, {-1,-1});
        
        int row = -1;
        while (l < r) {
            int m = (l + r)/2;
            auto [row_, m_val] = get_val(mat, m);
            row = row_;

            if (m_val > get_val(mat, m+1).second) {
                r = m;
            } else {
                l = m + 1;
            }
        }
        return {get_val(mat,l).first, l};
    }
};
```
