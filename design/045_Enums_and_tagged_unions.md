# Question

Should flang distinguish `enum` as a closed set of named, compile-time values of one separate type from `union` as a closed, tagged set of alternatives that may carry data?

The proposed distinction is:

```flang
comp Colors = enum Color {
    RED   = Color.new(255, 0, 0);
    GREEN = Color.new(0, 255, 0);
    BLUE  = Color.new(0, 0, 255);
}

comp Enemy = union {
    Slime: struct {
        health: i32,
        damage: f64,
    };
    Evil: EvilStruct,
    Thing: f32,
}
```

`Color` is an independently defined type. `Colors` values would carry only an option identifier, stored in the smallest unsigned integer type able to distinguish its options; the `Color` values belong to the compile-time definition and are not copied into each `Colors` value. Both types would support exhaustive matching. A union value would carry its active tag and any associated payload. The examples and punctuation are provisional syntax.

For example, a match on `Colors` could name `RED`, `GREEN`, and `BLUE`, while a match on `Enemy` could bind the `Slime` fields, an `EvilStruct`, or an `f32`. A plain integer-like collection of named values could be expressed by either mechanism, but `enum` would be the idiomatic form.

[Question 046, answer A](046_Duplicate_enum_values.md) settles that equal `Color` values under different enum names are distinct options. This question does not settle how an enum's underlying type is exposed, whether the chosen width is an arbitrary bit width or a fixed-width type such as `u8`, or how an empty or single-option enum is represented. Union layout and tag numbering are also separate questions.

# Research

- [Zig language reference: enum, tagged union, and exhaustive switch](https://ziglang.org/documentation/master/) — Zig keeps enums and unions distinct. A tagged union associates a tag with payload fields, and `switch` can bind those payloads while checking that all cases are covered. Flang's proposed enum differs because its cases name arbitrary compile-time `Color` values, rather than merely tag names or integer discriminants.
- [Rust Reference: enumerations](https://doc.rust-lang.org/reference/items/enumerations.html) — Rust uses one `enum` construct for both fieldless variants and data-carrying variants. Its logical discriminant identifies the active variant, but the compiler may choose a different physical representation unless one is specified. This is the main naming and representation alternative to flang's proposed split.
- [OCaml language reference: variant values](https://ocaml.org/manual/5.5/values.html) — OCaml distinguishes constant constructors from constructors carrying arguments within one variant type. This shows that exhaustive case analysis does not require separate `enum` and `union` declarations.
- [OCaml documentation: memory representation](https://ocaml.org/docs/memory-representation) — Constant-only variants are represented as integers in OCaml's runtime. That supports the compact-tag motivation, though it does not determine flang's exact unsigned width or ABI.

# Answers

## A. Separate value-set enums and tagged unions

Adopt the proposed distinction. An `enum T` defines a closed set of named compile-time `T` values and runtime enum values contain only compact option identifiers. A `union` defines a closed set of tagged alternatives, each optionally carrying a payload. Both are exhaustively matchable. Physical layout details remain follow-up questions; duplicate values are settled by [Question 046](046_Duplicate_enum_values.md).

## B. One general variant type

Use one closed variant construct for both fieldless and payload-carrying cases, as Rust and OCaml do. Named `Color` values can be associated with cases as compile-time metadata, but there is no separate `enum` category. This reduces the number of type constructs, while giving up the proposed meaning of `enum` as a named value set.

## C. Separate integer enums and tagged unions

Keep `enum` and `union` distinct, but permit enum cases to assign only integer discriminants. The `Color` values would live in a separate compile-time lookup table. This keeps the compact, exhaustive enum representation, but makes the example's direct association of names with arbitrary `Color` values less natural.

# Selected answer

**A.** Enums are closed sets of named, compile-time values of a separate type. Runtime enum values store only a compact unsigned option identifier. Unions are closed, tagged alternatives that may carry payloads. Both support exhaustive matching. Duplicate enum values are settled by [Question 046](046_Duplicate_enum_values.md); exact integer widths and union layout remain open.
