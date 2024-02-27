---
title: "Post: Linking, Loaders and Shared Libraries"
categories:
  - Post
tags:
  - OS
---

Notes of the following resource:
* [Linkers, Loaders and Shared Libraries in Windows, Linux, and C++ - Ofek Shilon - CppCon 2023](https://www.youtube.com/watch?v=_enXuIxuNV4&list=PLqdbPkkHHWL7qB0wA7pePTTX4oI-xOsOu&index=1&t=1520s)
* [ELF interposition and -Bsymbolic](https://maskray.me/blog/2021-05-16-elf-interposition-and-bsymbolic)

## Intro

Binary: shared object and executable

**Linking Intro**:
1. Compiler: Build starts with source files and compiler compiles them to object files.
    * Translation unit: preprocessor add all includes in a source file
    * Object file: compiled translation unit
        * container of different sections (ie .text .data)
            * .data BBS?
2. Linker:
    * Pull together different sections from object files and concat them together
    * put all .text/.data from all object files together
3. Loader:
    * Load the different sections into memory as segments with the correct permission
        * .data: r/w
        * .text: r/x

**Shared library function call**:
* normal: code call a function in its own binary
* shared library: map shared library to its own address space and perform function call into the shared library.
    * shared library is able to call other shared library

Linking text relocation:
* Purpose call a function from another object file/shared libary:
* In the `.code` section it calls an arbitrary address 
    * ie `call 0x00000` at line `0x1000`
* In the `.reloc` (like a TODO for linker) it states "find `foo` and write its address at `0x1000`"
* Disadvantages:
    * Modifying the `.code` segment makes the section unshareable between **processes**
    * Relocation need to be done once per call-site vs once per function

Global Offset Table (GOT linux):
* Calling a function with an memory address of a placeholder instead of an address to be overwritten
    * ie `call [0x2000]`: will call the function at address `[0x2000]`
* `.reloc`: only need to write the address of the function in `0x2000` instead of every call site.
* Disadvantage:
    * could incur extra indirectin

## Linux import section

`.dynamic`:
    * a list of dynamic library names it depends on

`.dynsym`: symbol table
    * contains all symbols that the binary contains
    * The ones to be imported will be marked with `UND`

Semantically linux binary tells the loader:
1. Here are all the libraries I would like to load with the `.dynamic` section, I want to map to the process
2. Here are the bucket of symbols I want you to locate in any of the loaded libraries.

## Interposition

**Interposition**: overriding a symbol in one binary (executable / library) from another

**Library Search Order**:
* Linux: breadth first
* Loader will start from executable then the libraries the executable and then the libraries the first order libraries depend on.
* If a 3rd degree library uses `foo` and defines `foo`, the loader will start with the executable before that library
* Anything before the current library have a chance to interpose `foo`
* Search in current lib before bread first search (`--Bsymbolic`)
* `LD_PRELOAD`: load the libraries in the env var before the libraries in the executable.
    * Loader will still search executable first then the libraries in `LD_PRELOAD`

**Can a shared-library symbol be overridden from an executable?**
* Linux: yes

C++: `new`
* allows default implementation of `operator new` new to be overridden 

**Symbol Resolution Time**:
* For executable: undefined symbols are not allowed by the linker.
    * linker will find all libraries that one of them have the symbol? (isn't that the loader)
* For libraries: undefined symbols are allowed because the symbols might be in the library/executable that imports it
    * This is because of linux breadth first search
    * `--allow-shib-undefined`

**Process-wide singleton**:
* Linux: just put the singleton in the executable

**Circular library dependencies**:
* Linux: yes
* Shared libraries allow undefined symbols at link time

Weak vs Strong symbols:
* **strong symbols**:
    * can override weak symbols of the same name
    * if two strong symbols of the same name, linker will choose the first symbol (interposition)
        * allow overriding of malloc
* **weak symbols**:
    * does not need definition
    * usually are declaration

## Position Independent Code

What is not PIC:
* Hard coded function address is not PIC: if the binary is loaded in another address the call will not work
    * `call 0x789`
* Instruction pointer relative call: call a function that is an offset from the current instruction pointer (`rip`)
    * `call rip-12`
    * Not interposition: loader cannot hijack the address of the function to call
    * Used for `hidden symbols`
* GOT: Indirection: call is made to a offset of a table of function address
    * If the binary is move to another address, the `.got` needs to be updated with the new function address
        * `.got` is not pic
    * The code segment is still PIC as the offset into the `.got` is the same
    * Do each binary have its own `.got`?
    * Uses the `.got` table
    * Indirection might occur in shared library:
        * compiler don't know if the function definition in the shared library will be interposed

Building a PIC executable:
* Executable symbols cannot be interposed
* `-fpie`: to help when executable are located at a different memory address.

## Lazy Binding

* Resolution only done at the first time
    * yes
    * Intervened with `-no-plt`

First call to an unresolved symbol:
1. calls `f() => call procedure_lookup_stub_42`
    * `42` is an arbitrary identifier given to `f`
2. `procedure_lookup_stub_42: jmp [got_slot_42]`
    * `got_slot_42: procedure_lookup_stub_42 + 1`
    * `got_slot_42` points to the next address of this address
        * jumps back
3. jump backs to the next address of `jmp [got_slot_42]`
    * `push 42; jmp <ldr resolver>`
    * calls the resolver to resolve the `f()` symbol
4. resolver resolves the symbol and overwrites the `got_slot` with the address of `f` found
5. After the first call `jmp [got_slot_42]` it will got to `f` directly

`plt`: the table of the all the `procedure_lookup_stub`

Actual function calls are into `plt` slots
* `plt` slot is per binary

Downside:
* security possibility as the GOT is re-writable


## Function Pointers

* The binary that defines the symbol will have the address of the symbol in `.dynsym`
    * The address of the symbols is the offset into the `plt`
* Binary that imports the symbols from other binary will load the address from `.dynsym`

## Symbol Visibility

* `.dynsym`: contains all (**global**) symbol and is always visible to all other binaries
    * all global symbols are potentially exported
* Symbols can be marked `UNDEFINED`
    * needs to be "imported" by the loader from other binaries
* symbols have visibility: `default` / `protected` / `hidden`
    * `hidden`: program instruction relative code is `hidden`
        * not on the `.got` and other binary don't know how to call them
    * `default`: symbols are in `.got`

