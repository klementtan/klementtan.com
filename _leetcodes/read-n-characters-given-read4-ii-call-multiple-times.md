---
title: "158: Read N Characters Given read4 II - Call Multiple Times"
excerpt: Read N Characters Given read4 II - Call Multiple Times
problem_id: 158 
categories:
  - Leet Code
tags:
  - String 
---

## Problem

**Problem ID**: [158](https://leetcode.com/problems/read-n-characters-given-read4-ii-call-multiple-times/)

**Title**: Read N Characters Given read4 II - Call Multiple Times

**Difficulty**: Hard

**Description**:

Given a file and assume that you can only read the file using a given method read4, implement a method read to read n characters. Your method read may be called multiple times.

Method read4:

The API read4 reads four consecutive characters from file, then writes those characters into the buffer array buf4.

The return value is the number of actual characters read.

Note that read4() has its own file pointer, much like FILE *fp in C.

## Thoughts

I found this question quite interesting but I think Leetcode could have written a better
problem description as it seems much more complicated than what it actually is. I strongly
advise attempting [Read N Characters Given Read4](https://leetcode.com/problems/read-n-characters-given-read4/)
first.

## Solution

**Challenge:
* `read4` will always read 4 characters. This could result in separate calls to `read` sharing the same chunk
of characters from `read4` (ie read(buf, 2) then read(buf,2))


**General Idea**: Instead of writing to `read`'s buffer directly, we will store the results from `read4` into a
separate buffer. This would let us to easily share the same chunk of characters from `read4`. 


**State**:

* `read_buf`: An array of 4 characters. Act as a buffer to store the result for all calls to `read4`.
* `size`: As `read4` could read less than 4 characters at the end of the file. We will need to store
the number of characters that are in `read_buf`
* `ptr`: Pointer to the next character in the buffer. This would let us to keep track of
the current character in `read_buf`

**Solution**
* While the return buffer size is less than `n`
  * If `ptr` size equals to size of `read_buf`, signifies than we have reached the end of the buffer.
  Call `read4` and reset the value `ptr` and `size`
  * Break from while loop if `size == 0` (reached the end of file).
  * Copy over the character from `read_buf` into the return `buf` and increment `ptr`

### Implementation

```cpp
/**
 * The read4 API is defined in the parent class Reader4.
 *     int read4(char *buf4);
 */

class Solution {
private:
    char read_buf[4];
    bool read_empty;
    int size;
    int ptr;
public:
    Solution() : read_empty(false), size(0), ptr(0) {}
    /**
     * @param buf Destination buffer
     * @param n   Number of characters to read
     * @return    The number of actual characters read
     */
    void refresh_buf() {
        size = read4(read_buf);
        ptr = 0;
        if (size == 0) read_empty = true;
    }
    int read(char *buf, int n) {
        int buf_size = 0;
        for (;buf_size < n; buf_size++) {
            if (ptr == size) {
                refresh_buf();
            }
            if (read_empty) break;
            if (ptr < size) {
                buf[buf_size] = read_buf[ptr];
                ptr++;
            }
        }
        return buf_size;
    }
};
```
