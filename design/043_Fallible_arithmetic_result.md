# Question

What should the error type be in the `Result` returned by a fallible integer operation such as `try_add`?

[Question 042, answer A](042_Checked_overflow_failure.md) makes ordinary checked-build overflow stop the current execution. The user wants a separate operation for code that needs to detect overflow at runtime and continue. The user has narrowed its return type to `Result<T, E>` because overflow has only one failure state. This question compares a general unit error, a distinct named unit-struct error, and an error-set alternative. The names `try_add`, `unit`, `Result`, and `OverflowError` are provisional.

```flang
let outcome = try_add(value, 1); // Result<i32, E>; what is E?
```

# Research

- [Rust standard library: `Result<T, E>`](https://doc.rust-lang.org/std/result/) — `Result` separates a success value from an error value. Its error type may be a unit type or a named type; the return structure itself does not require a detailed error payload.
- [Rust Reference: unit type](https://doc.rust-lang.org/reference/types/tuple.html#unit) — The unit type has one value. A `Result<T, unit>` error therefore communicates only that the operation failed.
- [Rust Reference: unit-like structs](https://doc.rust-lang.org/reference/items/structs.html#unit-like-structs) — A named fieldless struct also has one value, while retaining a distinct type identity. This supports distinguishing overflow from other single-state failures in generic error handling.
- [Zig language reference: error sets](https://ziglang.org/documentation/master/) — Zig can name one error as `error.Overflow`, merge error sets, and narrow remaining cases after handling one. This is a precedent for a composable set of named failure cases rather than a separate struct for each.
- [LLVM Language Reference: overflow intrinsics](https://llvm.org/docs/LangRef.html#llvm-sadd-with-overflow-intrinsics) — LLVM provides an arithmetic result plus an overflow flag; either error type could be built from that flag on the LLVM target.
- [Question 042](042_Checked_overflow_failure.md) — Establishes the explicit, recoverable path as a separate API from ordinary checked arithmetic.

# Answers

## A. Use the general unit type as the error

`try_add(a, b) -> Result<T, unit>` returns the sum or the sole unit error value. This is minimal and reflects that there is only one failure state. The error type alone does not identify overflow when several unrelated operations also use unit errors.

## B. Use a named unit struct as the error

`try_add(a, b) -> Result<T, OverflowError>` returns the sum or a value of a distinct fieldless `OverflowError` type. The error carries no extra data, but its type identifies the failure and can support type-specific handling. It requires naming and defining an error type for this one case.

## C. Use a singleton error set

`try_add(a, b) -> Result<T, error{Overflow}>` uses a named `Overflow` case in a set of possible errors. A function combining fallible addition with other operations could return a union of their error cases. This avoids a separate struct declaration for every single-state failure, but requires a general error-set feature and rules for merging and matching sets. The syntax is illustrative.

# Selected answer

**B.** A fallible integer operation such as provisional `try_add` returns `Result<T, OverflowError>`, where `OverflowError` is a distinct unit struct with no data fields. Its one value represents the only failure state, while its type identifies overflow to callers. The broader question of convenient error propagation and whether function error types can be inferred is open in [Question 044](044_Error_propagation_and_inference.md).
