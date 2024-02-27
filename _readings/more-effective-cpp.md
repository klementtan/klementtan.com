---
title: "Readings: More effective C++"
toc: true
---

# Basics

## Item 1: Distinguish between pointers and references

References do not allow null
* References must always be initialised
* If initializing a reference with nullptr (UB)
* We should always use references if we know that it will always refer to something

When to use pointers: when it can refer to nothing

Good standard: always return reference for `operator[]`

## Item 2: Prefer C++ - style casts

Downfalls of C style cast:
* does not show the intent of the cast
* can perform any kind of cast (cast always `const` or cast to base)

* `const_cast`: only cast away `const`ness and `volatile`ness
* `static_cast`: like a c sytle cast but without casting away the constness
* `dynamic_cast`: cast down or across an inheritance (ie between uncles)
    * only allowed for structs (cannot use on primitive types)
* `reinterpret_cast`: type conversion between well defined type
