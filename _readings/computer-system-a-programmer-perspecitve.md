---
title: "Computer System A Programmer Perspective"
toc: true
---

## 7 Linking

Purpose: combining various pieces of code and data into a single file that can be loaded (copied) into memory and executed

Happens at:
* compile time: source code translated to machine code
* load time: program loaded into memory
* runtime: by application

### 7.1 Compiler Drivers

Compiler driver: invokes preprocessor, compiler, assembler, and linker

Steps:
1. Driver runs preprocessor translate source file into ASCII intermediate file (`main.i`)
2. Driver runs compiler translates intermeidate file (`main.i`) into an ASCII assembly language file (`main.s`)
3. Driver runs the assembler (`as`) translate the `main.s` into a binary relocatale object file (`main.o`)
4. Driver runs the linker (`ld`) to combine multiple object file to create the binary executable
5. Shell invoke the excutes the executable the binary through a `loader`
    * loader copies the code and data of the executable into the prog

### 7.2 Static Linking

Static linking: takes in a collection of relocatable object files and generate a fully linked executable object file

Relocatable object files:
* contians code and data sections
* instructions are one section
* initialised global variables are another section
* unitialised variables are in another

Linker tasks:
1. Symbol resolution:
    * Object file **defines** and **references** symbols
    * coresponds to:
        1. function
        2. global variable
        3. static variable
    * purpose: associate each reference with a **exactly one** definition
2. Relocation:
    * compiler / assembler generate code and data sections that start at address 0
    * linker **relocates** these sections by giving a memory location to each symbol defintion => modify all references to the symbol
    * relocation is performed blindly by the linker by **relocation entries** generate from assembler

Object files are collection of blocks of bytes (different sections) => linker concats the blocks and decide on the run-time location of the blocks

### 7.3 Object Files

Types of object file:
* **Relocatable object file**: contains binary code that can be combined with other relocatable object files
* **Executable object file**: binary code that can be copied into memory and executed
* **shared object file**: relocatable object file that can be loaded into memory and linked dynamically at load time and run time
    * diffrence: relocatable object file is at compile time

Compiler and assembler crate relocatable object files and linker generates executable files.

### 7.4 Relocatable Object Files

Format:
* Starts with a 16 byte sequence:
    1. word size
    2. byte ordering
    3. object file type
    4. machine type
    5. file offset of the section header table
        * section table header contain information on where to find the sections
        * the header for the section is at the end of all section instead of the start (normally)
* sections:
    1. `.text`: machine code of the compiled program
    2. `.rodata`: read only data (ie string literals, jump tables and switch statements)
    3. `.data`: **Initialised** global and static C varibale:
        * the actual bytes of the initialised data will be stored here
        * runtime local variables are stored in the stack and not `.data`
    4. `.bss`: unitilialised global and static variables
        * unitialised => no initialised to zero and we do not need to store N * zero bytes
        * occupies no actual space
    5. `.symtab`: symbol table
        * information of function and global variables that are **defined** and **referenced**
        * every relocatable object file has `.symbtab`
        * does not contain local variables
    6. `.rel.text`:
        * a list of location in the `.text` that need to be modified when the linker combines this object file with others
        * any instruction that calls an external function / global variable will need to modified
        * not needed for executable object files
    7. `.rel.data`:
        * any global variable that are referenced by the module
    8. `.debug`: debug symbol table
    9. `.line`: mapping between the original C source program and machine code

### 7.5 Symbols and Symbol Tables

**Symbols**:
1. _Global symbols_ denfined by module `m` and can be referenced by other modules
    * nonstatic C function and variables
2. _Global symbols_ that are referenced by module m
    * nonstatic C function and variables
3. _Local symbols_ defined and referenced exclusively by module m
    * use `static` attribute
    * different from local non-static variables! those are not "exported"

**Symbol tables**:
```
typedef struct { 
    int name; /* String table offset */ 
    char type:4, /* Function or data (4 bits) */
         binding:4; /* Local or global (4 bits) */
    char reserved; /* Unused */
    short section; /* Section header index */
    long value; /* Section offset or absolute address */
    long size; /* Object size in bytes */ 
} Elf64_Symbol;
```
* `name`: byte offset into the string table for the symbol name
* `value`:
    * relocatable: the offset from the beginning of the section
    * executable: the absolute address
* each symbol is assigned to some section of the object file (denoted by `section`)
* pseudosection section that symbols can be part of but not in the actual elf
    * `ABS`: symbol that should not be relocated
    * `UNDEF`: undefined symbols that are referenced in this object module
    * `COMMON`: unitialised data objects that are not yet allocated
        * COMMON vs BSS
            * COMMON: uninitiliased global variables
            * BSS: uninitialised static variables, global or static variables that are initialised to zero

### 7.6 Symbol Resolution

* Symbol resolution: associating each reference with exactly one symbol definition
* Local symbol resolution: compiler enforces onde definition => easy
* Global symbol resolution: compiler see undefined symbol => generate linker symbol table entry => linker handle

#### 7.6.1 How Linker Resolve Duplicate Symbol Names

* problem: Linker takes in a collection of relocatable object modules. Each of the modules might define multiple global symbols with the same name
* Compiler exports global symbols as either **STRONG** or **WEAK** and the assembler econdes these information in the symbol table
* strong symobls: functions and initialised global variables
* weak symbols: uninitialised global variables

Deduplication rules:
1. Multiple strong symbols with the same name => not allowed
2. Given a strong symbol and multiple weak symbols with the same name => choose strong symbol
3. Given multiple weak symbols with the same name => choose any weak symbols

deduplication does not care about the type!(relevant to C)

#### 7.6.2 Linking with Static Libraries

* releated functions can be compiled into separate object modules and packaged into a single library
* linker will only copy the object module that is referenced
    * this will reduce the size of executable on disk
* linux: static libraries are stored as *archive* on disk
    * archive is concated relocatable object files with a header

#### 7.6.3 How Linkers Use Static Libraries to Resolve References

* Linker scans the relocatable files and archives left to right
* Linker maintains
    * a set `E` of relocatable object files
    * a set `U` of unresolved symbols
    * a set `D` of symbols that have been defined in the previous input files
* algo:
    * for each input file
        * if `f` is an **object file**
            * adds `f` to `E`
            * update `U` (unresolved symbols) and `D` (resolved symbols) in `f` then proceed to the next
        * if `f` is an **archive**
            * the linker match the unresolved symbols in U against the symbols deinfed the archive
            * if some archive member `m` defined symbol that resolves a reference in `U`, `m` is added to `E`
                * archive members are added on demand (only if it defines symbols that is unresolved)
            * linker updates U and D to reflect the symbol definitions
            * any member not in `E` is discarded by the linker
    * if `U` is not empty at the end, throw an error

### 7.7 Relocation

Relocation:
* After symbol resolution step -> every reference is associated with one definition
* Merges the input modules and assigns **run-time address** to each symbol

Reolocation Steps:
* *Relocating sections and symbol definitions*: 
    * linker merges sections of the same type (ie input modules' `.data`s are merged)
    * each instruction and global variable has a unique runtime memory address
* *Relocating symbol references*:
    * linker modifies every symbol reference so that it points to the correct run-time address

#### 7.7.1 Relocation Entries

Problem:
* Assembler does not know where the code and data will be stored
* Assembler does not know the location of externally defined functions / variables

Solution:
* When assembler encounters a reference to an object whose loction is unknown
* Assmebler generates *relocation entry* that tells the linker how to modify the reference when it get merged into object file
* Relocation entries stored in `.rel.text` `.rel.data`

Relocation entry:
* `offset`: offset in the section which is a reference and will need to be modified
* `symbol`: symbol that the modified reference should point to
* `type`: how to modify the reference

Relocation types (32 total)
* `R_X86_64_PC32`: relocates 32-bit PC relative address
    * PC relative address: an offset from the current run-time value of the program counter
        * TODO(klementtan): look at section 3.6.3
        * CPU excutes an instruction using PC relative address
        * we do not know the relative PC until all the input modules have been given a unique address
        * effective address: PC relative + current runtime value of the PC (address of the next instruction)
```
refptr = s + r.offset /* ptr to reference to be relocatd */
refaddr = ADDR(s) + r.offset /* ref's run-time address */
*refptr = (ADDR(r.symbol) + r.addend - refaddr)
```
    * `addend`: is used bias the modified value
* `R_X86_64_32`: a reference that uses 32-bit absolute address

**Relocating PC-Relative References**:

assembly
```asm
e: e8 00 00 00 00   callq 13      // sum()
                    a: R_X86_64_PC32 sum-0x4
```

relocation fields
```
r.offset = 0xf // the address to oververwrite - one after e8
r.symbol = sum
r.type = R_X86_64_PC32 r.addend = -4
```

suppose the following:
```
ADDR(s) = ADDR(.txt) = 0x4004d0 // starting address of text segment
ADDR(r.symbol) = ADDR(sum) = 0x4004e8 // address of sum function
```

* `callq` instruction is at offset `0xe` (relative)
* followed by 32-bit placeholder for 32-bit PC relative reference
    * to be replaced by the linker with the actual address of the function

linker first compute the run-time address of the symbol reference to overwrite - the placeholder address for `callq` address
```
refaddr = ADDR(s) + r.offset
        = 0x4004d0 + 0xf
        = 0x4004df
```

linker then find the address to replace the place holder. The PC relative address of sum. Relative to the next instruction (`refaddr`)
```
*refptr = ADDR(r.symbol) - refaddr + r.addend
        = 0x4004e8 - 4 - 0x4004df
        = 0x5
```

**Relocating Absolute References**: straight forward

### 7.8 Executable Object Files

* very similar to relocatable object files
* `.init` section defines a small function to initialise

### 7.9 Loading Executable Object Files

* Loader copies the code and data in the executable object file from disk into memory
* setups the different memory segments 
    - user stack grow from top down
    - kernel stack grow from top up
    - heap grow from bottom up
* calls the `_start` function => `__libc_start_main` => user `main`

### 7.10 Dynamic Linking with Shared Libraries

* disadvantage of static libraries:
    * any updates to the library requires a rebuild / relink
    * code duplication - every process' code is copied to the text.
    * If a host with a lot of process with the same function, the code will be duplicated in the text segment for all the process.

shared libraries:
* goals:
    * exactly one `.so` for a particular library - do not need the duplicate object files
    * a single copy of shared library `.text` section in memory - shared by multiple proceses
* compile with `-fpic` position independent code and `-shared` to create a shared object file
* none of the shared object code is copied into the executable - linker copies some relocation and symbol table information that will allow the references to the shared object be resolved
* running an executable linked to shared object and partially loads the executable sections into memory
    - sees that there is `.interp` section - path the dynamic linker
    - runs the dynamic linker:
        - relocates (if not already relocated by other process?) `text` and `data` of `libc.so` into memory segment
        - relocates `text` and `data` of `sharedlibrary.so` into memory segment
        - relocating **any references** (resolve reference) in executable defined by the shared libraries.

### 7.11 Loading and Linking Shared Libraries from Application

* applications are able to request the dynamic linker to load and link when the application is running (after load time)
* application don't need to link it at compile time
    * don't need `-l`
    * allow for lazy loading of shared libraries
    * the references of the shared libraries will still need to be added at compile time
    * use `-rdynamic` to add all symbols to the dynamic symbol table - resolved at load time
* run time loading of shared libraries:
    - dlopen: load the shared library
    - dlsym: get the symbol address from the loaded shared library

### 7.12 Position Independent Code (PIC)

* To allow a single shared library to be loaded for all process, the address positioin in which the shared library is loaded cannot affect all the processes.
* Position Independent Code (PIC): code that can be loaded without needing any relocations
* References to symbols in the same executable don't need PIC

#### PIC Data References

* dereferences the address directly
* Compiler generate PIC reference (references are PIC) by exploiting:
    * DATA segment and CODE segment have a constant distance
    * Distance between both sections will always be the same regardless of the absolute address
* GOT
    * Compiler create a *global offset table (GOT)* at the begining of the data segment
    * 8-byte entry for each global data (procedure or global variable) referenced by the object module
        * utlimately contains the absolute address of the global data
    * Compiler generate a relocation record for each GOT entry. At load time, the dynamic linker writes the correct address to the entry (used by reference)
        * additional indirection

#### PIC Function Calls

* use `callq` on the instruction directly
    * allow more indirection
* problem:
    * cannot generate reolocation record for each reference - no PIC as the code segment is modified
        * (klementtan) is modifying PLT also not PIC?
    * generating relocation for each function in the shared library can take very long.
* lazy binding: 
    * address of the shared library resolved at the first call
* Procedural Linkage Table (PLT):
    * stores the instruction to the GOT entry with the absolute address and a fallback if the absolute address has not been resolved
* lazy resolution steps:
    1. Instead of calling `addvec` directly will call the corresponding PLT entry for that symbol (ie `callq PLT[i]`)
    2. The first PLT instruction does an indirect jump to the corresponding `GOT[i]` for that symbol.
    3. Since it is the first time calling `GOT[i]`, `GOT[i]` back to the next address in `PLT[i] + 1`.
    4. The next instruction in the `PLT[i]` pushes the id of the GOT to modify and jumps to `PLT[0]` (invoke the dynamic linker)
    5. resolve the symbol: `PLT[0]` pushes the argument for dynamic linker and invokes the dynamic linker to resolve the symbol
    6. `GOT[i]` will be overwritten with the correct value - next call to the `PLT[i]` will jump to `GOT[i]` with the correct address.


### 7.13 Interpositioning

Compile time:
* `-I.` to include your header that overwrites all call to a function to your call
* `--wrap f`: overwrites calls to `f` to `__wrap_f` instead

Link Time: `LD_PRE_LOAD` load your shared library before anything else
    
