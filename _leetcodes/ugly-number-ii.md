---
title: "264: Ugly Number II"
excerpt: Ugly Number II
problem_id: 264 
categories:
  - Leet Code
tags:
  - Leet Code
---

## Problem

**Problem ID**: [264](https://leetcode.com/problems/ugly-number-ii/)

**Title**: Ugly Number II

**Difficulty**: Medium

**Description**:

<p>An <strong>ugly number</strong> is a positive integer whose prime factors are limited to <code>2</code>, <code>3</code>, and <code>5</code>.</p>

<p>Given an integer <code>n</code>, return <em>the</em> <code>n<sup>th</sup></code> <em><strong>ugly number</strong></em>.</p>

<p>&nbsp;</p>
<p><strong>Example 1:</strong></p>

<pre>
<strong>Input:</strong> n = 10
<strong>Output:</strong> 12
<strong>Explanation:</strong> [1, 2, 3, 4, 5, 6, 8, 9, 10, 12] is the sequence of the first 10 ugly numbers.
</pre>

<p><strong>Example 2:</strong></p>

<pre>
<strong>Input:</strong> n = 1
<strong>Output:</strong> 1
<strong>Explanation:</strong> 1 has no prime factors, therefore all of its prime factors are limited to 2, 3, and 5.
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= n &lt;= 1690</code></li>
</ul>


## Thoughts

I found this problem very hard and had to look at the solution.

## Solution

**General Idea**

Key Observations:

* The `k`th ugly number is a `2`, `3` or `5` multiple of a previous `w`th ugly
number where `w` is less than `i`.
* To find the `w`th ugly number that will lead to `i`th ugly number, `w` is either
the index of a previous ugly number that multiple by `2`, `3` or `5` will give the
an ugly number that is just above `i-1`th ugly number


Algorithm:
* To find the `w`th ugly number that would lead to the `k`th ugly number, we will keep
a three indexes:
  * `i`: the smallest index such that when `i`th ugly number multiply by `2`, the number is more than `i-1`th ugly number
  * `j`: same as `i` but for `3`
  * `k`: same as `i` but for `5`
* Advance `i`, `j`, `k` by `1` every time we compute a new ugly number


### Implementation

```cpp
```
