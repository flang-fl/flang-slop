# Question

How should a function propagate errors from `Result`-returning calls, and may its own error type be inferred from its body?

[Question 043, answer B](043_Fallible_arithmetic_result.md) makes `try_add` return `Result<T, OverflowError>`. The user wants to revisit whether propagation should be convenient, perhaps with `Result<T, inferred>` and a compiler-created tagged union of encountered error types. The concern is that adding an error path inside a function could silently change its signature. Restricting error inference to non-public functions is one possible boundary. No propagation syntax, error union mechanism, or inference policy has been chosen. The example is illustrative only.

```flang
fn compute(text: String, value: i32) -> Result<i32, inferred> {
    let parsed = parse_i32(text)?; // Suppose this returns Result<i32, ParseError>.
    try_add(parsed, value)        // Returns Result<i32, OverflowError>.
}
```

Could the compiler infer an error type containing both `ParseError` and `OverflowError`? If `compute` is public, would adding another fallible call silently change its published type?

# Research

- [Rust Reference: try propagation operator](https://doc.rust-lang.org/reference/expressions/operator-expr.html#the-try-propagation-operator) — Rust's `?` propagates `Result::Err` from the current function, converting the error with `From` where applicable. The function still declares its result type; propagation does not infer a public union of errors.
- [Zig language reference: inferred error sets](https://ziglang.org/documentation/master/) — Zig can infer the set of errors a function returns from its implementation. Its documentation notes complications for function pointers, consistency across build targets, and recursion, and recommends explicit sets in such cases. This directly informs the public-signature stability concern.
- [Zig language reference: merging error sets](https://ziglang.org/documentation/master/) — Zig can merge error sets, showing one way to represent several propagated error cases without hand-writing a separate union type for each function.
- [Question 043](043_Fallible_arithmetic_result.md) — Selects a named `OverflowError` type for fallible arithmetic. Any inferred union would need to include that type or define a conversion into another declared error type.

# Answers

## A. Require an explicit error type in every function signature

Propagation may still have concise syntax, but the declared `Result<T, E>` fixes the error type. Each propagated error must be the declared type or be explicitly/implicitly convertible under a separate rule. Implementation edits cannot silently widen the function signature. This makes private helper signatures more verbose.

## B. Infer error types only for non-public functions

Private helpers may write a placeholder such as `Result<T, inferred>` and let the compiler form an error union from their bodies. Public functions must name a stable error type and map propagated errors into it. This balances convenience with a deliberate published API, but adding an error path can still change private callers and inferred types may complicate recursion.

## C. Allow inferred error types in any function

Both private and public functions may infer an error union from their bodies. This minimizes annotations and keeps propagation concise. A body edit can change a public function's type, so callers and downstream builds may break even when the function declaration text is unchanged.

# Selected answer

Pending. The user asked to revisit convenient error propagation and inferred error types later, especially their effect on public function signatures. No option has been selected.
