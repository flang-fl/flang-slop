# Question

How should a DiamondFire-native type such as `dfnum` relate to ordinary flang numeric types?

The user has decided that DiamondFire is a future compilation target, existing flang types should work there, and native built-in types should be available to avoid some representation overhead. Provisional examples include `DfString`, `DfVector`, and `DfNum`; their names and exact meanings remain open. This question concerns the boundary between those type families. `dfnum` is an earlier illustrative name; its numeric range, precision, and arithmetic semantics are not yet decided. All syntax below is provisional.

```flang
fn score_bonus(score: dfnum) -> dfnum {
    score + 10
}

let ordinary: i64 = 42;
let native: dfnum = /* how does ordinary become a DiamondFire number? */;
```

# Research

- [DiamondFire Docs: Code Slots](https://mcdiamondfire.com/docs/code-slots/) — DiamondFire's own documentation shows action slots with expected value types, including `Number`. It also describes type information as useful for optimization. This supports exposing target-native values, but does not specify a flang representation or prove a particular overhead difference.
- [Go specification: conversions](https://go.dev/ref/spec#Conversions) — Go distinguishes explicit conversions from ordinary assignment. This is a precedent for making a change of numeric representation visible at the source level.
- [Rust Reference: type cast expressions](https://doc.rust-lang.org/reference/expressions/operator-expr.html#type-cast-expressions) — Rust defines explicit numeric casts, including behavior when the destination cannot preserve the source value. It illustrates why flang would need to specify range and precision before permitting conversion to `dfnum`.
- [Question 018](018_Comptime_if_branch_checking.md) — Only the selected compile-time branch is semantically checked, which could permit target-specific types inside a DiamondFire-only branch.

# Answers

## A. Require explicit conversions in both directions

`dfnum` and ordinary numeric types are distinct. A program converts at the boundary with an explicit operation, for example `dfnum.from_i64(ordinary)` and `native.to_i64()`. The conversion API can report overflow or precision loss when needed. This makes the cost and semantics visible, but adds code around DiamondFire actions.

## B. Permit only proven lossless implicit conversions

The compiler inserts a conversion only when it can prove the value fits and its meaning is preserved; other conversions require an explicit operation. For example, a suitable literal might initialize `dfnum`, while a runtime `i64` might need checking. This eases common calls but depends on the eventual range and precision rules.

## C. Permit implicit conversions in both directions

Ordinary numbers and `dfnum` flow freely between APIs, while `dfnum` remains available for explicitly native storage and operations. This is convenient, but conversion costs and possible changes in numeric meaning can be hidden in ordinary expressions. The language would need a rule for mixed-type arithmetic.

# Selected answer

**A, qualified by [Question 037](037_Numeric_literal_context.md).** Conversion between an already typed ordinary number and a DiamondFire-native numeric value requires an explicit operation in either direction. Numeric literals acquire their type from context before this conversion rule applies; their eligible target types remain open. An untyped literal in a `dfnum` context is therefore a separate question, not an implicit conversion from `i32` or `f64`.
