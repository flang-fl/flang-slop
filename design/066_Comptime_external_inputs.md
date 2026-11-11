# Question

What information may compile-time code read from outside the program, and how does the compiler track those inputs?

The selected file-root design permits programmatic directory scanning in [Question 023](023_Compilation_unit_size.md). Target selection in [Question 022](022_Conditional_module_files.md) also requires compiler-provided target information. Compile-time introspection remains open in [Question 016](016_Compiler_introspection_and_messages.md). This boundary affects reproducibility, caching, and a compile-time evaluator prototype.

# Research

- [Jones, Gomard, and Sestoft, *Partial Evaluation and Automatic Program Generation* (1993)](https://www.cs.utexas.edu/~novak/jonesgomardsestoft.pdf): specialization depends on which inputs are fixed at the earlier stage.
- [Zig language reference, *Compile-Time Parameters*](https://ziglang.org/documentation/master/#Compile-Time-Parameters): compile-time parameters require values known to the compiler; target and file queries are separate built-ins.
- [Rust Reference, *Conditional compilation*](https://doc.rust-lang.org/reference/conditional-compilation.html): target and build configuration can select source forms before runtime.
- [Jonathan Blow, LambdaConf 2024 interview](https://www.youtube.com/watch?v=7BaWley751Y&t=930s): Blow describes an early Jai demo where a game ran during compilation and the compiler withheld the executable unless the player killed ten aliens. This illustrates observable host interaction and a compile result dependent on it.
- [Question 023](023_Compilation_unit_size.md): directory scans must become tracked build dependencies to avoid stale module results.

# Answers

## A. Pure compile-time code plus explicit, tracked queries

Ordinary compile-time computation is deterministic from its arguments. Compiler APIs expose target information, source files, directory listings, and other approved inputs; the compiler records their use as dependencies. This supports caching and reproducible results but requires a deliberate query API.

## B. Arbitrary host I/O during compilation

Compile-time code can read the filesystem, environment, time, network, or processes directly. This is flexible for code generation but makes builds harder to reproduce, cache, and sandbox. The compiler must decide how to detect external changes.

## C. All external inputs supplied by the build invocation

Compile-time code receives only inputs declared by the build configuration, including source directories and target settings. This makes dependencies explicit but gives the source program less autonomy; even the selected root-file scan would need a declared directory input.

# Selected answer

B, with qualification: compile-time flang code may use host facilities available to the compiler, including filesystem, environment, processes, network, and interactive or graphical I/O. It may perform observable side effects and let external events determine whether compilation succeeds. The language does not require compile-time computation to be pure or reproducible. The host API and FFI mechanisms, build-cache invalidation, and any opt-in sandbox or dependency-tracking tools remain open.
