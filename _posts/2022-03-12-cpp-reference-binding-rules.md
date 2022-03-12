---
title: "Post: C++ reference binding rules"
categories:
  - Post
tags:
  - C++
excerpt: "Reference binding rules."
---

There are not a lot of materials covering the rules on what types of arguments
can bind to the various types of parameters. If found the blog post on
["What are const rvalue references good for?"](https://codesynthesis.com/~boris/blog//2012/07/24/const-rvalue-references/)
and [Reference declaration](https://en.cppreference.com/w/cpp/language/reference)
page on C++ reference very useful in helping me understand the binding rules.

## Binding Rules

### const <-> non-const

Rule: `const` and non-`const` arguments can be bind to `const` ref parameters but
`const` arguments cannot bind to non-`const` parameters.

My reasoning: This makes sense as converting object to `const` on the callee side will not
change anything on the caller side even if the object is non-`const`. However,
the converse will not be true as converting a `const` object on the caller side
to non-`const` object on the callee side can allow the callee to modify the object
even though the caller expects the object to be `const`.

Example:
```cpp
void cfoo(const int& x) {}
void foo(int& x) {}
int main() {
    int x = 1;
    const int cx = 1;
    cfoo(x);
    cfoo(cx);

    foo(x);
    // error: binding reference of type 'int&' 
    // to 'const int' discards qualifiers
    foo(cx); 
}
```

### lvalue -> rvalue

Rule:`lvalue` arguments cannot bind to `const` and non-`const` `rvalue` parameters.

Explanation: As rvalue parameters indicates that the object is temporary and can
be moved away, allowing `lvalue` to bind to `rvalue` will be catastrophic as the
callee can move away the caller's object. Even though, `const` `rvalue` ref does
not make sense it will actually acts like a `const` `lvalue` ref and I am not 
sure why the standard does not allow it.

Example:
```cpp
void rfoo(int&& x) {}
void crfoo(const int&& x) {}
int main() {
    int x = 1;

    // error: cannot bind rvalue reference of type 
    // 'int&&' to lvalue of type 'int'
    rfoo(x);

    // error: cannot bind rvalue reference of type
    // 'const int&&' to lvalue of type 'int'
    crfoo(x);
}
```

### rvalue -> lvalue

Rule: `rvalue` arguments cannot bind to non-`const` `lvalue` parameters.

Explanation: I don't know what is the reason for this rule. If a caller
pass a temporary object (rvalue) to the callee, it should not matter what
type of object the callee sees it as? The only explanation I can think of
is that some functions utilise *out-parameters* and it does not make sense
if the argument provided is a temporary object (rvalue).

Example:
```cpp

void lfoo(int& x) {}
int main() {
    // error: cannot bind non-const lvalue
    // reference of type 'int&' to an rvalue of type 'int'
    lfoo(1);
}
```

### All (lvalue, rvalue, const, non-const) -> const lvalue 

Rule: `lvalue`, `rvalue`, `const` or non-`const` objects can
bind to `const` `lvalue` parameters.

Explanation: `const` `lvalue` indicates that the callee wants
a read-only view of the object and it does not matter what type
of object that the caller pass as the argument. Thus, the standard
allows all types of arguments to bind to `const` `lvalue`.
(klement: But why does this not apply for `const` `rvalue` parameters?)

Example:
```cpp
void clfoo(const int& x) {}

int main() {
    int x = 1;
    
    clfoo(x);
    clfoo(1);
    
}
```
