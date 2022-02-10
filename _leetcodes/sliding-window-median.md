---
title: "480: Sliding Window Median"
excerpt: Sliding Window Median
problem_id: 480
categories:
  - Leet Code
tags:
  - Sliding Window
---

## Problem

**Problem ID**: [480](https://leetcode.com/problems/sliding-window-median/)

**Title**: Sliding Window Median

**Difficulty**: Hard

**Description**:

The median is the middle value in an ordered integer list. If the size of the list is even, there is no middle value. So the median is the mean of the two middle values.

For examples, if arr = [2,3,4], the median is 3.
For examples, if arr = [1,2,3,4], the median is (2 + 3) / 2 = 2.5.
You are given an integer array nums and an integer k. There is a sliding window of size k which is moving from the very left of the array to the very right. You can only see the k numbers in the window. Each time the sliding window moves right by one position.

Return the median array for each window in the original array. Answers within 10-5 of the actual value will be accepted.

## Thoughts

I initially went of the approach of storing the sliding window in a multiset and at each iteration, iterate through the sliding window multiset to get the two median values. This approach is `O(NKlog(k))` as we will slide the windows `N` times, each slide we will add and remove an element (`log(K)`) and calculate the median (`K`) but traversing the whole multiset. Although this approach is accepted by LC, it is very slow as multiset does not support random access.

After looking at the solution, I realised I could use a similar technique (two multiset) as another LC question([295](https://leetcode.com/articles/find-median-from-data-stream)).
However, implementing this approach is tricker as there is a input data stream and an output data stream.
After implementing with this approach my execution increased by 10x (600ms -> 60ms)

## Solution

**General Idea**

We will always maintain two multiset `l` and `r` where the largest number in `l` is smaller than the biggest number in `r`
and we will ensure that the both sets can only have at most 1 more element than the other. By doing so will can reduce the time
for having the median two elements from `O(K)` to `O(logK)` (accessing smallest/largest element in balanced BST is `logN`) and the final
time complexity is `O(N(4*log(K))) -> O(NlogK)`

**Implementation**:
* I chose to create helper functions that will take the two multisets by reference and mutate them
* `balance`: rebalances `l` and `r` will ensure that `abs(l.size() - r.size()) < 2` while maintaining the largest in `l` less than the smallest in `r`
* `insert_val`: insert a value into `l` or `r` while maintaining the properties of `l` and `r`
* `remove_val`: remove a value from `l` or `r`
* `median`: calculate the median from the 2 multiset

### Implementation

```cpp
class Solution {
public:
    
    
    void balance(multiset<int, greater<int>>& l_wind, multiset<int>& r_wind) {
        // max(l_wind) < min(r_wind)
        
        if (l_wind.size() == r_wind.size()) return;
        
        while (abs<int>(l_wind.size() - r_wind.size()) > 1) {
            if (l_wind.size() > r_wind.size()) {
                int val = *l_wind.begin();
                l_wind.erase(l_wind.begin());
                r_wind.insert(val);
            } else {
                int val = *r_wind.begin();
                r_wind.erase(r_wind.begin());
                l_wind.insert(val);
            }
        }
    }
    
    void insert_val(multiset<int, greater<int>>& l_wind, multiset<int>& r_wind, int val) {
        int l_max = l_wind.empty() ? -INT_MAX : *l_wind.begin();
        int r_min = r_wind.empty() ? INT_MAX : *r_wind.begin();
        if (val >= r_min) {
            r_wind.insert(val);
        } else {
            l_wind.insert(val);
        }
    }
    void erase_val(multiset<int, greater<int>>& l_wind, multiset<int>& r_wind, int val) {
        int l_max = l_wind.empty() ? -INT_MAX : *l_wind.begin();
        int r_min = r_wind.empty() ? INT_MAX : *r_wind.begin();
        if (val >= r_min) {
            assert(r_wind.find(val) != r_wind.end());
            r_wind.erase(r_wind.find(val));
        } else {
            assert(l_wind.find(val) != l_wind.end());
            l_wind.erase(l_wind.find(val));
        }
    }
    
    double median(multiset<int, greater<int>>& l_wind, multiset<int>& r_wind) {
        bool is_valid = !(l_wind.empty() && r_wind.empty());
        assert(is_valid);
        assert(abs<int>(r_wind.size() - l_wind.size()) < 2);
        if (l_wind.size() == r_wind.size()) {
            return (static_cast<double>(*l_wind.begin()) + static_cast<double>(*r_wind.begin()))/2; 
        } else {
            return static_cast<double>(l_wind.size() > r_wind.size() ? *l_wind.begin() : *r_wind.begin());
        }
    }
    
    vector<double> medianSlidingWindow(vector<int>& nums, int k) {
        multiset<int, greater<int>> l_win;
        multiset<int> r_win;
        int to_remove = 0;
        int to_add = 0;
        for (; to_add < k; to_add++) {
            insert_val(l_win, r_win, nums[to_add]);
            balance(l_win, r_win);
        }
        vector<double>ret(nums.size() - k + 1);
        
        for (int i = 0; i < nums.size() - k + 1; i++) {
            balance(l_win, r_win);
            ret[i] = median(l_win, r_win);
            
            if (to_add < nums.size()) insert_val(l_win, r_win, nums[to_add]);
            erase_val(l_win, r_win, nums[to_remove]);
            to_add++;
            to_remove++;
        }
        return ret;
    }
};
```
