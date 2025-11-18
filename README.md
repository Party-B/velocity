# VELOCITY DESIGN README 

## 1. Vision and Constraints

### 1.1 Purpose

Velocity, or v was my 'opinionated' programming language idea. In reality, aren't all newer languages just opinionated programming languages? The ethos behind the language (and the reason I would say it is just an opionated language) is that I would come across several aspects of other langauges that I would love, which felt like they really added value and efficiency - so why not combine them into one approach.

The purpose of the language is like velocity itslef - speed with direction. I love high-level languages that focus on giving you a platform for being as efficient and quick in designing and implmenting ideas. However, I  disliked the fact that it sacrificed some things to get there. Speed with direction is conceptually about achieving that efficiency but without the losses. Is that possible? Proabably not. You could probably just say that I've leveraged certain gains here to losses there and so on. Hence, opinionated programming language.

### 1.2 Target Use Cases

Being conceptually about speed and efficiency might evoke the idea that I'm moving away from low-level language concepts. Quite the opposite. ISAs and languages have continued to develop on older models, like strapping a jet engine to a horse and cart (but keeping the horse). I wanted to reimagine what was possible with keeping things low-level (memory management, less abstraction and bare metal operations) but extrapolating high-level features. Potentially this just creates pasta level jank.

In short, the target use case is whatever you'd imagine for Rust or C. Small footprints, blistering speed (hence the name [yeah I know velocity could be close zero and still considered as such]). But overall, the target use case in my mind was high level development with low-level function.

### 1.3 Explicit Non-Goals

### 1.4 Design Priorities

Prioritising in this order:

1. Efficiency of movement (hit less keys but get more);
2. Speed (speed of development, second to speed of execution);
3. Efficiency in size (small binaries);

## 2. Computational Model

### 2.1 Execution Model

### 2.2 Memory Model

### 2.3 Concurrency Model

### 2.4 Error Model

## 3. Syntax

### 3.1 Lexical Structure

### 3.2 Grammar Overview

### 3.3 Expression Constructs

### 3.4 Statement Constructs

Here is the meat of what I think will make this language worthwhile (for me at least).

**With statements**
    I am planning for V to be an OOP language, but even if that weren't the case, the value of with seems to outweigh the reason it wasn't in C.

    With object
        .method1
        .method2
        .publicVar = setVar
    End With

**Bang methods**
    I learnt a bit of ruby, bang methods are great. I guess the argument against it would be that it is at odds with the C ethos given that it already allows mutation via pointers. This is a syntactic sugar to some degree.

    object.variable = function(object.variable)

    Seems slow to me when we can just:

    function(object.variable)!

    This really was just trying following the fact that I love C's variable++/-- and I was having to write in VBA variable = variable + 1.

### 3.5 Module / System Syntax

## 4. Types

I am a self-taught programmer by way of Microsoft VBA macros. I love option explicit. I love strong typing. I know I said speed of development was priority one - I think strong typing enforces this. Yeah, you write a little more at first, but the compiler saves you a lot in the long run. Majority of this stuff is going to be a lift of C types pared to a VBA set of ideas - but minus bools. We going to the bitfields with this one (though in saying that, I think the compiler can handle bools for us - we'll see where we end up).

### 4.1 Primitive Types

### 4.2 Composite Types

### 4.3 Reference and Pointer Semantics

### 4.4 Mutability Rules

### 4.5 Type Inference Rules

### 4.6 Compile-Time Evaluation

## 5. Semantics

### 5.1 Variable Lifetime Rules

### 5.2 Name Resolution

### 5.3 Function Semantics

### 5.4 Memory Safety Rules

### 5.5 Undefined Behavior Strategy

## 6. Standard Library Scope

### 6.1 Core Data Structures

### 6.2 Unsafe / Low-Level Interfaces

### 6.3 I/O

### 6.4 Concurrency Tools

## 7. Intermediate Representation (IR)

### 7.1 IR Specification

### 7.2 Transformation Passes

### 7.3 Optimization Goals

## 8. Virtual Machine / Runtime

### 8.1 VM Architecture

### 8.2 Instruction Set Design

### 8.3 Memory Management Strategy

### 8.4 Safety / Isolation Features

### 8.5 Debugging Hooks

## 9. Compiler Architecture

### 9.1 Front-End

### 9.2 IR Generation

### 9.3 Optimization Pipeline

### 9.4 Backend (Bytecode or Native)

### 9.5 Build System and Tooling

## 10. Linking and Packaging

### 10.1 Binary Format

### 10.2 Module Linking

### 10.3 Versioning

### 10.4 FFI Strategy

## 11. Testing and Verification

### 11.1 Testing Strategy

### 11.2 Reference Interpreter Tests

### 11.3 Conformance Suite

### 11.4 Fuzzing and Security Tests

## 12. Example Programs as Design Tests

### 12.1 Minimal Program

### 12.2 Pointer / Memory Stress Tests

### 12.3 Concurrency Stress Tests

### 12.4 Library Design Experiments

### 12.5 Prototype Use Cases

## 13. Open Problems and Unresolved Questions

### 13.1 Syntax Decisions Pending

### 13.2 Semantic Ambiguities

### 13.3 VM Trade-Offs

### 13.4 Future Feature Ideas

