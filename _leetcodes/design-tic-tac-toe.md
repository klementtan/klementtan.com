---
title: "348: Design Tic-Tac-Toe"
excerpt: Design Tic-Tac-Toe
problem_id: 338
categories:
  - Leet Code
tags:
  - Design
---

## Problem

**Problem ID**: [348](https://leetcode.com/problems/design-tic-tac-toe/)

**Title**: Design Tic-Tac-Toe

**Difficulty**: Medium

**Description**:
Assume the following rules are for the tic-tac-toe game on an n x n board between two players:

## Thoughts

This is quite a classic design question(similar to connect 4). 

## Solution

The main idea behind this problem
is to traverse the board in all possible lines starting from where the last move was played.

We could cleanly perform this operation by traversing north-east(-1,-1), north-east(-1,0), north-west(-1,1), east(0,1)
and the corresponding opposite direction by multiplying the direction by `-1`. We will only keep score for the number
of consecutive board occupied by the same player and stop traversing once reach a grid occupied by another player.


### Implementation

```cpp
class TicTacToe {
public:
    vector<vector<int>> board;
    TicTacToe(int n) {
        board = vector<vector<int>>(n, vector<int>(n, 0));
    }
    
    int get_winner(int row, int col) {
        static constexpr const int directions[4][2] = {
            {-1,-1}, // Top left
            {-1, 0}, // Up
            {-1,1}, // Top Right
            {0,1} // Right
        };
        static constexpr const int forward[2] = {-1, 1};
        const auto is_valid = [this](int i, int j) {
            return i >= 0 && j >= 0 && i < this->board.size() && j < this->board.size();
        };
        int player = board[row][col];
        if (player == 0) return 0;
        for (auto direction : directions) {
            int score = 1;
            for (auto is_forward : forward) {
                int curr_direction_i = direction[0]*is_forward;
                int curr_direction_j = direction[1]*is_forward;
                for (int step = 1; is_valid(row + curr_direction_i*step, col + curr_direction_j*step); step++) {
                    if (board[row + curr_direction_i*step][col + curr_direction_j*step] == player) {
                        score++;
                    } else {
                        break;
                    }
                }
            }

            if (score == board.size()) return player;
        }
        return 0;
    }
    
    int move(int row, int col, int player) {
        board[row][col] = player;
        return get_winner(row, col);
    }
};
```
