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
