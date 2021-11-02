---
title: "1928: Minimum Cost to Reach Destination in Time"
excerpt: Minimum Cost to Reach Destination in Time
problem_id: 1928 
categories:
  - Leet Code
tags:
  - Graph
---

## Problem

**Problem ID**: [1928](https://leetcode.com/problems/minimum-cost-to-reach-destination-in-time/)

**Title**: Minimum Cost to Reach Destination in Time

**Difficulty**: Hard

**Description**:

There is a country of n cities numbered from 0 to n - 1 where all the cities are connected by bi-directional roads. The roads are represented as a 2D integer array edges where edges[i] = [xi, yi, timei] denotes a road between cities xi and yi that takes timei minutes to travel. There may be multiple roads of differing travel times connecting the same two cities, but no road connects a city to itself.

Each time you pass through a city, you must pay a passing fee. This is represented as a 0-indexed integer array passingFees of length n where passingFees[j] is the amount of dollars you must pay when you pass through city j.

In the beginning, you are at city 0 and want to reach city n - 1 in maxTime minutes or less. The cost of your journey is the summation of passing fees for each city that you passed through at some moment of your journey (including the source and destination cities).

Given maxTime, edges, and passingFees, return the minimum cost to complete your journey, or -1 if you cannot complete it within maxTime minutes.

## Thoughts

I initially thought this was an exponential time problem where you need to traverse all the paths
to the destination in decreasing order of cost (using dijkstra). However, if we sort by fees then 
time and only visit State that would improve time/fees, we would be able to traverse each edge and
node at most once.

Note to self: Always consider the time spent copying the stl containers. My initial solution was to 
have each `State` contain a `unordered_set<int> visited` data member. This could be avoided by
having a vector that store the current minimum time and fees.

## Solution

**Graph**:

* Each city is a node in the Graph.
* The weight of the edge would be the cost of entering the new city.

**Modified Dijkstra**:

* Sort by the fees then time
* Only visit the next node if the current path decrease the time or cost.

### Implementation

```cpp
class State {
public:
    int u;
    int time;
    int fees;
    State(int u, int fees) : u(u), time(0), fees(fees) {}
};

struct CmpState {
  bool operator()(const State& a, const State& b) {
      
      if (b.fees == a.fees) {
          return b.time < a.time;
      } else {
      return b.fees < a.fees;
      }
  }
};

class Solution {
public:
    int minCost(int maxTime, vector<vector<int>>& edges, vector<int>& passingFees) {
        int n = passingFees.size();
        // g[u] = [{v_i, dist}]
        vector<vector<pair<int,int>>> g(n);
        for (const auto& edge : edges) {
            int u = edge[0];
            int v = edge[1];
            int time = edge[2];
            g[u].push_back({v, time});
            g[v].push_back({u, time});
        }
        
        priority_queue<State, vector<State>, CmpState>pq;
        pq.push(State(0, passingFees[0]));
        vector<int> dp_time(n, INT_MAX);
        vector<int> dp_cost(n, INT_MAX);
        while (!pq.empty()) {
            State u_state = pq.top();
            pq.pop();
            if (u_state.time > maxTime) continue;
            if (u_state.u == n-1 && u_state.time <= maxTime) return u_state.fees;
            
            cout << u_state.u << endl;
            for (const auto [v, dist] : g[u_state.u]) {
                State v_state(u_state);
                v_state.time += dist;
                v_state.fees += passingFees[v];
                v_state.u = v;
                if (v_state.time > maxTime) continue;
                
                if(v_state.time < dp_time[v] || v_state.fees < dp_cost[v]) {
                    dp_cost[v] = v_state.fees;
                    dp_time[v] = v_state.time;
                    pq.push(v_state);
                }
            }
        }
        return -1;
    }
};
```
