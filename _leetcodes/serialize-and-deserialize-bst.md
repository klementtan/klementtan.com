---
title: "449: Serialize and Deserialize BST"
excerpt: Serialize and Deserialize BST
problem_id: 449
categories:
  - Leet Code
tags:
  - DP
---

## Problem

**Problem ID**: [449](https://leetcode.com/problems/serialize-and-deserialize-bst/)

**Title**: Serialize and Deserialize BST

**Difficulty**: Medium

**Description**:

Serialization is converting a data structure or object into a sequence of bits 
so that it can be stored in a file or memory buffer, or transmitted across a 
network connection link to be reconstructed later in the same or another computer environment.

Design an algorithm to serialize and deserialize a binary search tree. There is 
no restriction on how your serialization/deserialization algorithm should work. 
You need to ensure that a binary search tree can be serialized to a string, and this string can be deserialized to the original tree structure.

The encoded string should be as compact as possible.

## Thoughts

I have previously completed [Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)
problem and could easily reuse that solution to the problem. However, the generalized solution for all Binary Tree requires the usage of null nodes to state the represent the end of leaves.

I tried very hard to find a more optimal solution that is specific to BST
but had to eventually rely on the provided solution to do so.

The solution provided by LC is very interesting as it uses the post-order
position of the nodes to rebuilt the identical BST which is new to me.

## Solution

This solution utilises the concept that the left sub tree of a BST always contain values
that are smaller than the node and the right sub tree always contain values that are
larger than than the node. The last node of a postorder traversal will always 
be the parent node for all other nodes.

Combining both concept we can form a recursive function that has upper and lower bound value.
When the last node in the postorder traversal is not within the upper and lower range, it means
that that the last node is no longer the descendant of the previous nodes and the last node
belongs to the next subtree. Using this idea we can rebuilt the BST by constantly updating the
lower and upper value.


### Implementation

```cpp
class Codec {
public:
    string kDelimiter = ",";
    // Encodes a tree to a single string.
    string serialize(TreeNode* root) {
        if (!root) return "";
        string ret;
        if (root->left) {ret += serialize(root->left);
        ret += kDelimiter;}
        if (root->right) {ret += serialize(root->right);
        ret += kDelimiter;}
        ret += to_string(root->val);
        return ret;
    }
    
    TreeNode* recur(int upper, int lower, vector<int>& nodes) {
        if (nodes.empty()) return nullptr;
        int root_val = nodes.back();
        if (root_val < lower || root_val > upper) return nullptr;
        nodes.pop_back();
        TreeNode* root = new TreeNode(root_val);
        root->right = recur(upper, root_val, nodes);
        root->left = recur(root_val, lower, nodes);
        return root;
    }
    
    // Decodes your encoded data to tree.
    TreeNode* deserialize(string data) {
        if (data.empty()) return nullptr;
        vector<int> nodes = SplitDelimiter(data);
        return recur(INT_MAX, INT_MIN, nodes);
    }
    vector<int> SplitDelimiter(string data) {
        vector<int> ret;
        size_t end = 0;
        cout << data << endl;
        while((end = data.find(kDelimiter)) != std::string::npos) {
            string tok = data.substr(0, end);
            cout << end << endl;
            ret.push_back(stoi(tok));
            data.erase(0, end+1);
        }
        assert(data.size() > 0);
        cout << data << endl;
        ret.push_back(stoi(data));
        return ret;
    }
};
```
