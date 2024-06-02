---
title: "FPGA PROTOTYPING BY VERILOG EXAMPLES"
toc: true
---

Title: FPGA PROTOTYPING BY VERILOG EXAMPLES

Author: Pong P. Chu

## Chapter 1: GATE-LEVEL COMBINATIONAL CIRCUIT

**Verilog**

* Verilog: hardware description language (HDL)
* Looks like C but the semantics is concurrent hardware operation
* Verilog is non-deterministic

### 1.2 1-bit equality

1-bit equalitty: return true if both inputs are `0` or `1`

Operator with just `not`, `or`, `and`, `xor` (basic logic gates)

C++
```cpp
bool eq(bool a, bool b) {
    return (a && b) || (!a && !b);
}
```

logic gates

$$eq = i0 \cdot i1 + i0' \cdot i1' $$

Verilog Code

```sv
module eq1
    // I/O ports
    (
        input wire i0, i1,
        output wire eq
    );

    // signal declaration
    wire p0, p1;

    // body
    // sum of product terms
    assign eq = p0 | p1;

    // product terms
    assign p0 = ~i0 & ~i1;
    assign p1 = i0 & i1;
endmodule
```

HDL breakdown:
* I/O port portion:
    * describes the input and output **ports** of the circuit
    * Q(klement): are ports always used for input and output
* Signal declaration:
    * specifies the internal connecting signals
    * describes the connecting signals between different circuit
* body:
    * describes the internal organization of the circuit

1-equality example:
* three continuous assignment constituite the three circuit parts
    * Q(klement): what is a circuit? is a gate a circuit
    * Q(klement): what is a circuit part?
* connection between the parts are implicity defined by the signal and port names
    * assignment uses the ports (I/O) and signals

### 1.3 Basic Lexical Elements

**Lexical Elements**
* **Identifier**: gives a unique name to an object (ie `eq1`, `eq2`)
    * `$` is usually used for system task or function
* **Keywords**: predefined identifer to describe language constructs (ie `module`, `wire`)
* **White space**: use for improving readability but does not have any impact
* **Comments**: `//`

### 1.4 Data Types

**Four-value system**
* `0`: logic 0 - false
* `1`: logic 1 - true
* `z`: high - impedance state
    * output of a tri-state buffer
    * Q(klement): what does this mean?
* `x`: unknown value
    * usedto present uninitialized input or output conflict

#### 1.4.2 Data type groups

**Net group**
* Represent the physical connections between hardware components
* Used as the output of continuous assignment
    * continous assignment hardware components
* `wire`
    * most common data type
* one-dimensional array:
    * a collection of singal grouped into a bus
    * example:
        ```sv
        wire [7:0] data1, data2; // 8-bit data
        wire [31:0] addr;        // 32-bit addr
        wire [0:7] revers_data   // ascending index
        ```
    * ascending index should be avoided - it should correspond to the MSB binary number where the MSB is on the left
* two-dimensional array:
    * 32-by-4 memory (ie a memory has 32 words and each word is 4 bits wide)
    ```sv
    wire [3:0] mem1 [31:0];
    ```
* other data types: `wand` (for wired-and-connection) `supply0` (circuit ground connection)

**Variable group**
* represent abstract storage
* outputs of procedural assignment
* Datatypes: `reg`, `integer`, `real`, `time`, `realtime`

#### 1.4.3 Number representation

General format for integer constants

```sv
[sign][size]'[base][value]
```

`[base]`: base of the number
* `b` or `B`: binary
* `o` or `O`: octal
* `h` or `H`: hexadecimal
* `d` or `D`: decimal

`[value]`: value of the number in the corresponding base
`[size]`: the number of bits in a number
* optional: if set the number is a **sized number** or **unsized number otherwise**
* sized number: zeros are padded in front to extend the number if it is smaller than the size
* unsized number: size depends on the host machine

#### 1.4.4 Operators

bitwise operators: `~`, `&`, `|` or `^`

#### 1.5.1 Port declaration

I/O declaration specifies the `modes`, `data types` and `names`

```sv
module [modules_name]
(
    [mode] [data_type] [port_names]
)
```
* `[mode]`: `input`, `output`, `inout`
* `[data_type]`: `wire`

#### 1.5.2 Program Body

* A verilog module (function/subsystem) is a collection of **circuit part**
    * the collection of circuit can be executed in parallel
* **circruit part** can be described as:
    * **Continuous assigment**:
        * simple combinational circuits
        * `assign [signal_name] = expression`
        * left-handside: is the output of the circuit part
        * right-hanside: is the input of the circuit part
        * example: `assign eq = p0 | p1`
            * When `p0` or `p1` changes value, the statement is activated and expression is evaluated
            * new value is assigned to `eq` after the propogation delay
        * Each assignment corresponds to a circuit part and the order of the assignment does not matter
    * **always block**:
        * more abstract procedural assignment (order maters?)
        * for more complex circuits
    * **module instantiation**:
        * use predesigned modules as subsystem of the current behaviour

#### 1.5.3 Signal declaration

* Declaration: specifies the internal signals and parameters used in the modules 
* Interconnecting wires between the circuit parts

```sv
wire p0, p1;
```

**Implicit net**:
* if an identifier is not declared, it is defaulted to `wire`

### 1.6 Structural Description

Allow for a larger system to bebuilt with multiple predesigned components (`module instantiation`)

```sv
module eq2
    (
    input wire[1:0] a,b,
    output wire aeqb
    );

    // internal signal declaration
    wire e0, e1;

    // body
    // instantiate two 1-bit comparators
    eq1 eq_bit0_unit (.i0(a[0]), .i1(b[9]), .eq(e0))
    eq1 eq_bit1_unit (.eq(e1), .i0(a[1]), .i1(b[1]))
    assign aeqb = e0 & e1;
```

module instantiation statement
```sv
[moldule_name] [instance_name]
(
    .[port_name]([signal_name]),
    .[port_name]([signal_name]),
    ...
);
```

* `[module_name]`: the name of the module to instantiate, in this case it is `eq1`
* `[instance_name]`: unique id for the instance of the module we instantiated
* `.[port_name]([signal_name])`: 
    * connection by name  - similar to keyword args
    * `.[port_name]`: the name of the I/O port of the module
    * `.[signal_name]`: the external signal name of the current module 
    * ordering does not matter

**Connection by ordered list**:
```sv
eq1 eq_bit0_unit (a[0], b[0], e1);
```
* same as connection by name but based on position of args - bug prone

**Verilog primitive**:
verilog provides some primitive modules that can be used

```sv
module eq1_primitive
(
    input wire i0, i1,
    output wire eq
);

wire i0_n, i1_n, p0, p1;
not unit1 (i0_n, i0); // i0_n = ~i0
not unit2 (i1_n, i1); // i1_n = ~i1
and unit3 (p0, i0_n, i1_n); // p0 = i0_n & i1_n
and unit4 (p1, i0, i1); // p1 = i0 & i1;
or unit5 (eq, p0, p1); // eq = p0 | p1;
```

### 1.7 TESTBENCH

To test the module we can simulate in a host computer (how?) and verify the correctness

testbench can be created that instantiate the module we would like to test with a set of test vectors

```sv
'timescale 1ns/10ps

module eq2_testbench;
    reg [1:0] test_in0, test_in1;
    wire test_out;
    eq2 uut (.a(test_in0), .b(test_in1), .aeqb(test_out))
    initial
    begin
        // test vector 1
        test_in0 = 2'b00;
        test_in1 = 2'b00;
        # 200;
        test_in0 = 2'b01;
        test_in1 = 2'b00;
        # 200;
        ...
        $stop;
    end
endmodule
```
* `'timescale`: `1ns / 10ps`: time unit is `1ns` and each step is `10ps`
* the module we would like to test is instantiated `uut`
* `initial block` is a verilog construct that is executed once when simulation starts
* each statment in the block are executed sequentially
* `test_in{0,1} = XXXX`: values of the signals (input)
* `#200` means the values will last for 200 timeunit
* `$stop` stops the simulation and returns the control to simulation software
* to monitor the result, on the simulator's waveform



Questions:
* Difference between procedural assignment and continous assignment
* What is synthesis

## Chapter 2: Overview of FPGA and EDA Software

### FPGA

**Overview of a general FPGA device**:
* Field Programmable Gate Array (FPGA): a two-dimensional array of generic **logic cells** and **programmable switches**
    * (klement): You can think of it as a 2D matrix and the edges containing the nodes are programmable switches
* logic cell: can be configured (ie programmed) to perform a simple function
* programmable switch: can be customized to provide interconnec tion among the logic cells
* a custom design can be implemented by specifying the function of each logic cell and setting the connection of each programmable switch
* A simple adapter can be used to dowload (in the field) the logic info the FPGA device instead of fabrication.

**LUT-based logic cell**
* Look up table (LUT) based logic cells with $$n$$ inputs, is a $$2^n$$-by-$$1$$ memory where it enumerates through the all the permutations of the input and store the output in the table
* It is able to implement any combinatorial function with `n`-input
* D-type flip-flop (DFF): output of the LUT can be used directly or stord to the D FF
    * take a clock as an input
    * at the rising edge of clock the output will show the input
    * use as a way to store data

**Marco cell**:
* special cells that are fabricated at the transitor level with special functionallity

#### Overview of Xilinx Spartan-3 device

**Logic cell**:
* four-input LUT and a D FF
* contains a carry circuit to implement arithmetic
* multiplexing circuit
* can be used as a 16-by-1 static random access memory

**Slice**: * two logical cells are grouped to form a slice

**Configurable logic block (CLB)**: four slices are grouped to form a configurable logic block

**Macro cell**:
* combinatorial multiplier: accept two 18-bit numbers and calculates the product
* RAM: 18K-bit synchronous SRAM (what does synchronous mean here?)
* Digital clock manager
* IO block (IOB): controls flow of data between devices' I/O pin

### 2.4 Development Flow

1. Design the system and create the HDL file.
2. Develop testbench in HDL.
    * perform RTL simulation
    * RTL: register transfer level (what does this mean?)
3. synthesis and implementation:
    * Synthesis process:
        * Known as logic synthesis - transform HDL construct to generic gate-level compoenents
        * changing code to logic gates
    * Implementation process:
        * translate process:
            * merges multiple design files into a single netlist
            * (klement): what is a netlist?
        * map process:
            * maps generic gates in the netlist to FPGA's logic cells and IOBs
        * placement and route:
            * derives the physical layout inside the FPGA chip
            * places the cells in physical locations
            * determines the routes of various signals
        * static timing anaylsis (in Xilinx):
            * determines the timing parameters (ie maximal propgation delay, maximal clock frequency)
            * last step of implementation
4. Generate and download the progamming file
    * fpga will be configured based on the programming file

Questions:
* unsynthesizable code? How does it look like
* what is time step?
* RTL vs HDL
* IOB
