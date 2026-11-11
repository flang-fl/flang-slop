# Question

When a compile-time function may perform arbitrary host side effects, when must the compiler execute it again, and when may it reuse a previous result?

[Question 066, answer B](066_Comptime_external_inputs.md) permits host I/O and interactive compilation. The compile-time build entry in [Question 065](065_Program_entrypoints.md) can register runtime functions. If an earlier build result is reused, a side effect such as playing a game, writing a file, or checking a live service may not happen. Syntax below is provisional.

# Research

- [Jonathan Blow, LambdaConf 2024 interview](https://www.youtube.com/watch?v=7BaWley751Y&t=930s): describes an early Jai build whose result depended on an interactive game during compilation.
- [Cargo Book, *Build Scripts*](https://doc.rust-lang.org/cargo/reference/build-scripts.html): build scripts can perform host actions; Cargo uses change-detection instructions to decide whether to rerun them.
- [Zig Build System](https://ziglang.org/learn/build-system/): build functions configure artifacts and tasks, illustrating a programmable build entry whose execution and cache behavior matter.
- [Question 023](023_Compilation_unit_size.md): programmatic source-directory scans create a build dependency even when their computation has no visible side effect.

# Answers

## A. Execute compile-time code on every requested build

Each compiler invocation runs the designated build entry and executes any demanded compile-time calls according to source control flow. Persistent caches may hold parsing or code generation results but may not suppress compile-time side effects. This makes interactive behavior predictable and simplifies the first implementation, but repeated builds can be expensive.

## B. Permit caching and skipping compile-time execution

The compiler may reuse a prior compile-time result when it believes relevant inputs have not changed. Builds can be fast, but arbitrary untracked host input and side effects make the skip observable. A game-gated build might not ask the player to play again.

## C. Effectful by default, with explicit cacheable compile-time computations

The build entry and ordinary compile-time code execute on every requested build unless a function or expression is explicitly declared cacheable under a dependency contract. This preserves the unrestricted model and permits deliberate caching, at the cost of a purity or dependency-checking mechanism and rules for calls between cacheable and effectful code.

# Selected answer

A, with qualification: each requested build runs the designated compile-time build entry and executes its demanded compile-time calls according to source control flow. Persistent compiler caches may not suppress observable compile-time side effects. Explicitly cacheable computations may be added later as an opt-in refinement; their contract remains open.
