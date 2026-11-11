# Question

Who owns runtime values, when are resources released, and what may two names alias while one mutates the value?

This shapes references, containers, closures, concurrency, and native interoperation. [Question 013](013_Local_binding_mutability.md) permits mutation through `let mut` but leaves aliasing open. All syntax below is provisional.

# Research

- [Tofte and Talpin, *Region-Based Memory Management* (1997)](https://doi.org/10.1006/inco.1996.2613): infers region allocation and deallocation with type and effect analysis, avoiding a collector while tying object lifetimes to regions.
- [Aiken, Fähndrich, and Levien, *Better Static Memory Management* (1995)](https://www.microsoft.com/en-us/research/publication/better-static-memory-management-improving-region-based-analysis-of-higher-order-languages/): improves region placement by allocating later and releasing earlier; region lifetime determines memory use, not mutation safety by itself.
- [Wadler, *Linear Types Can Change the World!* (1993)](https://www.researchgate.net/profile/Philip-Wadler/publication/2429119_Linear_Types_Can_Change_the_World/links/6410b420315dfb4cce7cf9bc/Linear-Types-Can-Change-the-World.pdf): single-use values enable controlled in-place update and disposal.
- [Jung et al., *RustBelt* (2018)](https://people.mpi-sws.org/~dreyer/papers/rustbelt/paper.pdf): analyzes ownership and borrowing safety, including obligations on unsafe libraries.
- [Boehm and Weiser, *Garbage Collection in an Uncooperative Environment* (1988)](https://hboehm.info/spe_gc_paper/): shows tracing collection can work without precise pointer maps, with conservative pointer recognition.

# Answers

## A. Ownership and checked borrowing

Values have one owner. Passing may move a value. Temporary aliases are checked, especially during mutation. Cleanup can be deterministic, at the cost of a substantial checker.

```flang
let mut items = List(1, 2)
let view = &items
// Mutating items while view is live requires a safe rule.
```

## B. Shared references and automatic collection

Values may be freely aliased; a tracing collector reclaims unreachable memory. Concurrency needs a separate data-race policy, and non-memory resources need explicit cleanup.

```flang
let mut items = List(1, 2)
let alias = items
items.push(3) // alias sees the change
```

## C. Region or arena lifetime management

Objects live in a region and are released together. References can share a lifetime tied to that region, but safe mutation still needs an aliasing rule. Returning an object beyond its region requires escape or transfer rules. Regions can coexist with ownership and borrowing; they answer a different part of the question.

```flang
region scratch {
    let items = List.in(scratch, 1, 2)
    use(items)
} // scratch storage is released
```

## D. Explicit allocation and deallocation

The programmer controls heap lifetime. Safe code would need an additional way to prevent dangling aliases, double frees, and leaks.

```flang
let ptr = allocate<List<i32>>()
defer free(ptr)
```

Hybrids are possible. A selection should say which model ordinary safe code uses and which are optional tools.

# Selected answer

A, with qualification: ownership and checked borrowing are the base model for ordinary safe values. An owned value ceases to exist when its owner leaves scope, with observable destructor timing defined by [Question 059, answer B](059_Destruction_timing.md). Optional regions or arenas remain possible; they are not the base lifetime model.
