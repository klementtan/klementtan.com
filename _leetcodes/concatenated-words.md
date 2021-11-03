---
title: "472: Concatenated Words"
excerpt: Concatenated Words
problem_id: 472
categories:
  - Leet Code
tags:
  - Trie
---

## Problem

**Problem ID**: [472](https://leetcode.com/problems/concatenated-words/)

**Title**: Concatenated Words

**Difficulty**: Hard 

**Description**:

Given an array of strings words (without duplicates), return all the concatenated words in the given list of words.

A concatenated word is defined as a string that is comprised entirely of at least two shorter words in the given array.

## Thoughts

This is a very hard problem as the time complexity is potentially exponential and you will need
to perform some sort of pruning or micro optimisation to solve this problem.

Note to self: if you need to check if a multiple different substring exist in a
set string, it would be better to use a Trie instead of a hash set. Trie will only
require the pointer to the current character while hash set will require constructing
a copy of the substring which would incur an increased O(N) complexity.

## Solution

Before jumping into the solution we can make a few observations:
1. A concatenated word can only be made up of smaller concatenated words
2. If word `A` is a concatenated word, word `B` that is a concatenated with word
`A` can be made up of other smaller word other than `A`.


**General Idea**:

1. We will sort the words in ascending order of size. This will allow us to lazily
add large words into the dictionary of words that would have no contribution to the
smaller words (observation #1)
2. Iterate through the words
    1. Check if the current word is concatenated by performing a dfs traversal on the Trie.
    2. If the word is not concatenated, add it to the Trie as the word cannot be formed by other smaller word (observation #2)
    3. Else add the word to the returned words


### Implementation

```cpp
class Node {
    unordered_map<char, unique_ptr<Node>> children;
    bool terminal;
public:
    Node() : children(), terminal(false) {}
    
    Node* get_child(char c) {
        if (has_child(c)) {
            return children[c].get();
        } else {
            return nullptr;
        }
    }
    
    bool has_child(char c) {
        return children.count(c);
    }
    
    Node* get_or_create_child(char c) {
        if (!has_child(c)) children[c] = make_unique<Node>();
        return get_child(c);
    }
    
    void set_terminal() {terminal = true;}
    
    bool is_terminal() {return terminal;}
};

class Solution {
public:
    bool is_concated(Node* root, const string& word, int i, unordered_map<string, bool>& memo) {
        if (memo.count(word)) {
            return memo[word];
        }
        Node* node = root;
        int n = word.size();
        for (;i < n && node; i++) {
            node = node->get_child(word[i]);
            if (!node) break;
            
            if (node->is_terminal() && (i == n-1 || is_concated(root, word, i+1, memo))) {
                memo[word.substr(i, n - i)] = true;
                return true;
            } 
        }
        memo[word.substr(i, n-i)] = false;
        return false;
    }
    
    vector<string> findAllConcatenatedWordsInADict(vector<string>& words) {
        sort(words.begin(), words.end(), [](const auto& a, const auto&b){return a.size() < b.size();});
        unique_ptr<Node> root = make_unique<Node>();
        unordered_map<string, bool> memo;
        vector<string> ret;
        for (const auto& word : words) {
            if(is_concated(root.get(), word, 0, memo)) {
                ret.emplace_back(word);
            } else {
                Node* node = root.get();
                for (char c : word) {
                    node = node->get_or_create_child(c);
                }
                node->set_terminal();
            }
            
        }
        
        return ret;
    }
};
```
