# VELOCITY DESIGN README 

### Latest thoughts

So I've gotten really into rust, and have found that it largely meets a lot of the design principles I had in mind for a language. I'll continue learning rust and keep popping in here to make stream of conscience additions - after some time I'll have to reassess if pursuing this as a language construct actually yields any value.

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

Needs to support lazy evaluation, at least for `->oo` variadic parameters (3.4) - a running function like avg can't demand every argument up front since it doesn't know how many there'll be. Open question whether laziness is the default evaluation strategy across the language or something opted into specifically for variadic params.

### 2.2 Memory Model

### 2.3 Concurrency Model

Owes an answer to 3.4/13.2: read-only func methods only guarantee purity within a single call - whether the "same" object can produce different answers across two calls (another sub mutating it concurrently) depends entirely on what this section ends up saying about aliasing.

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

**func / sub separation**
    Lifting VBA's Function/Sub split and giving it teeth. `func` is the pure, functional world; `sub` is the dirty, imperative/OOP world where mutation, I/O, and calling other subs are allowed.

    func square x -> x * x
    sub print_squared x -> print(square(x))!

    The rule: a `func` body may only call other `func`s - never a `sub`, never anything that mutates non-local state or touches I/O. A `sub` body can call whatever it wants, `func` or `sub` alike. This isn't just a naming convention - the compiler walks the call graph from every `func` declaration and rejects it if anything impure is reachable. Bang methods (above) mutate by definition, so they're `sub`s only - never callable from a `func`.

    This gives the entry point a purity switch for free, without it being a special rule:

    func main -> ...   // the compiler checks the *entire* reachable call graph is func - the whole program must be pure
    sub main -> ...    // the traditional shape - main may freely mix func and sub beneath it

    `func main` isn't a separate check from the general func rule - main is just an ordinary func, and because it's the root of the whole program, requiring it to be pure transitively requires the whole thing to be pure. Where this earns its keep: it's a compile-time guarantee you can point a whole program at, e.g. for something meant to be deterministic/testable/embeddable, without needing a monad or effect-type system to express it - see 5.3 for the propagation rule and 13.2 for what "impure" actually has to mean for this to be checkable.

    Non-determinism counts as impure too, not just mutation/I/O - this was the open question, resolved by working through a concrete case:

    func Point.magnitude -> sqrt(self.x^2 + self.y^2)   // fine - pure field read, no external calls
    func Ticket.is_active -> self.date >= today()       // rejected - today() is non-deterministic

    Reading self is fine (that's just field access, nothing escapes). Calling today() is not, even though it doesn't mutate anything, because the result isn't determined by the arguments - call it twice and get two different answers. Three ways this could be handled, in order of preference:

    1. Compiler sees today() returns a Date and suggests pulling it out as a parameter: func is_active(date: Date, ticket: Ticket) -> date >= ticket.date. This isn't a downgrade - it's the standard functional-core/imperative-shell move. The func becomes genuinely pure (testable without mocking time, reusable for historical/future checks), and the one non-deterministic call (today()) gets pushed to whichever sub calls is_active and has to supply the date itself. The impurity doesn't disappear, it just becomes visible at exactly one call site instead of being hidden inside something that looks pure.
    2. If there's no clean single-value substitution, fall back to just telling the user: "this has a subroutine-style signature - if it needs a value that varies per-call outside its own arguments, it needs to be a sub."
    3. Rejected for now: "dirty funcs" that openly accept a sub as an argument (an explicit escape hatch). Doesn't work - if a func can take a sub as a parameter, its own purity becomes contingent on whatever the caller passes in, which defeats the point of the call-graph check being a static, unconditional guarantee. A func being generic over purity is a real thing other languages solve (effect polymorphism), but it's a much heavier mechanism than this is trying to be.

    Read-only `self` methods (does a method that only reads fields count as a func?) turn out not to need a special rule at all - `self` is just an implicit first parameter to the method, so the existing call-graph check already covers it: a method is a func iff nothing in its body ever writes through self (no field assignment, no bang call, no calling another method that itself needs write access). This is Rust's `&self` / `&mut self` split wearing func/sub vocabulary instead of borrow-checker vocabulary - func methods get an immutable view of self, sub/bang methods get a mutable one.

    What this doesn't give you: a read-only func method is only pure within its own call. If something else mutates the object between two calls (another sub, possibly on another thread), the "same" object can legitimately produce two different answers across calls. That's not a func/sub problem - it's aliasing/data-race freedom, and it's deliberately left to 2.3 (Concurrency Model) and 4.3 (Reference and Pointer Semantics), both still blank. func/sub purity is scoped to "does this call graph write anything," not "is this object safe from concurrent mutation" - trying to solve both at once is how you end up needing a full borrow checker before you can ship a working compiler.

    `constrained` - a modifier on func, not a third category alongside func/sub. Started from wanting a func you could "drop in and not worry about" - full confidence it does exactly what its signature says. That turned out to split into two separate guarantees with very different costs, see 5.3 for which one constrained actually claims.

**Loops**
    Here I was a bit unsure as to how I'd like to fill this out. I hated the fact that python doesn't have a VBA equivalent to one of the loops, I can't remember right now but it didn't have until I think? So definitely want that, but also, I started playing around with rust and I like the loop statement there. So I think I'll be aiming for:

    loop {...} // infinite loop without break - compile warning
    do {...} until x < y; // the condition is checked at the end of the code block
    until x < y {...} // the condition is checked at the start of the code block - semantics, just the !while.
    while x > y {...} // conditional looping evaluating at entry
    

**Was variables**
    I know you can declare 2 variables - 'x' and 'was_x' and then deal with it like that. But I wanted to combine it so that you can use more english natural language syntax. So, I'm planning to implement a one-deep buffer on variable types that are declared with a '?'
    The idea is that it will automatically store the one previous value to allow for syntax like:
    
    ?x = 6
    ?y = 9

    x = +3  // somewhere down the line I'll explain that I want to support += or x = +3 (where the operator obviously indicates that you're operating on the variable itself) ... note to self, consider how this might affect order of operations of complex assignments

    if x was 9 {...}

    OR

    if x was 6 and x = 9 {...}

    AND

    if y was x and y != 9 {...}
    

**Variadic ("approaching-infinity") parameters**
    Wanted a way to say "this function takes one or more values, and I don't know how many ahead of time" without falling into C's variadic-macro mess. Borrowing the idea of a limit - a parameter marked `->oo` (reads as "x, tending toward infinity many values") instead of a fixed name means the function is variadic in that argument. Contrast:

    (define avg x->oo n) (x + n) / n
    (define avg_2 x y) (x + y) / 2

    avg can be called with any number of numeric args (avg 4 8 15 16 23 42). avg_2 only ever takes exactly two. n is never supplied by the caller - it's bound automatically to "how many values have arrived so far" whenever a `->oo` parameter shows up in the signature; the runtime fills it, not the call site.

    The catch: for this to work, avg has to be stated recursively over its own running value rather than as a closed-form fold over a fully materialized argument list - x is meant to be "the running answer so far", not a raw list of everything passed in. So the runtime can't require every argument to be in hand before it starts producing a value; a `->oo` parameter has to behave like a lazy sequence under the hood - values pulled one at a time, only as far as something forces them. Still deciding how this interacts with the execution model (2.1), but this is the concrete example that makes lazy evaluation a real requirement rather than a nice-to-have.

    Working out a second example (running max) surfaced a gap: avg's body never named "the value that just arrived" - it only referenced x and n. Standardizing on three bound names per `->oo` step resolves it:

    x     - the running accumulator, seeded by the first value received
    next  - the value that just arrived this call
    n     - count so far, including next

    func avg x->oo n -> x + (next - x) / n
    func max x->oo n -> if next > x then next else x

    This settles the recurrence ambiguity that was open in 13.2 - the author writes the exact step expression, so it's inherently the single-accumulator/incremental form, never a full-history recompute. Still open: what happens on the very first call (n = 1, no prior x) - does it skip the step and seed x = next directly?

### 3.5 Module / System Syntax

## 4. Types

I am a self-taught programmer by way of Microsoft VBA macros. I love option explicit. I love strong typing. I know I said speed of development was priority one - I think strong typing enforces this. Yeah, you write a little more at first, but the compiler saves you a lot in the long run. Majority of this stuff is going to be a lift of C types pared to a VBA set of ideas - but minus bools. We going to the bitfields with this one (though in saying that, I think the compiler can handle bools for us - we'll see where we end up).

### 4.1 Primitive Types

### 4.2 Composite Types

**Sum types**
    Stealing Haskell's algebraic data types directly - a value that is exactly one of a fixed set of tags, and the type itself is distinct, not just an int with labels stuck on:

    data Car = Porsche | Ford | Holden

    Once pattern matching is designed, matching on a Car should force every one of Porsche/Ford/Holden to be handled - which is the exact same exhaustiveness check that constrained func (5.3) needs for "no unhandled case." One mechanism (has every case of this type been handled?), used in two places - a function's branches, and a match expression's arms.

**Type aliases**
    Also lifting Haskell's type aliasing - name a type for readability without introducing a new type:

    type Name = String
    type Age = int

    Name and String are fully interchangeable here - this is a readability tool, not a distinct type. Open question: do I also want a stronger, non-interchangeable wrapper (a Name you can't accidentally pass a raw String into)? That's a different tool than aliasing (closer to Haskell's newtype) and I haven't decided if both are worth having.

### 4.3 Reference and Pointer Semantics

Owes an answer to 3.4/13.2: func methods take self as an immutable view, sub/bang methods take it as mutable (Rust's &self/&mut self, in func/sub clothing) - this section needs to say what "immutable view" actually means at the reference level for that split to be enforceable.

### 4.4 Mutability Rules

### 4.5 Type Inference Rules

### 4.6 Compile-Time Evaluation

### 4.7 Compile-Time Function Definitions

Inspired by C macros and Rust's `constexpr`, Velocity will support **compile-time guaranteed inline functions**. This feature allows developers to define small, frequently-used functions that are always substituted at compile-time for maximum performance, while retaining type safety, scope, and idiomatic function semantics.

#### Syntax Proposal

```velocity
const_fn is_odd(int x) -> bool {
    return x & 1
}
```

* `const_fn` signals that the function is evaluated at compile-time where possible and inlined everywhere.
* Function semantics remain fully respected: type checking, scoping, and argument evaluation.
* If all arguments are compile-time constants, the compiler will evaluate the function during compilation.
* If arguments are runtime values, the function will still be inlined for performance, eliminating function call overhead.

#### Example Usage

```velocity
let a = 5
if is_odd(a) {
    print("a is odd")
}
```

* The `is_odd` function is inlined, producing fast code equivalent to `if (a & 1) {...}` without any function call overhead.

#### Advantages

1. **Readable and idiomatic code** without sacrificing performance.
2. **Type-safe alternative to macros**.
3. **Deterministic compile-time evaluation** for constant expressions.
4. Supports **generic and templated types** in future extensions.

#### Implementation Notes

* The compiler will maintain a compile-time evaluation context for `const_fn` functions.
* Recursive `const_fn` calls are allowed as long as they terminate at compile-time for constant inputs.
* For runtime values, the compiler ensures inline substitution, preventing unnecessary stack frames or jumps.

This feature bridges the gap between traditional macro substitution and runtime function calls, giving developers the best of both worlds: **speed and safety**.


## 5. Semantics

### 5.1 Variable Lifetime Rules

### 5.2 Name Resolution

### 5.3 Function Semantics

I would like to have functions return multiple types that could be specified as optional. (Also, side note - I want to have fn and mod where fn expects a return value).

fn parse_ticket(&ticket: ticket_struct) -> name: String, Option<active: bool> {

    if ticket.date < today().date() {
        active = false
    }

    name = ticket.purchaser

See 3.4 for the `->oo` variadic/running parameter form - it's the sharpest example so far of why function semantics can't assume strict evaluation of arguments. Arity and laziness have to get decided together, not laziness bolted on after arity is settled.

**Purity propagation (func/sub, 3.4)**: a `func` is only well-formed if every function reachable from its body is itself a `func`. A `sub` has no such restriction - it can call funcs or subs freely. This is checked at the definition of each function individually, not just at `main` - `main` being `func` or `sub` just happens to sit at the root of the whole call graph, so declaring `func main` is the one place this check spans the entire program. No monad or effect type needed to express it, just a call-graph check - but that check is only as good as the definition of "impure" it's built on (13.2).

**constrained func**: an optional modifier on func (3.4), orthogonal to purity - purity means no side effects, constrained means no unhandled case. A pure function can still crash on a bad input (head of an empty list); constrained closes that gap by requiring every input the function's declared type allows to produce a defined output:

    constrained func divide(a: int, b: NonZero) -> int         // bad case excluded by the type itself
    constrained func divide(a: int, b: int) -> Option<int>     // bad case pushed into the return type, every path must produce Some or None

Either form satisfies it - exclude the bad input via the type, or admit it and force the caller to handle failure explicitly. What constrained deliberately does NOT claim: that the function terminates. Termination is undecidable in general - every language that attempts to check it (Idris, Agda, Coq) does so by accepting only a conservative slice (structural recursion, a supplied decreasing measure) and rejecting or demanding manual proof for everything else, which buys false rejections on code that's obviously fine and fights directly against efficiency of movement (1.4). So constrained is scoped to the cheaper, still-valuable claim - no unhandled case, not no infinite loop - since an unhandled edge case is the failure mode that actually bites people in practice; an accidental infinite loop tends to get caught fast anyway, since it just hangs.

This reuses the same exhaustiveness check that sum-type pattern matching needs (4.2) - "has every case been handled," checked once for a match expression's arms, once for a constrained function's branches.

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

The `->oo` examples in 3.4 use a generic `define` keyword, but 3.4 also now introduces `func`/`sub` as the actual declaration keywords. Does `define` get subsumed (i.e. `avg` should really be written as `func avg x->oo n -> ...`), or does `define` stay as a keyword-inference form that picks func vs sub for you based on whether the body is pure? Needs reconciling before more examples pile up using one or the other.

### 13.2 Semantic Ambiguities

`->oo` running functions (3.4): RESOLVED (mostly) - working max (func max x->oo n -> if next > x then next else x) alongside avg showed the recurrence isn't ambiguous once the step function is written explicitly with x/next/n bound separately; the author's step expression is the only source of truth, so there's no separate "sum vs incremental" question. Still open: base case behavior on the first call (n = 1, no prior x - does it skip the step and seed x = next?).

func/sub purity (3.4, 5.3): RESOLVED - non-determinism (today()) counts as impure even without mutation. Read-only `self` methods are funcs (self is just another parameter, read vs write access decides func/sub, same as any other name - no special case needed). Still open: does mutating a local variable inside the func's own body count (leaning no - nothing escapes the call). Deliberately deferred, not forgotten: aliasing/data-race safety for read-only self across concurrent mutation - depends on 2.3 (Concurrency Model) and 4.3 (Reference and Pointer Semantics), both still unwritten.

### 13.3 VM Trade-Offs

### 13.4 Future Feature Ideas

Termination-checking as a stronger opt-in on top of constrained func (5.3) - accept structural recursion only, Idris-style, reject or require a manual annotation for anything else. Deliberately deferred, not forgotten: constrained currently only claims "no unhandled case," not "definitely terminates," because full termination checking is undecidable in general and the conservative version costs false rejections against 1.4 (efficiency of movement).

