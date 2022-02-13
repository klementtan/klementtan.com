---
title: "630: Course Schedule III"
excerpt: Course Schedule III
problem_id: 630 
categories:
  - Leet Code
tags:
  - Leet Code
---

## Problem 

**Problem ID**: [630](https://leetcode.com/problems/course-schedule-iii/)

**Title**: Course Schedule III

**Difficulty**: Hard

**Description**:

<p>There are <code>n</code> different online courses numbered from <code>1</code> to <code>n</code>. You are given an array <code>courses</code> where <code>courses[i] = [duration<sub>i</sub>, lastDay<sub>i</sub>]</code> indicate that the <code>i<sup>th</sup></code> course should be taken <b>continuously</b> for <code>duration<sub>i</sub></code> days and must be finished before or on <code>lastDay<sub>i</sub></code>.</p>

<p>You will start on the <code>1<sup>st</sup></code> day and you cannot take two or more courses simultaneously.</p>

<p>Return <em>the maximum number of courses that you can take</em>.</p>

<p>&nbsp;</p>
<p><strong>Example 1:</strong></p>

<pre>
<strong>Input:</strong> courses = [[100,200],[200,1300],[1000,1250],[2000,3200]]
<strong>Output:</strong> 3
Explanation: 
There are totally 4 courses, but you can take 3 courses at most:
First, take the 1<sup>st</sup> course, it costs 100 days so you will finish it on the 100<sup>th</sup> day, and ready to take the next course on the 101<sup>st</sup> day.
Second, take the 3<sup>rd</sup> course, it costs 1000 days so you will finish it on the 1100<sup>th</sup> day, and ready to take the next course on the 1101<sup>st</sup> day. 
Third, take the 2<sup>nd</sup> course, it costs 200 days so you will finish it on the 1300<sup>th</sup> day. 
The 4<sup>th</sup> course cannot be taken now, since you will finish it on the 3300<sup>th</sup> day, which exceeds the closed date.
</pre>

<p><strong>Example 2:</strong></p>

<pre>
<strong>Input:</strong> courses = [[1,2]]
<strong>Output:</strong> 1
</pre>

<p><strong>Example 3:</strong></p>

<pre>
<strong>Input:</strong> courses = [[3,2],[4,3]]
<strong>Output:</strong> 0
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= courses.length &lt;= 10<sup>4</sup></code></li>
	<li><code>1 &lt;= duration<sub>i</sub>, lastDay<sub>i</sub> &lt;= 10<sup>4</sup></code></li>
</ul>


## Thoughts

I found this problem very difficult. I tunnel visioned into the solving this problem using the typical interval + greedy method. However, as each course
is not fixed, that approach will not work. I had to look at the solution to solve this problem

## Solution

**General Idea**
* The key observation you have to make is:
  * Given a list of selected courses we can replace the selected course with a better course (same end but shorter duration)
* In the optimal solution we will schedule all the courses back to back without any gaps in time in between them.
* Sort the courses in ascending order of their ending duration and keep track of the next available time
* Iterate through the sorted course:
  * If the current course can be schedule (current time + duration < end time) then we will add the course duration into the priority queue and advance the time
  * Else if the current course is shorter than the largest course duration in the priority queue.
    * This means that we would replace the largest previous schedule with the current schedule


### Implementation

```cpp
class Solution {
public:
    int scheduleCourse(vector<vector<int>>& courses) {
        sort(courses.begin(), courses.end(), [](const auto& a, const auto& b) { return a[1] < b[1];});
        priority_queue<int> pq;
        
        int time = 1;
        for (const auto course : courses) {
            if (course[1] - course[0] + 1 >= time) {
                time += course[0];
                pq.push(course[0]);
            } else if (!pq.empty() && course[0] < pq.top()) {
                time += (course[0] - pq.top());
                pq.pop();
                pq.push(course[0]);
            }
        }
        return pq.size();
    }
};
```
