---
title: "1912: Design Movie Rental System"
excerpt: Design Movie Rental System
problem_id: 1912
categories:
  - Leet Code
tags:
  - Design
---

## Problem

**Problem ID**: [1912](https://leetcode.com/problems/design-movie-rental-system/)

**Title**: Design Movie Rental System

**Difficulty**: Hard

## Thoughts

This problem is not very hard to solve but it requires very careful state management. I found
it very useful to write in comments what the corresponding index/key in the DS represents
and writing the steps that will be required for each step.

My solution manages to get accepted but IMO it is not an elegant solution. Looking through the discussion it seems
that there is no elegant solution to this problem.

## Solution

We can reduce the problem to essentially, finding the cheapest **5** unrented movie-copy for **a given movie**
and the cheapest **5** rented movie-copy for **all movies**.

To solve this problem we will maintain three data structures:
1. `m_rented_movies`: A sorted list of all movies
  - `report()`: return the first 5 elements
  - `rent(int shop_id, int movie_id)`: Add `{price, shop_id, movie_id}`
  - `drop(int shop_id, int movie_id)`: Erase `{price, shop_id, movie_id}`
2. `m_unrented_movies`: A map of movie id to a sorted list of unrented movie-copy of a shop
  - `search(int movie_id)`: return the first 5 elements in the sorted list of `movie_id`
  - `rent(int shop_id, int movie_id)`: Remove `{price, shop_id}` for the sorted list of `movie_id`
  - `drop(int shop_id, int movie_id)`: Add `{price, shop_id}` for the sorted list of `movie_id`
3. `m_shop_movies`: A 2D map to query the price of a movie-copy for a given shop
  - Initialize during construction

### Implementation

```cpp
class MovieRentingSystem {
public:
    // [[price,shop_id, movie_id]]
    set<vector<int>> m_rented_movies;
    // shop_movies[shop_id][movie_id] = price
    unordered_map<int,unordered_map<int,int>> m_shop_movies;
    // m_unrented_movies[movie_id] = [<price, shop_id>]
    unordered_map<int, set<pair<int, int>>> m_unrented_movies;
    
    MovieRentingSystem(int n, vector<vector<int>>& entries) {
          for (const vector<int>& entry : entries) {
              int shop_id = entry[0];
              int movie_id = entry[1];
              int price = entry[2];
              m_unrented_movies[movie_id].insert({price, shop_id});
              m_shop_movies[shop_id][movie_id] = price;
          }
    }
    
    vector<int> search(int movie_id) {
        // return the first 5 in m_unrented_movies[movie_id] 
        set<pair<int,int>>& unrented = m_unrented_movies[movie_id];
        vector<int> shop_ids;
        for (auto it = unrented.begin(); it != unrented.end() && shop_ids.size() < 5; it++) {
            shop_ids.emplace_back(it->second);
        }
        return shop_ids;
    }
    
    void rent(int shop_id, int movie_id) {
                
        int price = m_shop_movies[shop_id][movie_id];
        
        // remove <price, shop_id> from m_unrented_movies[movie_id]
        auto it = m_unrented_movies[movie_id].find({price, shop_id});
        assert(it != m_unrented_movies[movie_id].end());
        m_unrented_movies[movie_id].erase(it);
        
        // add <price, shop_id, movie_id> to m_rented_movies
        m_rented_movies.insert({price, shop_id, movie_id});
    }
    
    void drop(int shop_id, int movie_id) {
        int price = m_shop_movies[shop_id][movie_id];
            
        // remove <price, shop_id, movie_id> from m_rented_movies
        auto it = m_rented_movies.find({price, shop_id, movie_id});
        assert(it != m_rented_movies.end());
        m_rented_movies.erase(it);
        
        // add <price, shop_id> to m_unrented_movies[moved_id]
        m_unrented_movies[movie_id].insert({price, shop_id});
    }
    
    vector<vector<int>> report() {
        // return the first 4 in m_rented_movies
        vector<vector<int>> movies;
        for (auto it = m_rented_movies.begin(); it != m_rented_movies.end() && movies.size() < 5; it++) {
            vector<int> movie = *it;
            movies.push_back({movie[1], movie[2]});
        }
        return movies;
    }
};
```
