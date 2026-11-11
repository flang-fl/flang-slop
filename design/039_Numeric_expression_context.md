# Question

How far does an expected numeric type propagate into an arithmetic expression made from literals?

[Question 037, answer B](037_Numeric_literal_context.md) lets context choose the concrete type of each literal within its numeric family. [Question 038, answer A](038_Numeric_literal_range.md) rejects a literal that does not fit its selected or default type. Those decisions do not yet say whether the expected type of an entire expression reaches the literals and operators inside it, or whether constant arithmetic is evaluated exactly before assigning a type. The examples assume ordinary integer `+` and `-`; runtime overflow behavior remains a separate question.

```flang
let wide: i64 = 2147483647 + 1;
let canceled: i32 = (2147483647 + 1) - 1;
```

# Research

- [Swift language reference: type inference](https://docs.swift.org/latest/documentation/the-swift-programming-language/types/) — Swift explicitly allows type information to flow from an expression's root down to its literal leaves. This is a precedent for letting `i64` in `wide` shape both operands.
- [Rust Reference: literal expressions](https://doc.rust-lang.org/reference/expressions/literal-expr.html) — Rust chooses unsuffixed literal types from surrounding context, defaulting to `i32` only when context under-constrains them. It shows why the point at which defaulting happens matters inside a larger expression.
- [Zig language reference: peer type resolution and integer literals](https://ziglang.org/documentation/master/) — Zig resolves types across operands and can keep integer literal values exact at compile time. This illustrates the more permissive constant-expression approach without requiring flang to adopt Zig's compile-time numeric types.
- [Question 038](038_Numeric_literal_range.md) — An unconstrained integer literal defaults to `i32` and errors if it does not fit. This question asks when an enclosing expression counts as determining context.

# Answers

## A. Expected type flows into operators and literals; each intermediate is checked

`wide` is valid: `i64` shapes the addition and both literals. `canceled` is an error because its first addition exceeds `i32` before subtraction brings the value back into range. This gives consistent typed operations and catches intermediate overflow.

## B. Each subexpression determines its type locally before the outer expectation

Both examples are errors: the inner additions default to `i32` and overflow before their results can be assigned to the declared type. This limits top-down inference but makes explicit type annotations or casts inside an expression more common.

## C. Constant-only arithmetic is exact until the final result receives a type

Both examples are valid: the compiler evaluates constant arithmetic exactly and checks only the final result against `i64` or `i32`. This supports convenient constant formulas, but compile-time expressions can behave differently from the corresponding arithmetic on runtime values.

# Selected answer

**C, extended by [Question 040, answer B](040_Typed_constants_in_exact_arithmetic.md).** Arithmetic whose numeric inputs are all known during compilation is evaluated exactly before its final result receives the context-selected type. In the examples, both `wide` and `canceled` are valid because their final values fit `i64` and `i32`, respectively. Named typed compile-time constants also participate by their exact values. Runtime-dependent arithmetic uses its concrete numeric types and follows the overflow rule in [Question 041](041_Runtime_integer_overflow_modes.md).
