# Question

When an arithmetic expression uses a named, already typed compile-time constant, does it still receive the exact-intermediate rule from [Question 039, answer C](039_Numeric_expression_context.md)?

The user wants constant formulas to be judged by their final result. The open boundary is whether a typed `comp` binding remains a fixed-width numeric operand or contributes its known mathematical value to exact arithmetic. The chosen [Question 038, answer A](038_Numeric_literal_range.md) still requires the binding's initializer to fit its declared or default type.

```flang
comp maximum: i32 = 2147483647;
let same: i32 = (maximum + 1) - 1;
```

The final value fits `i32`, but `maximum + 1` does not.

# Research

- [Rust Reference: constant evaluation](https://doc.rust-lang.org/reference/const_eval.html) — Rust permits paths to constants in constant expressions, but overflow in a required compile-time evaluation is an error. This is a precedent for a named typed constant retaining fixed-width arithmetic semantics.
- [Zig language reference: compile-time expressions](https://ziglang.org/documentation/master/) — Zig distinguishes values known at compile time from concrete numeric types; its examples show typed `u32` arithmetic can still overflow during compile-time evaluation. This separates compile-time knowledge from arbitrary precision.
- [Question 039](039_Numeric_expression_context.md) — Selects exact evaluation for arithmetic built from numeric literals before checking the final result, while reserving the treatment of already typed named constants for this question.

# Answers

## A. A typed constant keeps its declared numeric type during arithmetic

`same` is an error: the `i32` value `maximum` participates in `i32` arithmetic, and `maximum + 1` overflows. Exact intermediate arithmetic applies to literal-only formulas until a typed binding creates a boundary. This preserves the meaning of the binding's type but may require an explicit wider type for named constant formulas.

## B. Any compile-time-known numeric value participates by its exact value

`same` is valid: the compiler uses the mathematical value of `maximum`, computes the entire constant expression exactly, and checks the final result against `i32`. This supports named formulas directly, but arithmetic involving a typed constant can differ depending on whether its other inputs are known at compile time.

## C. Exactness through names requires an explicitly exact numeric binding

A separate declaration or type would mark `maximum` as an exact compile-time number; ordinary typed `comp` bindings retain fixed-width arithmetic. The example as written fails, but changing the binding to the exact form makes it work. This keeps the distinction visible at declaration sites, at the cost of another numeric facility whose spelling and scope need design.

# Selected answer

**B.** A named compile-time constant participates by its exact numeric value in an arithmetic expression whose inputs are all known during compilation, even if the binding itself has a concrete numeric type. The binding's own initializer must still fit that type under [Question 038](038_Numeric_literal_range.md). The compiler checks the final expression result against its context-selected type; thus `same` is valid. When an operand depends on runtime state, ordinary typed arithmetic applies instead. Runtime integer overflow is checked and fails in debug and release-safe builds; behavior in other build modes is open in [Question 041](041_Runtime_integer_overflow_modes.md).
