---
title: "309: Best Time to Buy and Sell Stock with Cooldown"
excerpt: Best Time to Buy and Sell Stock with Cooldown
problem_id: 309 
categories:
  - Leet Code
tags:
  - DP
---

## Problem 

**Problem ID**: [309](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

**Title**: Best Time to Buy and Sell Stock with Cooldown

**Difficulty**: Medium

**Description**:
You are given an array prices where prices[i] is the price of a given stock on the ith day.

Find the maximum profit you can achieve. You may complete as many transactions as you like (i.e., buy one and sell one share of the stock multiple times) with the following restrictions:

After you sell your stock, you cannot buy stock on the next day (i.e., cooldown one day).
Note: You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).


## Thoughts

I found this problem very difficult and had to look at the solution. I was not able to understand the solution
initially and had to slowly attempt it again  before finally being able to understand it.


## Solution

**General Idea**

We will keep track of two separate states; what is the maximum profit if the last action till `i` inclusive
is **buy** or **sell** and the answer would be max profit if the last action on day `n-1` is sell.

**DP states**

* `buy[i]`
  * Represents the max profit if the last action till day `i`(inclusive) is buy.
  * This means that we could either buy the stock on `i` or buy on `i-1`, `i-2`,... but from  `i-k+1` till `i`
    we cannot perform any action.
  * `buy[0] = -prices.front()`: we will need to deduct the cost of stock from our current profit($0) if we choose
    to buy on day 1
* `sell[i]`
  * Represents the max profit if the last action till day `i`(inclusive) is sell.
  * This means that we could either sell the stock on `i` or buy on `i-1`, `i-2`,... but from  `i-k+1` till `i`
    we cannot perform any action.
  * `sell[0] = 0`: We will not make any profit we sell on day 0
 
**DP transitions**:
* `buy[i] = max(buy[i-1], i > 1 ? sell[i-2] - prices[i] : -prices[i]);`
  * `buy[i-1]`: We can either no do anything (carry the max buy from previous day)
  * `i > 1? sell[i-2] - prices[i] : -prices`: We could take the profit from selling on `i-2`, take `i-1` as a cool down day
    and buy on `i`
* `sell[i] = max(sell[i-1], buy[i-1] + prices[i])`
  * `sell[i-1]` We can either don't do anything
  * `buy[i-1] + prices[i]`: sell at `price[i]`
    * As buy has `-prices[i]`, this means that `buy[i-1]` already contain the lowest cost to buy the stock. Thus we could simply
    add `prices[i]` to count the profit
  


### Implementation

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        vector<int> buy(n);
        vector<int> sell(n);
        buy[0] = -prices[0];
        sell[0] = 0;
        for (int i = 1; i <n; i++) {
            buy[i] = max(buy[i-1], i > 1 ? sell[i-2] - prices[i] : -prices[i]);
            sell[i] = max(sell[i-1], buy[i-1] + prices[i]);
        }

        return sell[n-1];
    }
};
```
