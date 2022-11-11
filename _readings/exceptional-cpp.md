---
title: "Exceptional C++"
excerpt: "Notes for Exceptional C++"
toc: true
---

Title: Exceptional C++ 47 Engineering Puzzles, Programming Problems, and Solutions

Author: Herb Sutter

## Item 1: Iterators

```cpp
int main()
{
    vector<Date> e;
    copy( istream_iterator<Date>( cin ), istream_iterator<Date>(), back_inserter( e ));
    vector<Date>::iterator first = find( e.begin(), e.dn(), "01/01/95" );
    vector<Date>::iterator last = find( e.begin(), e.dn(), "12/31/95" );
    *last = "12/31/95";
    copy( first, last, ostream_iterator<Date>( cout, "\n") );
    e.insert( --e.end(), TodaysDate() );
    copy( first, last, ostream_iterator<Date>( cout, "\n") );
}
```

**Problems**:


```cpp
vector<Date> e;
copy( istream_iterator<Date>( cin ), istream_iterator<Date>(), back_inserter( e ));
```
* No problem
* As long as `Date` class implement the extractor function (`operator>>(istream&, Date&)`), `istream_iterator` will use this function
that reads `Date` from `cin`
* Copy

```cpp
vector<Date>::iterator first = find( e.begin(), e.end(), "01/01/95" );
vector<Date>::iterator last = find( e.begin(), e.end(), "12/31/95" );
*last = "12/31/95";
```
* `last`: might be `e.end()` and not deferenc-able
* `find` will return the second argument if it cannot find the element

```cpp
copy( first, last, ostream_iterator<Date>( cout, "\n") );
```
* Invalid: because `[first, last)` might not be a valid range - first after last
* `std::copy` requires that the `first` is before `last`


```cpp
e.insert( --e.end(), TodaysDate() );
```
* `--e.end()` - illegal, modifying temporaries of builtin types
    * `vector<Date>::iterator`: most implementation use pointer to underlying `Date*` as the implementation of it
    * `e.end()` is a temporary during an expression and `--e.end()` will modify the temporary builtin type
        ```cpp
        Date* f(); // function that returns a Date*
        p = --f(); // error, because f() is a temporary type
        ```
        * Should use `e.end() - 1` instead
* If `e` is empty then any attempt to insert to the iterator before `e.end()` is invalid

```cpp
copy( first, last, ostream_iterator<Date>( cout, "\n") );
```
* `first` and `last` might not be valid anymore - pointer invalidation
* The previous the statement before modifies the vector and the pointer might get invalidated due to the vector growing


## Item 2: Case-Insensitive Strings - Part 1

* To implement a case insensitive string (or any custom string), we can easily define a custom `traits` (or a specialization of `char_traits`)
* The custom traits will need to just define the custom `eq()` `lt()` `compare()` `find()`
    ```cpp
    struct ci_char_traits : public char_traits<char>
    {
        static bool eq(char c1, char c2) { return toupper(c1) == toupper(c2); }
        static bool lt(char c1, char c2) { return toupper(c1) < toupper(c2); }
        static int compare(const char* s1, const cahr* s2, size_t n) {...}
        static const char* find(const char* s, int n, char a) {...}
    }
    typedef basic_string<char, ci_char_traits> ci_string
    ```
* Note: for custom comparison traits, you will need to compare transitivity
    * `ci_string{"aAa"} == string{"aaa"}` and `yz_string{"AAa"} == string{"aaa"}` does this mean `ci_stirng{"aAa"} == yz_string{"AAa"}`
* If the string is just for custom comparison, might be better to define a custom comparison function instead.

## Item 3: Case-Insensitive Strings - Part 2

* Inheriting from type traits (from part 1)
    * Traditionally should model a IS-A / WOKRS-LIKE-A (LSP) - however in this scenario there
    is no dynamic polymorphism and LSP principle of derived classes being able to satisfy all
    the requirements of the base class does not work
    * Uses Generic LSP (GLSP): for classes derived of tags and traits class, they just need to confirm to the
    requirement list of the template argumenet.
        * Requirement List: the requirement for that template arguement

## Item 4 and 5: Maximally Reusable Generic

Fixed vector example:
```cpp
template<typename T, size_t size>
class fixed_vector
{
public:
    typedef T*          iterator;
    typdef const T*     const_iterator;
    fixed_vector() { }
    template<typename O, size_t osize>
    fixed_vector(const fixed_vector<0, osize>& other)
    {
        copy(other.begin(), other.begin() + min(size, osize), begin());
    }
    template<typename O, size_t osize>
    operator=(const fixed_vector<O, osize>&other)
    {
        coppy(other.begin(), other.begin() + min(size, osize), begin());
        return *this;
    }
}
```
* Templated Constructor:
    * By the standard, they are not a copy constructor and a copy assignment.
    * They are not allowed to suppress the default copy constructor and copy assignment
    * ie: `T` cant be the `X` (`X` is the class type it self)
    * Will participate in overload resolution
* Why default constructor: any custom defined contructor will not generate the default
constructor
* Exception safety:
    * Given copy assignment could fail when we copy a portion of the element and an
    exception was thrown
    * Strong Exception safety for assignment requires **atomic and nonthrowing `Swap()`** (ie copy and swap)
    * For Generic value type array, there is no way to perform atomic nonthrowing swap
    * Solution: dynamically allocate the array instead of having it as a value member

## Item 6: Temporary Objects

Problematic Code
```cpp
string FindAddr(list<Employee> emps, string name)
{
    for (auto i = emps.begin(); i != emps.end(); i++)
    {
        if (*i == name) {
            return i->addr;
        }
        return "";
    }
}
```
* Passing by values:
    * Arguments are passed by values -> uneccessary copies
* `i != emps.end()`:
    * For each iteration we are creatign a sentinel `emps.end()` iterator -> unnecessary construction and destruction of it
* Postincrement (`i++`):
    * Less efficient as it produces an unnecessary "temporary" of the old value
    * To implement a post increment use the canonical form:
        ```cpp
        const T T::operator++(int) {
            T old(*this);
            ++*this; // use the preincrement -> unneccassry 
            return old;
        }
        ```
    * To allow the compiler to optimise out post increment, define the canonical form of post increment and define it as `inline` -> compiler will be able to see that there is an uneccessary temporary
* `*i == name`: there will be a temporary generated that converts from employee to string or string to employee
    * Would be better to do `i->name == name` if it is possible
* Single return path:
    * Alternative:
        ```cpp
        string ret;
        ...
        ret = i->addr;
        ...;
        return ret;
        ```
    * This creates a temporary `ret` which will be default constructed
* Returning a reference:
    * depends on the situation as it can easily lead to dangling references
 

