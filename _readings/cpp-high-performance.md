---
title: "C++ High Performance"
toc: true
---

Title: C++ High Performance
Author: Bjorn Andrist and Viktor Sehr

# Chapter 1: A brief Introduction to C++

(klement: This chapter briefly explains C++ which most people should know but
contains some key insights that I have overlooked)

Zero-overhead principle by Bjarne:

- What you don't use, you don't pay
- What you do use, you couldn't hand code any better

Non-performance-related C++ language features
(klement: I am aware of these features but this section provides a very good
structured answer as to why C++. Could be useful in interviews.)

- Value Semantics:
  - Allows callers to be guaranteed that their provided arguments will not be
    modified as all arguments are copied.
- Cons correctness:
  - Provide const guarantees in compile time. Available in java (`final`) but
    it is more flexible as member functions can be delclared const
- Object ownership: with RAII and smart pointers we can clearly express the
object ownership. In Java all resource have shared ownership.
- Deterministic destruction:
  - Uses RAII for destruction which is deterministic.
- Avoiding null objects with references:
  - As references in C++ cannot be null and reassigned, functions that takes
  parameters as reference can be guaranteed that the argument is not a null
  object.
