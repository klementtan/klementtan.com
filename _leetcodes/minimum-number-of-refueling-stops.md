---
title: "871: Minimum Number of Refueling Stops"
excerpt: Minimum Number of Refueling Stops
problem_id: 871 
categories:
  - Leet Code
tags:
  - Greedy 
---

## Problem

**Problem ID**: [871](https://leetcode.com/problems/minimum-number-of-refueling-stops/)

**Title**: Minimum Number of Refueling Stops

**Difficulty**: Hard

**Description**:

A car travels from a starting position to a destination which is target miles east of the starting position.

There are gas stations along the way. The gas stations are represented as an array stations where stations[i] = [positioni, fueli] indicates that the ith gas station is positioni miles east of the starting position and has fueli liters of gas.

The car starts with an infinite tank of gas, which initially has startFuel liters of fuel in it. It uses one liter of gas per one mile that it drives. When the car reaches a gas station, it may stop and refuel, transferring all the gas from the station into the car.

Return the minimum number of refueling stops the car must make in order to reach its destination. If it cannot reach the destination, return -1.

Note that if the car reaches a gas station with 0 fuel left, the car can still refuel there. If the car reaches the destination with 0 fuel left, it is still considered to have arrived.



## Thoughts

I found this problem very hard and was not able to solve it without looking at the solution. 
I was too fixated on reducing this problem into [Jump Game ii](https://leetcode.com/problems/jump-game-ii/)
and totally missed on the obvious solution.

Note to self: Always have an open mind when tackling any problem and do not tunnel vision on a single solution

## Solution

**General Idea**: If you are driving and run out of petrol, you will choose to refill
at **any** of the previous petrol station. To solve this problem, you will just need to
simulate the state of the car and greedily refill the car once it runs out petrol.

```cpp
class Solution {
public:
    int minRefuelStops(int target, int startFuel, vector<vector<int>>& stations) {
        stations.push_back({target, -INT_MAX});
        sort(stations.begin(), stations.end());
        int tank = startFuel;
        int prev_loc = 0;
        priority_queue<int> fuels;
        int count = 0;
        for (const auto& station : stations) {
            int pos = station[0];
            int fuel = station[1];
            tank -= (pos - prev_loc);
            
            while (!fuels.empty() && tank < 0) {
                const auto top_up = fuels.top();
                tank+= top_up;
                count++;
                fuels.pop();
            }
            if (tank < 0) return -1;
            if (pos == target) return count;
            fuels.push(fuel);
            prev_loc = pos;
            
        }
        return -1;
    }
};
```
