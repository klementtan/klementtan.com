---
title: "Readings: Semi Static Branch"
toc: true
---

Title: SEMI - STATIC CONDITIONS IN LOW- LATENCY C++ FOR HIGH FREQUENCY TRADING : BETTER THAN BRANCH PREDICTION HINTS

**Static Branch**:
* The branch predictor will always take a certain branch. If the branch evaluates to `false`, the state will be rolled back and the instruction pointer will move to the other branch

Pipelining Hazards:
* Structural: sequential instructions are using the same hardware resource
* Data Hazards: Instructions depends on each other, out of order execution solves it
* Control Hazards: branch predictions

Solutions to Control Hazards:
* Pipeline stall - do not pipeline any instruction during a branch => slow
* Static Branch - always take a branch
* Branch delay slots - perform unrelated instruction during the evaluation of the condition => always jump to the correct branch wihtout any rollback

Dynamic Branch Prediction:
* One Level Branch Prediction:
    * One-dimensional prediction buffer
    * Buffer cached by the instruction pointer
    * Buffer stores a single bit that indicates the last branch taken
    * Disadvantage: limited historical information => frequent branch misprediction
    * Mitigating: instead of a single bit, a two-bit or integer counter can be used that increments or decrement the counter.
    * 0,1,2,3 is the level of likelyhood
* Two Level Branch prediction
    * Instead of using the branch instruction address as a index of the branch, the last (*N branches taken*, *instruction address*) is used as the key to prediction out come
    * This will allow the pattern of last `N` prediction to be used to determine the outcome
    * Steps:
        * A branch history register (BHR), index by instruction address stores the last `N` branches taken for the instruction
        * The value of the BHR and the instruction is used to index into PHR => a two-bit to show how likely a branch is

Branch Prediction Hints:
* Does not affect hardware based branch prediction
* Reorders the positions of instructions to help static branch prediction (how?) and instruction caceh - hot path code near each other.
* Compiler will pre-fetch the instruction before it is time to execute, ordering the instruction will allow the correct likely instruction to be fetched first
    * Modern processors assume the jump equal / jump not equal are take
* This optimisation if the branch prediciton unit (BPU) has not visited the branch before
    
