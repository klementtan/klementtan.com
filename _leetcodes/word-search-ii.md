---
title: "212: Word Search II"
excerpt: Word Search II
problem_id: 212
categories:
  - Leet Code
tags:
  - Trie
  - Backtracking
---

## Problem

**Problem ID**: [212](https://leetcode.com/problems/word-search-ii/)

**Title**: Word Search II

**Difficulty**: Hard 

**Description**:
Given an m x n board of characters and a list of strings words, return all words on the board.

Each word must be constructed from letters of sequentially adjacent cells, where adjacent cells are horizontally or vertically neighboring. The same letter cell may not be used more than once in a word.

## Thoughts

This is another one of those exponential time problem that requires the right implementation
to not get TLE.

To find all paths (each path do not visit the same node twice) we could actually utilise
the call stack to maintain a single visited DS instead of having to store a visited set
for every path. This technique is very useful and could be used in many different type
  or problems.

## Solution

Create a Trie that stores all the words to match. This would prevent unnecessary time spent
building a string if we used a set of string approach.

Perform dfs for each grid, this would allow us to temporary change the visited grid to `#`
before visiting its neighbors and changing it back to the original grid value after
visiting its neighbors. 

DFS vs BFS: we use DFS over BFS as it would ensure that at any point in time there
would only be one path that is being processed and we can use the grid to maintain 
the visited nodes instead of having a visited set for each path.

### Implementation

```cpp
class TrieNode {
private: 
    unordered_map<char, unique_ptr<TrieNode>> children;
    bool is_terminal;
    string s;
public:
    TrieNode(string s) : is_terminal(false) , children(), s(s) {}
    bool has_child(char c) {
        return children.count(c);
    }
    
    TrieNode* get_child(char c) {
        if (has_child(c)) {
            return children[c].get();
        } else {
            return nullptr;
        }
    }
    
    TrieNode* get_or_create_child(char c) {
        if (!has_child(c)) {
            children[c] = make_unique<TrieNode>(s + c);
        }
        return get_child(c);
    }
    void set_terminal() {
        is_terminal = true;
    }
    bool terminal() {
        return is_terminal;
    }
    string get() {
        return s;
    }
};

class Solution {
private:
    vector<vector<int>> directions = {
        {0,1},
        {0,-1},
        {1,0},
        {-1,0}
    };
public:
    void populate_trie(TrieNode* root, const vector<string>& words) {
        for (const auto& word : words) {
            TrieNode* node = root;
            for (const char c : word) {
                node = node->get_or_create_child(c);
            }
            node->set_terminal();
        }
    }
    void dfs(vector<vector<char>>& board, TrieNode* node, int i, int j, unordered_set<string>& ret_s) {
        if (!node->has_child(board[i][j])) return;
        if (board[i][j] == '#') return;
        node = node->get_child(board[i][j]);
        if(node->terminal()) ret_s.insert(node->get());
        char prev_c = board[i][j];
        board[i][j] = '#';
        for (const auto& direction : directions) {
            int v_i = i+ direction[0];
            int v_j = j+ direction[1];
            if (v_i < 0 || v_j < 0 || v_i >= board.size() || v_j >= board.front().size()) continue;
            if (board[v_i][v_j] == '#') continue;
            dfs(board, node, v_i, v_j, ret_s);
        }
        board[i][j] = prev_c;
    }
    vector<string> findWords(vector<vector<char>>& board, vector<string>& words) {
        unique_ptr<TrieNode> root = make_unique<TrieNode>("");
        populate_trie(root.get(), words);
        unordered_set<string> ret_s;
        for (int i = 0; i < board.size(); i++) {
            for (int j = 0; j < board.front().size(); j++) {
                if (!root.get()->has_child(board[i][j])) continue;
                dfs(board,root.get(), i, j, ret_s);
            }
        }
        
        return vector<string>(ret_s.begin(), ret_s.end());
    }
};
```
