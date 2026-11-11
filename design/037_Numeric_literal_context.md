# Question

Which numeric types can context give an untyped numeric literal?

The user has decided that a numeric literal remains untyped until context gives it a shape. With no determining context, an integer-form literal defaults to `i32`, while one with decimal places defaults to `f64`. This question asks whether spelling restricts the eligible type family before those defaults apply. For example, can `42` become `f64`, and can `42.0` become `i32`? A possible DiamondFire-native `dfnum` adds another context. The exact name and semantics of `dfnum` remain open.

```flang
let whole_float: f64 = 42;
let integral_decimal: i32 = 42.0;
let df_score: dfnum = 42;

let default_whole = 42;    // i32
let default_decimal = 42.0; // f64
```

These examples are provisional and do not assert that the first three assignments are legal.

# Research

- [Swift language reference: lexical structure and literals](https://docs.swift.org/latest/documentation/the-swift-programming-language/lexicalstructure/) — Swift says a literal has no type on its own, uses context to infer one, and applies default integer and floating types when context is insufficient. Its literal protocols let target types opt into integer or floating literal construction.
- [Rust Reference: literal expressions](https://doc.rust-lang.org/reference/expressions/literal-expr.html) — Unsuffixed integer and floating literals get types from context and default to `i32` and `f64` respectively. Rust also distinguishes integer and floating literal forms and reports errors when context over-constrains the type.
- [Zig language reference: numeric literals](https://ziglang.org/documentation/master/) — Zig's `comptime_int` and `comptime_float` hold numeric constants before a concrete runtime type is chosen. This shows another way to delay numeric representation, though flang has only committed to contextual literal typing, not to Zig's full compile-time number system.
- [Question 036](036_DiamondFire_native_types.md) — Converting an already typed ordinary numeric value to a DiamondFire-native numeric value requires an explicit operation. Giving a still-untyped literal a native type is a distinct design choice.

# Answers

## A. Any exactly representable numeric type may shape either literal form

Context may give `42` type `f64`, `42.0` type `i32`, and `42` type `dfnum` if each value is representable under that type's rules. The decimal point affects only the fallback default. This is flexible, but makes the spelling of a whole-valued decimal less informative about its eventual type.

## B. The spelling fixes the integer or fractional type family

`42` may become an integer type; `42.0` may become a floating type. Context chooses a type within that family, and the selected defaults apply when it cannot. Whether `dfnum` accepts either form depends on its eventual numeric semantics. This makes spelling informative but requires explicit conversion or another spelling when calling an API in the other family.

## C. Integer-form literals can take floating types, but decimal-form literals stay floating

`42` may become `i32`, `f64`, or a compatible `dfnum`; `42.0` can take only a floating type. The decimal point commits to floating semantics, while a whole-number spelling remains useful in floating contexts. This is asymmetric but matches common numeric-literal conventions.

# Selected answer

**B.** Literal spelling selects its numeric family before context selects a concrete type: `42` stays in the integer family and `42.0` stays in the floating family. Context may select a compatible type within that family. With no determining context, the defaults remain `i32` and `f64`, respectively. Whether a future DiamondFire-native type accepts either form depends on which numeric family that type belongs to.
