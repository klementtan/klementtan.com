---
title: "679: 24 Game"
excerpt: 24 Game
problem_id: 679
categories:
  - Leet Code
tags:
  - Recursion
---

## Problem 

**Problem ID**: [679](https://leetcode.com/problems/24-game/)

**Title**: 24 Game

**Difficulty**: Hard

**Description**:

You are given an integer array cards of length 4. You have four cards, each containing a number in the range [1, 9]. You should arrange the numbers on these cards in a mathematical expression using the operators `['+', '-', '*', '/']` and the parentheses `'(' and ')'` to get the value 24.

You are restricted with the following rules:

The division operator '/' represents real division, not integer division.
For example, 4 / (1 - 2 / 3) = 4 / (1 / 3) = 12.
Every operation done is between two numbers. In particular, we cannot use '-' as a unary operator.
For example, if cards = [1, 1, 1, 1], the expression "-1 - 1 - 1 - 1" is not allowed.
You cannot concatenate numbers together
For example, if cards = [1, 2, 1, 2], the expression "12 + 12" is not valid.
Return true if you can get such expression that evaluates to 24, and false otherwise.

## Thoughts

From a glance you can see that this is a back tracking problem but the difficulty is identify the right recurssive function. 

## Solution

**General Idea**

As any kind parentheses are allowed, each number could be left associative or right associative (ie (x+y)+z or x + (y+z)). We solve this by recurssively
splitting the numbers into two halves. By doing so we will abstract away the issue of left or right associative, as there
 are only two operands (ie (...) + (...)) in each iteration.

**Implementation**
* `vector<double> helper(vecotor<double> cards)`
  * Given a list of cards returns all the possible values that can be evaluated form the list of cards
* Base case: when the size of `cards` is `1` the only possible value to be evaluated is the card itself
* Recursion:
  * Iteratively split the current cards into two halves
  * Call the recursive function on each half to find the list of possible evaluated
  * For each possible value in the left and right half, iterate through all the operations to get a possible evaluated value for the current cards
* Iterate through the list of all possible evaluated values for the 4 provided digits and find if any of them equals to `24`

**Edge Cases**:
* To prevent divide by `0`, set the divide operation to return a dummy value if RHS is 0.
* Due to floating point precision, we will need to check `24 - epsilon < val < 24 + epsilon`.

### Implementation

```cpp
class Solution {
public:
    vector<function<double(double, double)>> ops = {
        [](double l, double r) {
            return l  + r;
        },
        [](double l, double r) {
            return l  * r;},
        [](double l, double r) {
            return l  - r;
        },
        [](double l, double r) { 
            if (r == 0) return 1e9;
            return l  / r;
        },
    };
    
    vector<double> helper(vector<double> cards) {
        if (cards.size() == 1) return {cards.front()};
        vector<double> ret;
        for (auto it = cards.begin() + 1; it != cards.end(); it++) {
            vector<double> l(cards.begin(), it);
            vector<double> r(it, cards.end());
            vector<double> l_s = helper(l);
            vector<double> r_s = helper(r);
            for (auto l_it = l_s.begin(); l_it != l_s.end(); l_it++) {
                for (auto r_it = r_s.begin(); r_it != r_s.end(); r_it++) {
                    for (auto op : ops) {
                        auto val = op(*l_it, *r_it);
                        ret.push_back(val);
                    }
                } 
            }
        }
        return ret;
    }
    
    bool judgePoint24(vector<int>& cards) {
        vector<double> d_cards;
        for (auto card : cards) d_cards.push_back(card);
        sort(d_cards.begin(), d_cards.end());
        vector<double> vals; 
        do {
            auto curr_vals = helper(d_cards);
            vals.insert(vals.end(), curr_vals.begin(), curr_vals.end());
        } while(next_permutation(d_cards.begin(), d_cards.end()));
        for (auto val : vals) if (val > 23.99 && val < 24.01) return true;
        return false;
    }
};
```
