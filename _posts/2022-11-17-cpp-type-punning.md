# Type Punning C++

## Notes on Taking a Byte Out of C++ - Avoiding Punning by Starting Lifetimes - Robert Leahy - CppCon 2022

Bad Mental Model: types are lenses to view a buffer of underlying bytes.

### Bad Example

```cpp
struct foo {
    std::uint32_t a;
    std::uint32_t b;
};
static_assert(sizeof(foo) == sizeof(std::uint64_t));

std::uin32_t bar(std::uint64_t& i, const foo& f) {
    if (f.a == 2) {
        i = 4;
    }
    if (f.a == 2) {
        return f.a;
    }
    return f.b;
}
int main() {
    foo f{2,3};
    bar((std::uint64_t&) f, f);
}
```

**Expected Naive Behaviour**:
1. Initial memory layout of `f`: `0x02, 0x00, 0x00, 0x00, 0x03, 0x00, 0x00, 0x00`
2. First branch taken (`f.a==2`) and load `i = 4` and memory layout is `0x04, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00`
3. Second branch not take `f.a == 2 => false`
4. Go the last return and return `f.b => 0`

**Actual Behviour**:

Generated Assembly:
```asm
bar(std::uint64_t& i, const foo& f)
    mov eax, dword ptr[rsi] # 1. load f.a into eax
    cmp eax, 2              # 2. first branch (f.a == 2)
    je .LBB0_1
    cmp eax, 2
    jne .LBB0_3
.LBB0_4:                    # first branch and second branch take
    ret
.LBB0_1:                    # 3. if first branch take
    mov qword ptr [rti], 4  # 4. store 4 at the memory address of `i` aka `foo`
    cmp eax, 2              # 4. check eax against 2 (f.a == 2) (second branch).
                            #    but no new value is loaded into `f.a`, compiler
                            #    did not reload `f.a` into eax
    je .LBB0_4              # 5. return f.a

.LBB0_3:                    # if first branch not taken but second branch taken
    mov eax, dword ptr [rsi + 4]
    ret
```
* Compiler assume that the store to `i = 4` cannot alias the store to `f.a` so it optimise out reloading `f.a` into eax
* Return `f.a => 2` instead of `f.b`

## Type Punning Rules

Accessing can be accessed by:
1. Through a reference to its type (addition of cv qualification)
    ```cpp
    int i = 0;
    int& y = i;
    y = 3;
    ```
2. Through a reference to its signed or unsigned equivalent
3. Through a reference to `char`, `unsigned char`, `std::byte`
4. Anything else is UB

In the example above we can access `f.a` through `unsigned int&`

```cpp
struct foo {
    std::uint32_t a;
    std::uint32_t b;
};
static_assert(sizeof(foo) == sizeof(std::uint64_t));

std::uin32_t bar(std::uint64_t& i, const foo& f) {
    if (f.a == 2) {
        i = 4;
    }
    if (f.a == 2) {
        return f.a;
    }
    return f.b;
}
int main() {
    foo f{2,3};
    bar((std::uint32_t&) f, f);
}

```

Generated Assembly:
```asm
bar(std::uint32_t& i, const foo& f)
    mov eax, dword ptr[rsi] 
    cmp eax, 2              
    je .LBB0_1
    cmp eax, 2
    jne .LBB0_3
.LBB0_4:                    
    ret
.LBB0_1:                    
    mov dword ptr [rti], 4  
    mov eax, dword ptr [ris] <-- reload f.a!
    cmp eax, 2              
                            
    je .LBB0_4              

.LBB0_3:                    
    mov eax, dword ptr [rsi + 4]
    ret
```

## C++ Object Model 

Model:
* Bytes: supply storage for objects
* Objects: have lifetime. Duration of storage not necessarily the same as object lifetime
* Accessing object outside of lifetime is UB

Storage vs Object:
* Object requires **lifetime begin and end** but storage does not have a concept of storage.

C++ Types Invariants:
* Invariants are estalbed by constructor
* If an invariant cannot be established, the constructor can throw an exception and there is no way for
user code to be able to access an object with invalid invariants.

### Aggregate Type

* An array is an aggregate
* class/structs/unions (must satisfy all):
    * No **user-declared** constructor - compiler generated constructor are ok
    * No `private` or `protected` **non-static** data member
        * Everything need to be public if non-static
    * No base class
    * No virtual functions
* Note:
    * Array of non-aggregate type is an aggregate type because it is an array
    * Aggregate can have user-declared/user-defined copy-assignment operator and/or destructor, just not constructor
    * Benefit: can be Initialised using `{}`
    * Struct containing string is not an aggregate because std::string have a custom constructor

**Initializing an array**:
* if the number of element in the initializer list is the **same** as the array size, element wise construction
* if the number of element in the initilizer list is **smaller** than the array size, first `m` will be element wise constructed and the rest will be **value initialized**
* if the number of element in the initilizer list is **larger** than the array size, error
* else: the array did not specify the size -> the array size will be assigned the number of elements in the initializer list

**Value Initialization**:
* Scalar Types (`bool`, `int`, `char`, `double`): initialized to `0` for that type, `false` for `bool` and `0.0` for `double`
* User Type:
    * If there is user declared default constructor, default constructor called
    * If the default constructor is implicity defined, all non-static members are **recursively value initialized**
    * **Reference cannot** be value initialized 
    * Value initialization for **non-aggregate class** can fail

Philosophy of Aggregate Type:
* An aggregate type is just a **sum of its members** (literally an aggregate)
* If a class needs a custom constructor, implies more work need to be done for construction and only initializing the members is wrong.

TODO: add POD type


### Trivail Types
* types where the constructor and destructor does not need to do anything
* Trivial types are still types with lifetime and we need to begin and end the lifetime.

### Implicit Lifetime Types

Criteria (one of):
* Array types
* Scalar Types
* Implicit-lifetime class types:
    * aggregate
    * or at least **one trivial eligible constructor** and **trivial non-deleted destructor**
        * Less restrictive than trivial (require all constructor to be trivial)

Creating an implicit lifetime type operations:
* `std::malloc`, `std::memcpy`, `::memmove`, starting lifetime with buffer (array of `char`, `unsigned char`, `std::byte`) and operator `new` and `new[]`

How does these operation start lifetime of a specific type if `malloc` returns `void*`?
* Performing these operations will implicitly start the lifetime of all implicit lifetime type (super position) - Can be many types.
* Performing specific type operation on this super position set will reduce the set of types to it - well defined
    * Leaving the types as all possible types is UB

```cpp
// implicitly start lifetime of ALL implicit lifetime types
// that the storage have sufficient size and alignment for
const auto ptr = (int*) std::malloc(sizeof(int)*4);


for (int i = 0; i < 4; ++i) {
    // implicitly create a lifetime of integer
    ptr[i] = i;
}
```

### Solving Type Punning

`std::bit_cast`: takes a reference `From&` to `To` type
* Problem `static_cast` a `void*` to `To*` require that the prvoided argument (`To*`) has already started a lifetime of `To*`
* Copies the `From&` into a buffer with `mempcy` -> implicitly starts the lifetime of a superset of types that includes `To`
* Reduce the set to `To` by `reinterpret_cast`-ing the buffer to `To`

As-if rule: the optimiser other than RVO cannot create the observable changes

`start_lifetime_as`: takes a pointer to storage and implicitly start lifetime of `T*`
* starts the lifetime of an implicit type on a storage.

### Ending an Object's Lifetime

* Value object goes out of scope: `t.~T()` or `ptr->~T()`
* Reuse of backing storage: if you reuse the storage of an implicit type, the original implicit type lifetime has ended.

`std::launder`: deal with reusing storage
* Problem:
    * Reusing storage invalidte pointers and references to the old object
    * Pointer points to the storage but no longer to the object - lifetime ended
* `std::launder` gives a pointer to the object from a pointer to the storage - do not start or end lifetime? (tell compiler you know better)

```cpp
open_query q(/* */);
//...
erased_update* ptr = q.last_update();

// getting a dervided class from the base class by `std::bit_casting`
// erased_update lifetime will ended as the storage is reused for update
update* auto u = update_as<update>(*ptr).value();

// tell the compiler to treat the storage that already has an object
ptr = std::launder(ptr);
std::cout << "Timestamp=" << ptr->time << std::endl;
std::cout << "Seq=" << u->seqno<< std::endl;
```

