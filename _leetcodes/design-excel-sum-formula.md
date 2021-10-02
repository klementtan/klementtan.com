---
title: "631: Design Excel Sum Formula"
excerpt: Design Excel Sum Formula
problem_id: 631
categories:
  - Leet Code
tags:
  - Design
---

## Problem

**Problem ID**: [631](https://leetcode.com/problems/design-excel-sum-formula/)

**Title**: Design Excel Sum Formula

**Difficulty**: Hard

**Description**:
Design the basic function of Excel and implement the function of the sum formula.

## Thoughts

At first glance, this question seem very complicated but once you can identify that the transitive property
of `sum` means that we would need to perform some sort of search to traverse all connected neighbours.
The tricky part of this question is in the implementation.

## Solution

The general idea to this problem is to maintain 2 separate board, one for keep tracking of the values that was set
and another one for keeping track the grids that it is a sum of.

Data Structures:
* `vector<vector<int>> m_sheets`: Represents an excel spreadsheet where `m_sheets[i][j]` stores the value set at
`row = i+1` and `column = 'A' + j`
  * When `m_sheets[i][j] = NullVal` this means that the grid's value is a sum of other grids and should be looked up
    on `m_sums` instead.
* `vector<vector<vector<pair<int,int>>>> m_sums`: Represents the `sum` data. `m_sums[i][j]` is a list of other grids
`[<i1,j1>, <i2,j2>,...]` that it is a sum of.

Handling:
* `get`: We will set the provided grid as our source and perform bfs. We will continue searching till `m_sheets[i][j] != NullVal`.
* `sum`: We will use the provided data to updated `m_sums`. For **range of cells**, we will iterate deconstruct the range into individual cells.
  * We will set the grid on `m_sheets` to `NullVal` to indicate that the grid does not have any values.
* `set`: We will update the `m_sheets` and clear the data in the corresponding `m_sums` as it would override the previous `sum` formula.

### Implementation

```cpp
class Excel {
private:
    // [i][j] = val
    vector<vector<int>> m_sheets;
    // [i][j] = [<i1,j1>, ...]
    vector<vector<vector<pair<int,int>>>> m_sums;
    int m_n;
    int m_m;
    int NullVal = INT_MAX;
    
    inline const pair<int,int> ExtractGrid(const string& grid) {
        assert(2 <= grid.length() && grid.length() <= 3);
        
        int row = 0;
        for (int row_i = 1; row_i < grid.size(); row_i++) {
            assert(isdigit(grid[row_i]));
            row *=10;
            row += (grid[row_i] - '0');
        }
        return {GetI(row),GetJ(grid[0])};
    }
    
    inline const vector<pair<int,int>> ExtractRange(string range) {
        assert(5 <= range.length() && range.length() <= 7);
        
        size_t colon_pos = range.find(':');
        string start_grid_s = range.substr(0, colon_pos);
        range.erase(0, colon_pos +1);
        string end_grid_s = move(range);
        const auto [s_i, s_j] = ExtractGrid(start_grid_s);
        const auto [t_i, t_j] = ExtractGrid(end_grid_s);
        vector<pair<int,int>> ret;
        for (int i = s_i; i <= t_i; i++) {
            for (int j = s_j; j <= t_j; j++) {
                ret.emplace_back(i,j);
            }
        }
        return ret;
    }
    
    constexpr inline int GetI(int row) {
        return row - 1;
    }
    constexpr inline int GetJ(char column) {
        return column - 'A';
    }
    
    constexpr inline bool IsRange(const string& grid) {
        return grid.find(':') != std::string::npos;
    }
        
    inline vector<pair<int,int>> GetSumOf(const vector<string>& numbers) {
        vector<pair<int,int>> v_grids;
        for (const string& num : numbers) {
            if (IsRange(num)) {
                vector<pair<int,int>> curr_v_grids = ExtractRange(num);
                v_grids.insert(v_grids.end(), curr_v_grids.begin(), curr_v_grids.end());
            } else {
                v_grids.push_back(ExtractGrid(num));
            }
        }
        return v_grids;
    }
    
public:
    Excel(int height, char width) {
        int m_n = height;
        int m_m = width - 'A'  +1;
        m_sheets = vector<vector<int>>(m_n, vector<int>(m_m, 0));
        m_sums = vector<vector<vector<pair<int,int>>>>(m_n, vector<vector<pair<int,int>>>(m_m));
    }
    
    void set(int row, char column, int val) {
        int i = GetI(row);
        int j = GetJ(column);
        m_sums[i][j].clear();
        m_sheets[i][j] = val;
    }
    
    int Get(int s_i, int s_j) {
        int sum = 0;
        queue<pair<int,int>> q;
        q.push({s_i,s_j});
        while(!q.empty()) {
            auto [i,j] = q.front();
            q.pop();
            if (m_sheets[i][j] == NullVal) {
                for (const auto [v_i, v_j] : m_sums[i][j]) {
                    q.push({v_i, v_j});
                }
            } else {
                sum += m_sheets[i][j];
            }
        } 
        return sum;
    }
    
    int get(int row, char column) {
        return Get(GetI(row), GetJ(column));   
    }

    int sum(int row, char column, vector<string> numbers) {
        vector<pair<int,int>> v_grids = GetSumOf(numbers);
        int i = GetI(row);
        int j = GetJ(column);
        m_sheets[i][j] = NullVal;
        m_sums[i][j] = v_grids;
        return get(row, column);
    }
};
```
