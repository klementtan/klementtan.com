---
title: "37: Sudoku Solver"
excerpt: Sudoku Solver
problem_id: 37 
categories:
  - Leet Code
tags:
  - Backtracking 
---

## Problem

**Problem ID**: [37](https://leetcode.com/problems/sudoku-solver/)

**Title**: Sudoku Solver

**Description**: Write a program to solve a Sudoku puzzle by filling the empty cells.

A sudoku solution must satisfy all of the following rules:

Each of the digits 1-9 must occur exactly once in each row.
Each of the digits 1-9 must occur exactly once in each column.
Each of the digits 1-9 must occur exactly once in each of the 9 3x3 sub-boxes of the grid.
The '.' character indicates empty cells.


## Thoughts

This is your typical dfs + backtracking question. It could be further improved by
using [AC-3](https://en.wikipedia.org/wiki/AC-3_algorithm) which is typically taught
in Artificial Intelligence classes.

## Solution

### Explanation

Utilise call stack to perform backtracking. If the current assignment is invalid, remove the
current grid from the set of values in the row, col and block.


**Implementation**

```cpp
class Solution {
public:
    int get_block_id(int i, int j) {
        int block_row = i/3;
        int block_col = j/3;
        return block_row*3 + block_col;
    }
    pair<int,int> next_grid(int i, int j) {
        if (j != 8) return {i, j+1};
        
        return {i+1, 0};
    }
    
    bool dfs_solve(int i, int j, vector<vector<char>>& board,  vector<unordered_set<int>>& rows_set, vector<unordered_set<int>>& cols_set, vector<unordered_set<int>>& blocks_set) {
        const auto& [v_i, v_j] = next_grid(i,j);
        if (i >= 9 || j >= 9) return true;
        if (board[i][j] != '.') {
            return dfs_solve(v_i, v_j, board, rows_set, cols_set, blocks_set);
        }
        int block_id = get_block_id(i,j);
        for (int num = 1; num <= 9; num++) {
            if (rows_set[i].count(num)) continue;
            if (cols_set[j].count(num)) continue;
            if (blocks_set[block_id].count(num)) continue;
            board[i][j] = num+'0';
            rows_set[i].insert(num);
            cols_set[j].insert(num);
            blocks_set[block_id].insert(num);
            if (i == 8 && j == 8) return true;
            if (dfs_solve(v_i, v_j, board, rows_set, cols_set, blocks_set)) return true;
            board[i][j] = '.';
            rows_set[i].erase( rows_set[i].find(num));
            cols_set[j].erase(cols_set[j].find(num));
            blocks_set[block_id].erase(blocks_set[block_id].find(num));
        }    
        return false;
    }
    
    void solveSudoku(vector<vector<char>>& board) {
        vector<unordered_set<int>> rows_set(9);
        vector<unordered_set<int>> cols_set(9);
        vector<unordered_set<int>> blocks_set(9);
        
        for (int i = 0; i < 9; i++) {
            for (int j = 0; j < 9; j++) {
                rows_set[i].insert(board[i][j] - '0');
                cols_set[j].insert(board[i][j] - '0');
                blocks_set[get_block_id(i,j)].insert(board[i][j] - '0');
            }
        }
        dfs_solve(0,0,board,rows_set,cols_set,blocks_set);
    }
};
```

