---
title: "Programming from the Ground up"
excerpt: "Notes for Programming from Ground up book"
toc: true
---

Title: Programming from the Ground up

Author: Jonathan Bartlett

## Chapter 2: Computer Architecture

Modern computer architecture is based on *Von Neumann* architecture where the computer is made up of the **memory** and **CPU**.

### Structure of Computer Memory

The computer memory is made up of a sequence fixed-size storage and each storage has a unique address for it.
* A memory address refers to a single byte storage in main memory.
* Aside from program data, normal programs also reside in memory
* Number attached to each storage is called **address** (aka Pointer)
* The size of each storage is a **byte**: on x86 a byte is number between 0 and 255

### CPU

**Main responsibility**: read instructions from memory one at a time and execute them.

**Fetch-execute** cycle:
* *Program Counter*:
  * Tell the CPU where to fetch the next instruction from. Holds memory address of the next instruction.
  * Instructions are stored in binary form (numbers)
* *Instruction decoder*:
  * Decode the binary form of instruction to actual instruction (add/sub/mult/...)
  * Computer instruction includes both the actual instruction and a list of memory involved.
* *Data bus*:
  * From the list of memory address, fetch the actual memory that will be used.
  * Actual wire that connects the CPU and memory
* *Registers*:
  * High speed memory locations (store bytes)
  * Types of registers:
    * General Purpose: perform main action (add/sub/mult/...)
    * Special purpose: TODO
  * Registers will be used to store temporary data during computation
  * Due to lack of registers, data is brought from main memory into registers, CPU will use the data in registers to perform computation and once done, the data is put back into main memory.
* Arithmetic and Logic Unit (ALU):
  * where instruction is actually executed -> result of execution will be written to register or main memory.

**Computer word size**: the size of a register (on `x86_64` => 8 bytes word size)
* An address is also word size => store an address in the register

Note:
* A CPU cannot tell the difference between a number, address, character. What type of data it is depends on the context => are you treating the data as a memory address?
* Anything variable length in a struct is stored else where and the struct will have a pointer to it.

### Data Addressing Methods

1. *Immediate Mode*:
  * Data to access is in the instruction itself (skip dereferencing an address)
2. *Register addressing mode*:
  * Instruction contains a register that stores the data.
  * Treat another register as a memory storage.
3. *Direct addressing mode*:
  * Instruction contains a memory address that stores the data
4. *Indexed addressing mode*:
  * Instruction contains a **memory address** and an **index register** that stores the offset of an address.
  * a multiplier can be specified in the instruction to state the multiplier to the offset
  * ie: memory address in instruction is `2002` and the index register has `3` and the multiplier is `4` then the memory address is `2002 + 3 * 4  = 2014`
5. *Indirect address mode*:
  * Instruction **contains a register** that **contains a memory address**
6. *Base pointer address mode*:
  * Instruction **contains a register** that **contains a memory address** and the instruction can include a **offset** to add to the memory address in the register memory address value.

## Chapter 3: Your First Programs

<iframe width="800px" height="800px" src="https://godbolt.org/e#g:!((g:!((h:codeEditor,i:(filename:'1',fontScale:14,fontUsePx:'0',j:1,lang:assembly,selection:(endColumn:14,endLineNumber:30,positionColumn:14,positionLineNumber:30,selectionStartColumn:14,selectionStartLineNumber:30,startColumn:14,startLineNumber:30),source:'%23PURPOSE:+Simple+program+that+exits+and+returns+a%0A%23+++++++++status+code+back+to+the+Linux+kernel%0A%23%0A%23INPUT:++++none%0A%23%0A%23OUTPUT:++returns+a+status+code.+This+can+be+viewed%0A%23+++++++++by+typing%0A%23%0A%23+++++++++echo+$%3F%0A%23%0A%23+++++++++after+running+the+program%0A%23%0A%23VARIABLES:%0A%23+%25eax+holds+the+system+call+number%0A%23+%25ebx+holds+the+return+status%0A%23%0A.section+.data%0A.section+.text%0A.globl+_start%0A_start:%0Amovl+$1,+%25eax+%23+this+is+the+linux+kernel+command%0A++++++++++++++%23+number+(system+call)+for+exiting%0A++++++++++++++%23+a+program%0Amovl+$0,+%25ebx+%23+this+is+the+status+number+we+will%0A++++++++++++++%23+return+to+the+operating+system.%0A++++++++++++++%23+Change+this+around+and+it+will%0A++++++++++++++%23+return+different+things+to%0A++++++++++++++%23+echo+$%3F%0Aint+$0x80+++++%23+this+wakes+up+the+kernel+to+run%0A++++++++++++++%23+the+exit+command'),l:'5',n:'0',o:'Assembly+source+%231',t:'0')),k:100,l:'4',m:100,n:'0',o:'',s:0,t:'0')),version:4"></iframe>
* The source code is in *assembly language* - human readable form
* Use `as exit.s -o exit.o` to compile the assembly code into machine code - the output of this is known as an object file
  * Each assembly file will generate a single a single object file and we will need to link them into a single executable
* Use `ld exit.o -o exit` to link the object file into a executable

### Outline of an Assembly Language Program

**Comments**: anything that starts with `#`

**Assembler directives** or **pseudo-operations**: anything that starts with period (`.`).
* Instruction (line) that will not be directly directly translated to machine instruction.
* Used by the assembler itself

**Section** (`.section`):
* `.section`: breaks the program into sections
* `.section .data`: starts the data section - where you should list all the memory storage you will need for data.
  * klement: this should not include the stack and heap but instead the static duration storage? Stack and heap memory requirements are not constant at compile time?
* `.section .text`: the text section of a program - where the program instruction lives

**Global** (`.global`):
* `.global SYMBOL` Indicate to the assembler that it should not discard the symbol after assembly.
* `.global _start`: `_start` is a special symbol that marks the location of the start of the program
  * Needs `_start` so that the symbol will still be present after assembly (in object file)
  * Without `_start` being present, the computer will not know which instruction to start at
  * Without a `_start` symbol the linker will throw "ld: warning: cannot find entry symbol `_start`; defaulting to 0000000000401000"

**Symbols**:
* A variable/text that will be replaced by something else during assembly/linking
* *label*: **defines** the value of a symbol
* Syntax: `_start:` - a symbol followed by a colon
* When assembler is assembling a program, it will need to assign each data value and instruction (line) an address
  * Labels tell the assembler that the symbol value is where the data element or address will be (allow you to change position of the label)
  * klement: does this mean all symbols will be replaced with the address of the value and not the value itself?

`movl $1, %eax`
* transfers the number `1` into the `%eax` register
  * Instructions in assembly can have *operands* (arguments), `movl` has the source (first operand) and destination (second operand). Some operand might be hard coded (require the value of operand to be in a prefixed register).
  * `$` (dollar) sign in front means that immediate mode addressing is used (literal). This means the operand for the instruction is the value `1`
    * Without dollar sign it would be *direct addressing* => read the value at address `1`
  * Setting the register `%eax` to `1` value will make the kernel execute system call 1 (exit) when the program interrupt and hand over to the kernel

`movl $0, %ebx`
* `%ebx` register stores the exit code for the kernel to read.
* Loading value to `%ebx` does not do anything but with the system call 1 (exit), the kernel will read whatever value in that register to set the exit code.

`int $0x80`
* send an interrupt to the program => transfer control from program to Linux (kernel)
* `int` => interrupt.

**System call**:
* Operating system features (file io) are accessed through system calls
* System calls are invoked by issuing `int $0x80` and setting `%eax` with the system call number
* Some system call might require additional parameters which can be provided by setting certain registers (ie `%ebx`)

### Finding a Maximum Value

This section covers an assembly program that returns the larges number in a list as the program exit code.
* Use `0` to represent the end of the list

<iframe width="800px" height="800px" src="https://godbolt.org/e#z:OYLghAFBqRAWIDGB7AJgUwKKoJYBdkAnAGhxAgDMcAbdAOwEMBbdEAcgEY3iLk68Ayoga0QHACw8%2BeAKoBndAAUAHuwAM3AFZji1BnVAM5CpgCNqAT2ILaiPDj7l6qAMLJqAVyZ0QAZgBMxM4AMjh06AByXqbohCDigQAOyHL4DnRunt5%2BSSlpfKHhUWax8YE26HbpAngMhHiZXj4B1ui29nw1dXiFkdGlCda19Y3ZLXLDPWF9JXEJAJTWyB6EiKxsAKT%2BvooyAEqKAPICmCAA1BcAKnA4cmeJhMjAhMxnVHSod3hw6GdMDMocEwvGc6P1CGdkBQzgwNmoAIJbXwXFGoi4KPCQ6GoBi1M74dBMOQAOjhiO2ZKRADV4XsAJLwgBCwUwAnO11%2BhHQwFueFidzgDAAbr9vr9eNRqMgAO5hYBnDwKOQgSkUhFIs5bACs6FwZwAtGcABLuT5nMX4j7oZRY80/M44vEEphnGJys7W5jTVCq5Ha9CmG2G4J1YDoCYO3EMfF8l28DwfX2a/w6gEGs4uFZc/iRp2x31JjlvdxS2UGP6EogWM5S4QdOh3Oq/RW6lXqtXk5GOhgAfWdd0NKH4DDCX3tztzDGJZ3hZzU%2BLuLZ97eRBHNsSYYVxovt3YLCOJCiqfDOxL3CO7fdjyuTvg5ChhXLtv27McJcjJxKl5d8xAAbAA7MQviSP4YHEOIWrEABUFapIIEQfBv7gRwHD/n%2BxBqJSB5HvWp58soeCfsAUrmGcPYTN02HwhRkxtvCTDIEK1BnAAJGoxDJjqeporxfEaoxIpzpaa4WmEGA2lyPITLEZKCSxl79hAxD%2BrgEGLFx6BpreNbIAwqDPm8OCEBGpgWHytrngxTEsf6AKcf6gZ8c5FwaqkdBrHatwLoZVAmZizoOSmWk2rcZIuRFkWudshmmDgwBhhM1GUfUPZSsgiTnFFzluZMukZWSiBMIkLHsUFqY2tlaIaogPyIAA1uayBnAovw4NC0roBsmD%2BBsAAcWEAJyDUJNyYhazhkpovzpYkPbWvgZJhIgtnBTxVX8TFUr6aC1qYkKIgeF1CLyZOV7vspqk4Op5UhYVxWrTqga3dpG01cgxVNmcB2eOGU20C1kxpcgGUbSiGqaF4iRNfl0MxDydB0O67WGeE0rhWDb0xXwbVyHQ3W9QNGzDZicUJbJJ02Zp9maU5YMCUxO6/D9R0wmOM2huGREIpoxWA90wOg5jENQzDs2utyYRIwY1GzfNgJ4PRGqOaF7P83gipnCgGDFhCE0Ky1FgyS6wiSkm%2BgGfgMLUFy%2BnVoKav/ICwIumCsxyVTrFodTlXCzFHA%2Bfr%2BAQPMhtyKb1BLTm7HKETCJsIs1DsFq3A%2BGwGjEMg7DwsYhLmNWcjLKsvxIlwxCK%2BnCeLPVYhqBxSdsOI3BMLXHFpxnWdsNwyocRXGjzIsIomek8RAA%3D%3D%3D"></iframe>

`data_items`
* For this program we will need to store the list of number in a memory
* Use `data_items:` (label) that refers to the memory location in the next line => the memory location of the first
element in the list
  * `data_items` == address of `3`
* `.long X,Y,...` tells the assembler to reserve `4` bytes for each element in the list
  * In this example, the assembler will reserve `14 * 4 = 56` bytes and initialize the memory storage with the provided data.
* Last element of the list `0` to indicate the end of the list (we need some sort of way to signal that the list has ended.)
* Do not need to mark the `data_items` label as the symbol is not needed after the assembly is done => only for convenience

Variable:
* In assembly, a variable is a dedicated storage (can be assigned to a register) for specific purpose
* For this problem, we need a variable to store for:
  * `%ebx`: the current maximum number
  * `%edi`: the index of the current element in the list 
  * `%eax`: the value of the current element.


**start**:

line 23 `movl $0, %edi`:
* Initialized the current element index to `0`

line 24 `movl data_items(,%edi,4), %eax`:
* `movl`: move(aka copy) a long (4 bytes) from source to destination
* `data_items` is the address of the first element in the list
* `%edi` will be `0` at this point => move the element at `data_items + 0` => first element
* `4` is the multiplier to the offset

line 25 `movl %eax, %ebx`:
* set the first element to be the largest

**start_loop**

Responsibility in the loop:
1. Check if the current value is `0`, if so break from the loop
2. Load the next value in the list
3. If the next value is larger than the previous max => update the max value variable
4. Return back to the start of the loop

line 29,30 `cmpl $0, %eax` => `je loop_exit`:
* `cmpl`: compares both `0` and `eax` and store the result in `%eflags` (hardcoded into the instruction - cannot be specified)
  * This comparison is the three way comparison `<=>`
  * `%eflags` is the status register
* `je end_loop`: jump to the `end_loop` label if the result `%eflags` is compared equal
  * Other conditional jumps `jg` (if second is greater)...

line 31, 32 `incl %edi` => `movl data_items(,%edi,4), %eax`
* `incl`: increment the value at register `%edi`
* `movl`: copy the values at `data_items + %edi*4` into `%eax` => `%eax` will have the new value that `%edi` points to

line 33, 34 `cmpl %ebx, %eax` => `jle start_loop`
* `cmpl`: three-way compare `%ebx``%eax`
* `jle start_loop`: if the current element (`%eax`) is smaller than the current max (`%ebx`) move back to the start of the loop
  * Immediately move to start of loop as it does not need to perform anything if the current element is smaller

line 36, 37 `movl %eax, %ebx` => `jmp start_loop`
* `movl`: at this point we know that the current element is the maximum => copy the current element to the current maximum register
* `jmp start_loop`: jump back to the start of the loop

**end_loop**

line 42, 43 `movl $1, %eax`
* `movl`: setting `%eax` to indicate to the kernel that we would like to exit (`1`). Program end and the current element value does not matter anymore
  * `%ebx` already populated with the current maximum => exit status will be this
* `int 0x80`: wake up the kernel

### Addressing Modes

General form of memory address format:

```asm
ADDRESS_OR_OFFSET(%BASE_OR_OFFSET,%INDEX,MULTIPLIER)
```
* `ADDRESS_OR_OFFSET` and `MULTIPLIER` must be constants (labels to constants)
* The other 2 must be registers
* All the fields are optional

Final Address:
```asm
FINAL_ADDRESS = ADDRESS_OR_OFFSET + % BASE_OR_OFFSET + MULTIPLIER * %INDEX
```

#### Direct addressing mode

`movl ADDRESS, %eax`
* Having a constant/label as one of the operand means that it is treated as an absolute address
* This example will load whatever value (long) into `%eax`

#### Indexed addressing mode

`movl string_start(,%ecx,1), %eax`
* Get the memory address from the `ADDRESS_OR_OFFSET` part and move it by `%INDEX` multiply by `MULTIPLIER` bytes.
* As the name suggest it is similar to how we access elements in an array
* This example will load whatever that is at `string_start + 1 * %ecx`

#### Indirect addressing mode

`movl (%eax), %ebx`
* Load whatever value that is in the address stored in `%eax` into `%ebx`

#### Base pointer addressing mode

`movl 4(%eax), %ebx`
* Get the memory address from the register `%eax` and adds it by `ADDRESS_OR_OFFSET`
* Treats `ADDRESS_OR_OFFSET` as the offset as the address is derived from `%eax`
* This example will load whatever that is at `%eax + 4`

#### Immediate mode

`movl $12, %eax`
* Loads a literal into the register
* Use a dollar (`$`) sign in front of the literal, without it direct address mode will be used

#### Register addressing modde

`movl %eax,...`
* treats the register as a memory address

Note: every mode can be used as source and destination operand except for immediate mode. Immediate mode can only be used as source (cannot load a new value into a literal).

#### Storage size

```txt
  
______%eax_______
[   |   |%ah|%al]
        ___%ax___
```
* `movl` treats the storage size as a word long 
* `movb` treats the storage size as a byte
  * cannot be used with a full register (ie `%eax`) as it does not make sense to move a byte into a word storag
* To move 2 bytes to a register you will move to the 2 least significant bytes (`%ax`)
* To move 1 byte you can move the least significant byte in `%al` or the most significant byte in `%ah`
  * Note: `%ax` is the concat of `%ah` and `%al`
* loading a value into `%eax` will wipe the values in `ax`, `ah` and `al` and loading to either of the smaller address will corrupt `eax`


## Chapter 4: All About Functions

### How Functions Work

**Components of a function**:
* *function name*:
  * A symbol that represents the starting address of the function code - assembly label
* *function parameters*:
  * Data to be passed to the function
* *local variable*:
  * Data storage that the function but thrown away after the function returns
  * Not accessible by other functions
* *static variables* (function static variable):
  * Data storage that the function uses but **not** thrown away after the function returns
  * Static variable will be reused every time the function is called
* *global variable*:
  * Data storage that is managed outside of the function
* *return address*:
  * A function parameter that tells the function where to resume after returning.
  * Usually stores the address of the next instruction on the call site. 
    * Allow the function to know what should be executed after returning
  * `call` instruction will pass the return address to the function for you
  * `ret` instruction allows the flow to be passed back to where the function was called from
  * (klement: this is something new that I did not know previously)
* *return value*:
  * Transfer data from the function (callee) back to the caller

**Calling Convention**: is the way the implementation choose to store and handles the various components of a function.

### Assembly-Language Functions using the C Calling convention

**Stack**:
* a region of memory
* Stack is at the very top of address (highest address)
* Grow downwards: pushing into the stack will decrease the top of the stack address
* `%esp`: register that stores the address of the top of the stack
* Assembly provides `pushl` (push long) and `popl` (pop long) to push/pop 1 word size of data on/off the stack
  * `%esp` will automatically get subtracted (push) and added (pop)
  * Each take one operand - register to push/pop
* Accessing the stack 
  * indirect: `movl (%esp), %eax` - access the word that is at the top of the stack
  * base pointer addressing mode: `movl 4(%esp), %eax` - access the word that is 4 ahead of the top of stack

#### Step 1 - prepare params (caller):
* Caller will push all parameters onto the stack in reverse order (last parameter lowest)
* Caller execute `call` instruction:
  1. Push the return address (address of next instruction on caller) onto the stack
  2. Modifies `%eip` (instruction pointer) to point to the start of the callee function
    * Allow the machine to know which instruction to execute next
* View of stack right after `call` is called:
  ```txt
  Parameter #N
  ...
  Parameter 2
  Parameter 1
  Return Address <-- (%esp)
  ```

#### Step 2 - base pointer (callee)

* Control flow is on the callee
* Base pointer `%ebp`: a special register used for accessing the function parameters and local variables (current stack frame)
  * Cannot use `esp` at it could change when we are trying to call other functions in the callee which will push the args onto the stack => `esp` change
  * We can treat `%ebp` as a constant whenever the control flow is in the callee function
* Parts:
  1. **Save the current base pointer (`%ebp`)**:
    * At this point the base pointer (`%ebp`) points to the base of caller stack.
    * The callee will push the data in `%ebp` onto the stack - `pushl %ebp`
    * This will allow the `%ebp` to return to the caller's stack frame when the callee returns
  2. **Copy stack pointer to the base pointer**:
    * At this point the top of the stack is where the `%ebp` should be - by C convention
    * Top of stack stores the caller's `%ebp`
    * Update the `%ebp` to the current (callee) stack frame - `movl %esp, %ebp`
      * This will allow us to easily access the current stack using base address mode
* The view of the stack:
  ```txt
  Parameter #N   <-- N*4+4(%ebp)
  ...
  Parameter 2    <-- 12(%ebp)
  Parameter 1    <-- 8(%ebp)
  Return Address <-- 4(%ebp)
  Old %ebp       <-- (%esp) and (%ebp)
  ```

#### Step 3 - local variables (callee)

* Control flow is on the callee
* **Reserve space** right above the base pointer for local variables
  * `subl $8, %esp` - if local variables are 8 bytes
* Local variables reserved at the "bottom" (right above ebp) vs `pushl` when the variable should be initialized 
  * We could call other function between variable initializations => could result stack being (local variable, params, local variable) which will result in the params being unable to be poped when the callee returns
* When function return, the memory after the ebp will be removed => local variable (life time as long as the function) 
* View of the stack:
  ```txt
  Parameter #N     <-- N*4+4(%ebp)
  ...
  Parameter 2      <-- 12(%ebp)
  Parameter 1      <-- 8(%ebp)
  Return Address   <-- 4(%ebp)
  Old %ebp         <-- (%esp)
  Local Variable 1 <-- -4(%ebp)
  Local Variable 2 <-- -8(%ebp) and (%esp)
  ```
* Note: static and global variables are stored in the data segment.

#### Step 4 - returning from a function (callee)

1. Stores it's return value in `%eax`
  * (klement: what if we want to return more than 1 word?)
2. Rests the stack to what it was right when the control is passed to the callee
  * Stack only has callee params at the top of the stack
  * Parts:
    1. Pop all local variables.
      * By reseting `%esp` to `%ebp` (stack pointer will be pushed to before the local variables)
      * `movl %ebp, %esp`
    2. Restore `%ebp` to the top of old `%ebp` on the stack and pop.
      * At this point the top of the stack is the `old %ebp`
      * Restore the `%ebp` to old `%ebp` - `popl %ebp`
3. Return control back to the caller.
  * At this point the stack only params and caller return address:
    ```txt
    Parameter #N
    ...
    Parameter 2
    Parameter 1
    Return Address <-- (%esp)
    ```
  * Done by calling `ret` instruction. Pop the top of stack (return address) to `%eip` => allow the control to be passed back to the caller.

Instructions for proper return:
```asm
movl %ebp, %esp
popl %ebp
ret
```

#### Step 5 - after function return (caller)

* Control now is on the caller
* Caller can examine the return value by looking at the `%eax` register
* Calling code needs to pop off all of the parameters/move stack pointer back to before parameters
* Destruction of register:
  * To save the current registers values, the caller will need to push the registers value on to the stack before.
  * `%eax` is guaranteed to be overwritten with the return value
  * Except for `%ebp` which will be the same (point to the same stack frame).

### Pow Function Example

Problem: program to calculate $$2^3 + 5^2$$

<iframe width="800px" height="800px" src="https://godbolt.org/e#z:OYLghAFBqRAWIDGB7AJgUwKKoJYBdkAnAGhxAgDMcAbdAOwEMBbdEAcgEY3iLk68Ayoga0QHACw8%2BeAKoBndAAUAHuwAM3AFZji1BnVAM5CpgCNqAT2ILaiPDj7l6qAMLJqAVyZ0dzgDI4dOgAcl6m6IQgABwATMQADshy%2BA50bp7eOonJ9nwBQaFmEdFxNuh2qQJ4DIR46V4%2BHNbotrl0VTV4%2BSFhxbHW1bX1mU1yg12BPUWRsQCU1sgehIisbACkMQDMANSKMgBKigDyApggu4TIwITM2wTbNJ5jN3jo23DIAO7bFB50FXw5NtPkQANZrNQAQQ2OwAKnAcED4pdrrdPo9tigmPEPK87nA3gA3EQeN7ICgQ6FbbYxNYAVkwOw2ACFtnT6ZhaVCYZSYdtMISIhY8AiDA86Pi3kwGIFtsirjcmA8gWMiOhUOLtoR0MBEa9CHJiLzqXJkJLtqgGNVtgoARLUMh0HI6GtOWsomo1gBOL14d4MQXbfTC0XAAB0lLDtra2zDluqvKhUfKMbDr2UeETkLDwGoyHM2wA%2BmNOlni%2BMQJScXI4NRtgASHbbZst1vbGHVuA28p8DU1YBeeh%2BiDoZSJIL8WZVjw1uv1mJtxftradn44A1%2B/uD/jbCCmIxvOi9QhTqHCah1xKfCJLlsw891kVvX7/NqUhioVBzqLEZd0p3xEuMJMMggZPja1SIKCcrIIE%2BrbPuUHbAAtFq6AgYGyLoISDgznKNTMOg%2BpyNOs5/ugDDKLezYwnIAZvOBVAbkGdByNehC7qq2oarKGz/pRswIegvDapS1HicumwPoEwDmkEGY/H8dqkbWDYLhJHYzl2tq9kGhADiwO4jmOfBDqekKdnOdISZJq5MWMekGUOu77go2xHtM5kPjB7HifeIiPgSimvqk76ft%2Bv58QBQFbBhDFBSWSGJHBN6IdBqHanFcrajhixIgRLDEVWyDxHWUWmFRflbPCbw6XQfasb5iJBtQ2ofhYYk2TFmy8TE/HKGG2wAOq1fRGpPp1XVtjC9mbo1N7SGa4GJaCRpQlNi60WadBfMCbzCBKmgzn6iSAfgk0bTRWyLH6cFmuVyhhV%2B5GUZFfXoBV01bB%2B40EkqBDAERBKEBdl0wuB2pyB41C3UCvX/hVlIYXOTQvZVX2bKO%2BC7g9ypoXgSxBKg5lwQ2ajKB6WYwnshwnGc2zws1L52njM7qncZpYjirzGjs4HEp4ZIUEG7nHlqMoKONyC8yLV4RBG3JbLzACSwR7LC5wAGLrg5W6GX6qHga5h7HjLAg9vVjnbgb5pyxxBAyzciJvNjDuK5svNHDIsLq%2BcQ0YrqYFBZD0ObkCDD44T2wC6SvO88ERywpgAjnDVPk3kwx1CdsHDbEQ2zXBR%2Bpx%2B7ABqkL7MrkLMn4yeVu75GfahHzUKgQJGweovTDLUWIFRzfuG3ttfBEJdUjsyHiBA5XxIJA%2Bt%2B3QWIEs2o7iHMNj3yUWUazks/PnrzYkQNQWBBx%2BA2PaYWPEbx27%2BAACzNvlCdv15Cl2tnyLiYgFMmYnwgp%2BChRflpMq71TCAWorReiedW4IU7slfgo8oTI3InIeIb0EaQLvLFBgoJaqQWgog%2BCHc3LEOQZCKGBZ6ySDQdgz%2BWxAZ%2BkuMgJUIk85LG2Hmc8Z8bgX25Cg0CdYojT3AbPTBH1KqaT9LNK2%2BtNTb0eoIwkdYOAxFEVg%2BY5E%2B62VxN2FAls9bOXhuUJRkJUEPV/JPDRH1Z6SS4vtFezl1420CCkEQOAABe7N7jGy7uEEGAiLIj0IIWPMJVyydDfogbEKMJE6M2lsHAwtwJ2zxqja8ek3gOiCJSTQbxnCFjtkjIRKEp4zy0YoySWVwLL0IKvZhTpQ7inuIoykOBM4XkbsoCRO8cGbE6fYUqp9alOLXk0mGCEOrrQ/ldTYpCTbTHbDM2ZfIXF52FoMnApUcDCBjOiLp4RQYbT5A4niEo6oahKhEfQGobECRKSotGVjyliMErRAg2pzR1IaWhKGG8oQYEQGA/8CSoFbCBW1NyqSQl5OxOnUJ4T4iRNqDCQgfx972yCvJE6sKgmFNfo8us1iKm9KomioiUcY5vGAI6OGEo2nKJBbYiR6CMaQ0%2BfFAhDAkqwSQRxCAmVQJvG4SIaONQcAMHME6cyp1mUQI0lsDlapzR%2BPIRxQ2JUNkQR5dBZq2oCaEAlD9SGJEoQGspGweY1B2B0m4D4NgGhiDIHYJQkw5hT6miWCsSSXBiB4HUFa%2BYoIxBqDULodg4huBMFDeGh1TqXVsG4HIEA4aA2OqDcQQUBpUggHEEAA"></iframe>

* Main program:
  * Push the arguments (`base_number`, `exponent`) for `power` onto the stack - pushing the last argument first
  * Save the result of the first function (stored in `%eax`) onto the stack - no guarantee that data will remain the same for all registers (except for `%ebp`)
* Function (`power`):
  * `.type power,@function`: tells the linker that the symbol should be treated as a function - only relevant when linking > 1 object file.
  * `jmp power` vs `call power`: call will add the return address to the stack but `jmp` does not.
  * C calling convention (callee)
    * Code:
      ```asm
      pushl %ebp
      movl %esp, %ebp
      subl $4, %esp   # reserve space for local variable (current result)
      ```
    * Stack view:
      ```txt
      Exponent       <-- 12(%ebp)
      Power          <-- 8(%ebp)
      Return Address <-- 4(%ebp)
      Old %ebp       <-- (%ebp)
      Current result <-- -4(%ebp) and (%esp)
      ```
  * Why local variable (on stack) vs register:
    * When passing data to another function, you can pass an address on the stack but cannot pass an address to a register.
    * Not enough register to store all of the local variables.
  * `imull`: multiplies operand 1 and 2 but store the result in operand 2

### Recursive Factorial Function

<iframe width="800px" height="800px" src="https://godbolt.org/e#z:OYLghAFBqRAWIDGB7AJgUwKKoJYBdkAnAGhxAgDMcAbdAOwEMBbdEAcgEY3iLk68Ayoga0QHACw8%2BeAKoBndAAUAHuwAM3AFZji1BnVAM5CpgCNqAT2ILaiPDj7l6qAMLJqAVyZ0dzgDI4dOgAcl6m6IQ6AA7IcvgOdG6e3tGx8XwBQaFmETo26HYJAngMhHhJXj4c1ui29nzFpXiZIWG51XIlZRUpHV3Nga05kRwAlNbIHoSIrGwApABMAMyKMgBKigDyApgA1AC0uwDiOABu9LsMu3RtJLt4cDhyu1GEyMCEzLsoTFEeeOhng90HM1ABBRZLXYUBh2Ig4EQAOl2ADEiLt0MpmFFaMR7nB0NDYQRCAjqLtkBRQRDlrsoU86bsAFS7BbM3bVCmEXYANmRABUCUS4aSRBSqeDIbtxLsGTKWVCWWyWZz0QtJJc6KhdnJkBS6IjqZCjctBQzXu9PkwdXBkAB3Z62u33PXCajkq4UDx0Qp8XaEApTOLnSyG8GIhS%2Bui7RGoBglI2S02PZ4Wj5fOBGa564DUZCmMVxhPhyP1aOIgHKPDUxG5/PkgD6nSaNbr5mFJLJu27Pd7UoeDIZ3qC6Aw2u9tGMuzthLt%2BjwLptpRB4N7a/XG77tIHzy9PrLlyYfGAFOB3LTVrk1Kb/RA1L%2Bcjg5IAJDLN%2B%2BpYLCTCRV2SgBrQF9UJUpgC8egF0OYFqXfWCNylG5hmnWd50uDt4TFSlkQEZA8XwGC4MI7spWAdA8FTDxH1Hak3XJH9OzFOCpUIb18W/YkMLo70o2pBhUFQF8NUWABWQEoiIqUUQ40UuP3BJvmQX5aABbVDiEFjTCBIUolKZgyIiXYCKIpjt0zBc5woqjtT9YEdRKRB/2pI9TnJET0AYZQ8Tc0xlBMqF6M4/0yKmOgtJA0KZ25QJDIWUSPLxUx/iM4yP1pGdp1Q/BZWjbzfIIHVnFlBcs0mblMXw1cUtgqVmzwSinOQFzdmfTk3I8iTaVotjdkAwggmoOZMAWOYAA41DmABOCbnnKhc9x48FAgXZ81GUMbExpJYzWeBlbOJDwxXmg8MCoOh0joGs8AsKJ2N/ERiAAASOhJqQCmS73BB8nxi0TTHElKapKLVSm1Z6/U6DwKAoA5kN2TNzhdZKqvXZjARJQlvPE/L8FTUl0Vqwlwl4AMkeRrcoQDOq%2BsCYA8V1WH4cJfKvqKhqmrcuQoi82L0D%2Bvzdm22VnnCYRKNnQlUD4QbhvWqbzNQ/Kj1wCgLFJsniNMwlmwcl5kCWiI6b1dKxZ%2B3mojDMFnPJEaIEx8ZTfa%2BDaUF5ygNsqhCE6S5CDAlh%2BEXNrlDV9WpXEW2eb%2B0Y4fcVAwqCqnoz41AA2MPF9FQYOyalG27ej6hY%2B6j2vZ0q19MIGilOa1qecd/mAElodsxDwii55OQeeMhYpKZdgLBRM%2BRqVhAUNOtVhuIlIseOQt2CAOCFgeqqlEQAz4qfosDy448pmfThEDx0FGalNEJZwGzeslqQwRBXJr3yl9pZAzztJ50Dxa/V4Ubq988FcwS%2B2%2BcV77GSlCzLKxNu7ci6vlC%2BIgaIiDotJLs/MuqwIGuCK2uwc4RyiPbXKm4pSb0zNvYKfVdg/wPobZCi8Aa0gDHmPikCXi6RYACKK/A9S5WpDgJgHh3Smx8tzIB1VaS8OoPYHEU8O4LlMFIoUKc%2BELkpGxGhIDaR6C9tAvUaDZ4bzvkfSq6s1z9iFPoOQkUu6dCIKObKDtPLTkeIgOAqiOr0meMAZAaAdSBBmPieM0tRrjSms8O0BIAzTz6i4wiUoKFAQ8RbM%2BaCPqW0aoAs2QixL82bMDQgoNuIHh3mQiGUMYYznvMgHEAj/q0KhIzRcKd0ZVM1NqDmWMjZhL/kYlGmsp6RUJugYmTMhRg2jM2Mo1FwSUy6RrCmpDoz5Xdvk%2BSEAdy6yiGFKJ/NCnRliXiAgyBRhsHGNQdgwluA%2BDYBoYgyB2BgmMOgMwlgdSlV8ZCLgxA8DqCOeMf8Yg1BqF0OwcQ3AmB/IBRcq5Ny2DcDkCAAFnzLnfOIOcT2CQQDiCAA%3D%3D%3D"></iframe>

**Main Function**

```asm
_start:
 pushl $4
 call factorial
```
* Push the value `4` onto the stack and call `factorial`

```asm
addl $4, %esp
movl %eax, %ebx
movl $1, %eax
int $0x80
```
* At this point the function `factorial` has return (all recursive call completed)
* `addl $4, %esp`: move the stack pointer back by `4` bytes (1 word) to clean up the `factorial` parameters
  * Should always clean up stack after a function returns
* `movl %eax, %ebx`: move the return value from factorial to the register that will be used for exit code

**Factorial Function**

```asm
.type factorial,@function
factorial:
```
* `.type` directive tells the linker that the `factorial` symbol is a function

```asm
pushl %ebp
movl %esp, %ebp
```
* Creates a stack frame and save the caller stack frame
* Standard C calling convention - should be executed in all instructions

```asm
movl 8(%ebp), %eax
```
* Stores the argument passed by the caller (at address `%ebp + 8`) in `%eax` register

```asm
cmpl $1, %eax
je end_factorial
```
* `%eax` now stores `n`
* Check for base case when `n == 1`, if true then move to `end_factorial` routine


```asm
decl %eax
```
* At this point `%eax` will not equal to `1`
* decrease `%eax` (`n`) by one so that it can be passed to the next recursive call

```asm
pushl %eax
call factorial
```
* Add `%eax` as an argument for the next `factorial` call and call the function

```asm
movl 8(%ebp), %ebx
```
* After the inner call to `factorial`, the register that stores `n` (`%eax`) will be over written by the inner call
* Copy back the original parameter (`n`) in `%ebx`

```asm
imull %ebx, %eax
```
* Multiply `n` (`%ebx`) with the result of the recursive call `%eax` and store in `%eax`


```asm
end_factorial:
 movl %ebp, %esp
 popl %ebp
 ret
```
* Return from function by resetting the stack pointer to base pointer (local variables will be cleaned up)
* Reset to the caller's base pointer and return to the next instruction in the caller.

## Chapter 5: Dealing wit Files

### UNIX File Concept

Overview: when we want access a file, the operating system will give us a *"number"* (**file descriptor**) which we will use to perform any file operation. Once the we are done , the file will be closed and the file descriptor will be useless.

Using asm to perform file operations:
1. File open:
    * To open a file, the program will need to execute a `open` system call, provide the **filename** and the number to represent the open **mode**
    * `%eax` will hold the system call number `5` (open file)
    * `%ebx` will contain the address of the first character in the file name (`const char*`)
    * `%ecx` will contain the file access mode (read/write)
    * `%edx` store the permission set
2. Linux will return the file descriptor in `%eax`
3. Perform read/write (system call) on the file:
    * Read is system call `3` => move `$3` into `%eax`
    * Place the file descriptor from \#2 into `%ebx`
    * Address of the read/write buffer into `%ecx`
    * The size of the buffer in `%edx`
    * OS will return the number of bytes read/written in `%eax`
4. Close the file by executing the system call `6` (move `$6` into `%eax`)

### Buffers and .bss

* Continuous block of bytes used for bulk data transfer
* Using data segment as a buffer:
    * Data segment is static data that is initialized using the source code
        * Source will specifically specify the value of each individual byte in the
        data segment of the code
        * ie `.byte 0 0 0 0`
    * Will require the users to type in the individual elements in the source code (redudant as we do not care what the initial value of the buffer)
    * The size of the data section directly translate to the size of executable
        * Executable will contain the same bytes as what the data segement should be initialized to
        * (klement: can imagine the OS performing a memcpy from the executable into data segment)
* `.bss`:
    * Similar to the `data` segment
    * Does not take up space in the executable
    * Specificy the size of the memory storage but unable to specific the initial value (uninitialized)
        * The memory location is initialized to `0`
    * The buffer can be initialized/modified during runtime but not at initialization of the program
    ```asm
    .section .bss
     .lcomm my_buffer, 500
    ```
    * `.lcomm` will create a symbol `my_buffer` that points to a 500 byte storage location.

### Capitalizing all characters in a file

Overview: a program that takes a path to a file and capitalize all the characters in it
1. A function that takes a block of memory and converts it to upper-case
    * Takes the address of the block of memory and size
2. Repeatedly read from into the buffer (read system call might not read all the bytes of a file)
3. Open and close the necessary files

<iframe width="800px" height="800px" src="https://godbolt.org/e?hideEditorToolbars=true#z:OYLghAFBqRAWIDGB7AJgUwKKoJYBdkAnAGhxAgDMcAbdAOwEMBbdEAcgEY3iLk68Ayoga0QHACw8%2BeAKoBndAAUAHuwAM3AFZji1BnVAM5CpgCNqAT2ILaiPDj7l6qAMLJqAVyZ0QAJmLOADI4dOgAcl6m6IR%2BAMySAA7IcvgOdG6e3n7ESSn2fMGhEWbRcZI26HZpAngMhHgZXj7%2BFVV8NXV4heGRpb7x1rX1jVktQ10hPSUx/eIAlNbIHoSIrGwApP2KMgBKigDyApggANQAKnA4cicJhMjAhMwnKHQAbtF41/onIQkeeCcqLR1moAIKbWInAgnb5LPB/AFA9AnADu%2BDgMOo1BOtDweGichB4P6zz473q6FQUOQJw8CQS0WECgAdESIWytjt9i5MAIBABJMIAcVOHDmJ32DLoULgyN%2B/0BNHQHMhvnFkvoMuRcIRiuBYIhJ3mJwA6pdaKjlZhfOsABxqdYATkdhGRdGQAIYALwspOzhOyAoWp%2BdF1SJVMPFroYVISnQDQaRIehS0IJxYTCIFhOpg8FAo0QjpnFwBpPruHmAGPQDEQGNMFnxCfT6EzhAsEZwQZ9yIbTauMJxyBR0QAtEzkbj8SQIy9yQD8NTafTGUZlQaSYhxSjCPhkT2W23s7n89ElzqFeGDWCTuz%2BoaACKgs6gk4CSr5OhEk7MhRtaXMqgXoMCqd6xGBYEuPsYQCC%2BYRnAIEH3sh4Eoey16vhCcgWHI%2BJMM8IjYnQvSEISN7MugACOHhvgAmgIAD6%2ByKJgYTECcACs34UdRdGMSaOz8mcmDseI3FUTRAj0QxOyYKCD7seB5ESXxDEuIEhwiScABs4m8VJjGYAAGkJ7EcGyN4QsgCSftcvBptZ9B6TR%2BwyQ%2B0GBLR7EOspvGuS4snPgxAkebRDFnDsMhhC43mxBwajmRht79Lh%2BhAYQVJJhgciILuNlEGRr48ZJZwPoK3nOW%2BpX7DIZxmZVsEPpgOw7OxNpJVhOF4QRWLJtEhB0nglWBIKMhGQxBkuKCgSBN5yj2hZRUqaxD5MQAYgxa38oEWlqMlsQXAOA4Hq6eDLNKrwiB466vicd33Q9j2PVZQbRlSKKXHWLb6NcI7rNadoOs67zfk9YNgxClzer6/qBsGV5LbxYQyAAsgAQs1DGgjsQqo6xCFtYtd1IZCaN8m%2BH5pCclBELSdAhKkIg4AAXpSOZ5gWpFzNxf6fj%2BpjGN%2BEJoxzZ6jjKR2/bKrrBkBtQ/Nc1DIDGbMhAQQskhQdz4QecsMHqyJpaiu54pqWvIEwGuQmrZa%2BheiJKsy5yXNcchwEs1CoFbJyhOSfrKKsbMcNpxBqGHgK05du5LIV%2B0nNGch8HIrK%2BTRaMyGta2YwKABaWkcWH3HUCgTD4enmeY0%2BL7seXWc7BN/J53HLjRk2%2BsnpzzYF3tSsGNcRMk%2Bc6DKAC77/tTtZnSIpIYNzYK/pTfA/viI9W7BoIuAA0icBwCmc/LQYhqdVQ3ecybyzUAGpaba92GhcbokTmjboNcoSUqr0oHqliAANYR2mJWwhsRRxwAwcwN0fwqVgptVa5UTijnEODOOD8bjJFSEvOGB55QOwtNlXKOB8ppggDuAEmwOLoFMAkOeiMSqwKYrVdio5b4QxJKg3IGDpRYLtv8MMSoTj4LygQYhpDkoUKoTQqB%2BkzhYxxjFE4e1kFxzqMAV4JxLqeGRCQ%2BoYjKHUIajI7GQpL4MTUKJZBhowjMG1EGW49xHj4W0WQ3w4j9HHxgUYkxHB2IsNYZCfkoZLz8MYCwamojyF6MkcVE%2BniGL%2BBOBwXw4NDT7F4UEi0IStHhJcZEomzJgBK3MCcBiqV6hEhKeMEAVswKCiEvyaajdMA7y5EKHYoIUYkytnIBg7xgw/3/kkNWhYbyZleNiCJcgEjsQiVQuOuUaxNgPJsXwTAGAhGWScfpOY1xoKGYQAe/RQRYmQMIJskzazInsgGZYBsBGvwIUQ2OVkv6%2Bn6d%2BOQHgikABIYG50wOfI4Oxr7TJyZMuZrdkTnNWAA4MQDp5ZXuUIyOdQwEQMKo5OgDEkRyFOIaPQURsQfU1MA2g70aDYmHpUf4vYPQYgxVi1ADEQgwjoFSXCRB0AMqZV%2BDC9KKCMpCFUyyaF%2BjMVYicQU2wzgnC2jtTpwrYgYs2ThEl35RnYh%2BdJMVbFdEMGUFbHBBtMnJhpDM/VIzkBjJiTjLxEAZnUJBeI5QcdRn7l9DGVArpjDNmwYE3BbprEmt0aYZ1H0cBfTRL1KItIFBUgbMGX%2B0RQjUCtm9UcfBLCAj0MANVlqNWuR2O5MInlHWVHNZhfoPoByoGQK/L8AMFrOgBNGLE2ZVmmzTFct6IQc0WqtV8tQ2kh2ltQOW/aJKTiFA8GOtWJwvkjWRuNSa01AiLXZa6Llgquk9LdciYAOB3jSgRTlJFDkXmQtqH/XNVqIl6vYjAtacCwh2pyRIxafLGVwiFRW1Cv7tUSlqlKmV21MDyp/Uqg8CN0x5rnQZJiLEdW3rHVZNJ/rjU22DWO9V1rjFxJfa4hYmGXWWt3bs/gZ5oQHntrc9D/BTWvudVcg8ibCDJqthQbNdlaY7lSAYJckGlTXvzWpQKMiQrFrChFKK8iImIGQ/0TMGAYWhBRLciAXYfhDQbUDR01x5lekpJI7DA6h0h10aOq2E6p0zv4HOhdY0Jr0SmjNNdwjOX8oYl%2Brpbn4b8MEYQ4RJxpaQOw0h%2B9MjH0MLOPh3JCqfZemWIRbMpZtl/yXKs5lStrKLRJhjIUgoTgo1BAVjSzEwNvQYllhIDEoj7p8NUlCgUHwSrCCcV8aMNJbxlVyFG5wAASTTJW1WA3KkVSlXzGbg010teqrbAHQNDOUfrbn%2BaIUJk%2BkXBQxYkaWkNVsDxwr5tCN6Jr1tfNrpXZ8oJS1yf268lmNjgwd2GRNmD52M511PlpCJFmFUCAe82Z7aYTsDlOudT%2BuqbMAnnaNJdTmV0HN/RCYyQkJVrVNKB7TTpHTXxOIFFwA3mtnAGycFaYHx2yjS1c2GiZ%2BGrMICx78iAmAJA1Stdam0QMzfk7EDTvAPCsoDGev0hA7gkBOClyjMNWXfk0BaZwlXkDZYwi8ewxFOUVaq9%2B/akFoLXx2NK4nTSOvcm3mcfYJwZCKBYjsE4U0jjk7%2BG7DVF365Vx/YdqmcMgffid3AcZOS9X7RSKzH1vofc3gnXOD4DECAMTpAyfZSUkis8h/tebi3NkA9MLWf%2B4tXSuqpAeW46BXgOA8F8QgwAvD0E%2BN%2BD1GrJC6LBRCL1PmJkJAa7%2BgSQkmlG5OCbrrNVDcW/78PoDsrQNja6QD7hvZRZJ9ezewPygR3OsNK6vpAPoQ8fxGduDPfhLc/Y/w6EFeQswYfatYf22HWYatp7zB3Zw8L7O67hi7ubtQ7s7DxzAhnOrodRjZQTwSjR94k6laKDk6aAs7xw1iMpVY1boB1ZxzuiECrKErmiThK6d4YQK5a6gQoTqSaT9ZNKT6ITT4KphD7DCQIKWgCJ8D/Q2iNqOgAjvxF40g1oi5i7PCU6/w9rexLw9gKDsRRDCDn7cG0wvC4C2Tew1r1rME6YAgpDAD0wUDZj6CNiXB8aTKVBgLYjBb77STEFHDH59rYhX5Ra36EZmrfizow6Lp/4AFGGMQmHfYr7raWFbb2o2EMZ2G2YOEObLouY3i3gKpgQo5nDk6TbSSRFmFL4apmL343j2H2Zw7/4I5AGxDbB7CaSnCHR2QC7jyTweBJYMGvxajezR6kRUxn4rhpgTgwrtxAK/zVIKpDZnD5G%2BhUCkQAhxgOILZnjHS%2BiP48o/ph69gtHNgZhZhLjVFDRxaoJ/h8Cxh1DWLTgKywr0DAA%2BgJjew%2BhejsyniL464Krj61RdEDgUBFF8wkaEC74VEHiIDLCuj8DexA6ojojBgJ5jhMhdg4BszkgpB8Apw/pWyXzYz1Ida8ja6GhIZ0G1YhD0x8be6v5xZmp0G0AGC7GonHHew/Y4B0HPGi615HGdyBgUAKALHjHkKIDYjizEmvEAh9i9joA9r%2BzMCTBexxaUA4C9E3DxhwyybKCSLoSYThFEGHxwQIQQQKqoJKwjhpimBLCsp1DZhwypibI1grBwDDT7AmjNT27/LXa3hY7OggRmmOhWyoI/FKkqnpTqlBiakKB1B1h6kGk7BGkMQ5zTKWnMxMGAzY5WxuBvAEhUxRB4Ajiai2kspUgKlngTiVRW425qR67NQCjQS%2BmKHY7giWl0EBksEWnZnOhd4QjrxdawQfbk7RIwLv47Q6q3wQiBDbE4lBgR60Inyu5mRJIQglHTztlhGgjzGx7IDx71GwkkguA9TUDsnzFpDsTdK9JPGETRBgBsDXA54KBkYbFCGvKXqtE3h%2B4B6uLrYd67a4HggSm/pHDSpW4nAQmCSgjQkUFI73jmGdkfbNTWHc5xyLmkZ2Jl4YBUgqI178DXDMonTIFXAbGeEyJ1msTfnJEJFzpJEEkGpBjtwL6fG7Gsx3A4gtkYgohGAS4HpOQKpn5yDsSaAV4Ai0A7pM4s6JG7ZjqaDIgK7DlVZxzyHIhErnpklDHXCtg2Qdgq5kgx4EEKoZ7BiMmkksnXqmDUxhYEnEBiiOp0nEZLnh4vwwjOIUJB4epeq/RfEEkJiUkLaxkJLpgeDUD2Cs4AlpgYa0kpqSVli2xujDzMnaUC60DeqLgDgRlRljE66WlFmBnOgWUFk6b%2BmWkMUJAKXzr6mGmghHBYxqXOWvhy4%2BweU1YvyxXxUaQelek%2BliJ0my7ABZUjw5V74Kq0rRBohbnzFPbaV1GJ6Jk3gerxXJmYwgH64ZmIYuLqWGiNUHbDgJk7J1hrF2BDF8Wuj7q4RnhOX8a%2BgxmNETWPBTUnG9mC7rpyjMm56DnQavAKVOXsQxZ3rKVii/k%2BY55pZPFiUUhUizXQVnjmw6y%2BhPXzUOV0ZPZomXmgihCVUsna4hCDU5K4B%2BKHXIiDLkZpjQgA2eXVWvjM6p7KVEYQiq4hDXR0w%2BWxxKJPQQh/SWm9LRh1hsw9igx413z9DOCy6hCkihn1CK7K6gjsX3V4BM0JATmxDuhwFnSsbqJXToBUU0X4X0Xvn2qlot4kht4crSUrkNEvGklbLQ0bFS7IgkroC9g7Iq0vZoIo2voJAQ0y0yzLlYhniblQ3IB7LfinSU1xwp4WWS5uXTlnjw0hi4QDT/hsALDUDsAcTcA%2BBsAaDEDIDsCgjGCtjmDZiJzLDQoQhcDEB4DqDe0LC/xiBhy6DsDiDcBMDp1mKB3B2h1sDcA4pmJJ1B0p3EBAlpAgDiBAA%3D%3D%3D"></iframe>

* Setup
    * `.equ` directive allow us to name literals (ie `$8`)
    * `STACK_POSITIONS`: store the offsets from base pointer as constants
* `convert_to_upper`:
    ```asm
    movl ST_BUFFER(%ebp), %eax 
    movl ST_BUFFER_LEN(%ebp), %ebx
    ```
    * In C calling convention, all the arguments are saved on the stack. Using the predefined constant %ebp offset, move the arguments into the registers
    ```asm
   cmpl $0, %ebx 
   je end_convert_loop
    ``````
    * Exit the loop if the provided buffer is `0` in size
    ```asm
    movb (%eax, %edi, 1), %cl
    ```
    * Using indexed indirect addressing mode, copy the character at `%eax` buffer with offset `%edi` and multiplier `1` => `%eax + 1*%edi`
* Opening files:
    * Command line arguments are pushed onto the stack before the first instruction
        * argv and argc will be stored
    * We will first save the current stack position in %ebp => used to easily access all the cli arguments
