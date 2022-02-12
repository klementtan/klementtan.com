---
title: "269: Alien Dictionary"
excerpt: Alien Dictionary
problem_id: 269
categories:
  - Leet Code
tags:
  - Graph
---

## Problem

**Problem ID**: [269](https://leetcode.com/problems/alien-dictionary/)

**Title**: Alien Dictionary

**Difficulty**: Hard 

**Description**:

There is a new alien language that uses the English alphabet. However, the order among the letters is unknown to you.

You are given a list of strings words from the alien language's dictionary, where the strings in words are sorted lexicographically by the rules of this new language.

Return a string of the unique letters in the new alien language sorted in lexicographically increasing order by the new language's rules. If there is no solution, return "". If there are multiple solutions, return any of them.

A string s is lexicographically smaller than a string t if at the first letter where they differ, the letter in s comes before the letter in t in the alien language. If the first min(s.length, t.length) letters are the same, then s is smaller if and only if s.length < t.length.

## Thoughts

This is quite a straight forward graph problem. The challenging part for me is trying to solve it optimally (combining topo sort with cycle check).

The provided solution uses the idea that if there is a cycle, you can traverse the reverse graph starting from indegree 0. If not all nodes are visited then there exists a cycle.

## Solution

### Implementation

```cpp
class Solution {
public:
    
    bool process(const string& smaller, const string& bigger, unordered_map<char, unordered_set<char>>& g) {
        bool has_diff = false;
        for (int i = 0; i < min(smaller.size(), bigger.size()); i++) {
            if (smaller[i] == bigger[i]) {
                continue;
            } else {
                g[smaller[i]].insert(bigger[i]);
                has_diff = true;
                break;
            }
        }    
        if (!has_diff && smaller.size() > bigger.size()) return false;
        return true;
    }
    bool topo_helper(const unordered_map<char, unordered_set<char>>& g, char u, unordered_set<char>& visited, stack<char>& s, unordered_set<char>& call_stack) {
        if (visited.count(u)) return true;
        visited.insert(u);
        call_stack.insert(u);
        if (g.find(u) != g.end()) {
            for (auto v : g.find(u)->second) {
                if (call_stack.count(v)) return false;
                if (!topo_helper(g, v, visited, s, call_stack)) return false;
            }            
        }
        call_stack.erase(u);
        s.push(u);
        return true;
    }
    string topo(const unordered_map<char, unordered_set<char>>& g) {
        unordered_set<char> visited;
        stack<char> s;
        unordered_set<char> call_stack;
        for (const auto& [u, _] : g) {
            if (visited.count(u)) continue;
            if (!topo_helper(g, u, visited, s, call_stack)) return "";
        }
        string ret;
        while (!s.empty()) {
            char c = s.top();
            s.pop();
            ret += c;
        }
        return ret;
    }
    string check_topo(const string& topo, const unordered_map<char, unordered_set<char>>& g) {
        unordered_map<char, int> idxes;
        for (int i = 0; i < topo.size(); i++) idxes[topo[i]] = i;
        for (auto [u, vs] : g) {
            for (auto v : vs) {
                if (idxes[u] > idxes[v]) return "";      
            }
        }
        return topo;
    }
    string alienOrder(vector<string>& words) {
        // u -> v if u < v
        unordered_map<char, unordered_set<char>> g;
        for (auto word : words) for (auto c : word) g[c] = {};
        for (int i = 0; i < words.size() - 1; i++) {
            if (!process(words[i], words[i+1], g)) return "";
        }
        return topo(g);
    }
};
```
