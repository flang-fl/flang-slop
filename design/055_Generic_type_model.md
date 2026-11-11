# Question

How are functions and types parameterized by types or compile-time values, and what contract lets generic code use an argument?

Traits already exist as explicit contracts; see [Questions 025–035](025_Platform_interface_conformance.md). Code-generation strategy can be decided later. Syntax below is provisional.

# Research

- [Reynolds, *Types, Abstraction and Parametric Polymorphism* (1983)](https://people.mpi-sws.org/~dreyer/ats/papers/reynolds.pdf): distinguishes uniform behavior over a type parameter from operations chosen for particular types.
- [Damas and Milner, *Principal Type-Schemes for Functional Programs* (1982)](https://doi.org/10.1145/582153.582176): establishes principal inferred type schemes for a functional core; trait constraints require further design.
- [Kennedy and Syme, *.NET Generics* (2001)](https://www.microsoft.com/en-us/research/wp-content/uploads/2001/01/designandimplementationofgenerics.pdf): illustrates runtime-supported generics, relevant to later implementation choices.
- [Zig language reference, *Compile-Time Parameters*](https://ziglang.org/documentation/master/#Compile-Time-Parameters): demonstrates that a type parameter can be known during compilation while ordinary values remain runtime values. Its formatting example accepts any compile-time-known format string, not only a literal.
- [Rust `format_args!` documentation](https://doc.rust-lang.org/std/macro.format_args.html): describes compile-time processing of a format string literal and runtime formatting arguments.

# Answers

## A. Explicit generic parameters and trait bounds

Definitions state their type or compile-time parameters and available operations. Calls may infer arguments where unambiguous. Public contracts are clear, with some verbosity.

```flang
fn first<T>(items: Slice<T>) -> T { items[0] }
fn sort<T: Order>(items: Slice<T>) { /* ... */ }
```

## B. Infer local polymorphism, declare public contracts

Local generic types are inferred where possible; exported definitions state parameters and constraints. Mutation and overloading need restrictions on generalization.

```flang
let identity = fn(value) { value }
pub fn sort<T: Order>(items: Slice<T>) { /* ... */ }
```

## C. Compile-time functions are the main generic mechanism

A generic declaration is a compile-time function producing a type or function. This aligns with compile-time computation but makes staging, caching, and diagnostics central.

```flang
comp fn List(T: Type) -> Type { /* construct a type */ }
```

## D. Inferred compile-time parameters in `<>`; explicit compile-time arguments in `()`

Parameters in `<>` are compile-time values, typically types, that callers may omit when type or value constraints determine them. A `comp` parameter in `()` must be supplied at the call site and must be known during compilation. The difference is whether the argument is inferred, not whether it is evaluated at compile time. The `#T` spelling below is the user's provisional proposal, not a decision about the `#` sigil in [Question 019](019_Builtin_sigil.md).

```flang
comp bla = fn<#T: type>(comp fmt_string: str, bla: T) {
    // T can come from bla; fmt_string must be supplied explicitly.
}
```

The format-string example raises a separate question about dependent runtime argument lists and runtime format strings; see [Question 060](060_Format_dependent_arguments.md). The design must still decide whether a caller may explicitly supply an otherwise uninferable `<>` argument and whether inference may use expected result types.

These approaches can be combined; a selection should identify the primary surface model.

# Selected answer

D, with qualification: parameters in `<>` are compile-time parameters that the compiler infers when sufficient call context determines them. Parameters marked `comp` in `()` are compile-time-known values explicitly supplied at the call site. The chosen distinction is inference versus explicit supply, not compile-time versus runtime evaluation. Whether callers may explicitly provide an otherwise uninferable `<>` argument, which contexts participate in inference, and the exact `#T`/`type` spelling remain open. Compile-time signature generation and optional compile-time specialization are selected separately in [Questions 060](060_Format_dependent_arguments.md) and [061](061_Optional_compile_time_specialization.md).
