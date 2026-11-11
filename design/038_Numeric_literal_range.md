# Question

What happens when a numeric literal does not fit the type selected by context, or does not fit its default type when context selects none?

[Question 037, answer B](037_Numeric_literal_context.md) assigns literals to integer or floating families before choosing a concrete type. Context can choose a type such as `i8`; without context, an integer literal defaults to `i32`. The compiler still needs a rule for values outside the chosen type's range. The examples focus on integers; floating rounding and overflow need their own rules.

```flang
let narrow: i8 = 300;
let huge = 2147483648; // no contextual type; larger than signed i32
```

# Research

- [Rust Reference: literal expressions](https://doc.rust-lang.org/reference/expressions/literal-expr.html) — Unsuffixed integer literals default to `i32` when context under-constrains them. Rust describes a cast to the selected type, while `rustc` denies its `overflowing_literals` lint by default, so a non-fitting literal is normally diagnosed.
- [Swift language guide: numeric type conversion](https://docs.swift.org/swift-book/LanguageGuide/TheBasics.html) — Swift reports an error when a number cannot fit into a sized integer destination. Its contextual literal model is close to the one already chosen for flang.
- [Zig language reference: runtime integer values](https://ziglang.org/documentation/master/) — Zig permits integer literals without a size limit at compile time, but a runtime integer value needs a concrete size. This is a precedent for retaining an exact constant until a runtime boundary.
- [Question 037](037_Numeric_literal_context.md) — The chosen fallback type is `i32` for integer-form literals; this question decides whether that fallback can change when the value will not fit.

# Answers

## A. Reject any literal that does not fit its selected or default type

Both examples are compile errors. The programmer must write an appropriate type context or otherwise request a larger type. This keeps the `i32` fallback literal and predictable, but a large unsuffixed constant needs an annotation even when a larger integer type exists.

## B. Reject a bad contextual fit, but widen an unconstrained fallback

`narrow` is an error because `i8` was requested. For `huge`, the compiler picks a wider integer type that fits instead of `i32`; the exact widening order would need a rule. This is convenient for large constants but makes the fallback type depend on the literal's value.

## C. Reject a bad contextual fit, but keep an unconstrained large value exact at compile time

`narrow` is an error. `huge` remains an exact compile-time integer value until a runtime use requires a concrete integer type. This supports very large constants and computations, but complicates the promise that an unconstrained literal defaults to `i32` and requires a rule for when defaulting occurs.

# Selected answer

**A, clarified after initially considering C.** A numeric literal must fit the concrete type selected by context or, when context does not determine one, its default type. An unconstrained integer-form literal always defaults to `i32`; it does not silently widen or remain an untyped value merely because it is too large. Thus `let huge = 2147483648` is an error unless a suitable integer type is supplied by annotation or another context. Likewise, `let narrow: i8 = 300` is an error. This decision does not yet settle how compile-time arithmetic represents intermediate values when a suitable final type is known.
