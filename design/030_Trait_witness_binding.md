# Question

After a file explicitly declares `impl PlatformApi`, how are the trait's required functions matched to the file's functions?

[Question 025, answer E](025_Platform_interface_conformance.md) requires explicit conformance declarations. The platform file examples have top-level functions with the required names, while [Question 027, answer E](027_Trait_scoped_reexport.md) places function-value bindings inside the root's `impl` block. The language needs one rule explaining both forms. All syntax is provisional.

## Example

```flang
trait PlatformApi {
    fn newline() -> String;
}

// linux.fl
fn newline() -> String { "\n" }
impl PlatformApi for #File;

// root.fl
comp api: impl PlatformApi = #import("linux.fl");
impl PlatformApi for #File {
    comp newline = api.newline;
}
```

Does the bare declaration in `linux.fl` find its existing `newline` automatically? Must the root explicitly bind its own `newline` because its selected function lives under `api`?

# Research

- [Swift language guide: declaring protocol adoption with an extension](https://docs.swift.org/latest/documentation/the-swift-programming-language/protocols/#Declaring-Protocol-Adoption-with-an-Extension) — Swift requires explicit protocol adoption, but an empty extension suffices when the type already has matching members. This is a close precedent for matching a file's existing top-level members after an explicit `impl` declaration.
- [Rust Reference: trait implementations](https://doc.rust-lang.org/reference/items/implementations.html#trait-implementations) — Rust requires the `impl` block to define each non-default associated item. This is a precedent for requiring explicit witness bindings within the block.
- [Go specification: interface types](https://go.dev/ref/spec#Interface_types) — Go matches methods structurally without an explicit conformance declaration. It illustrates member matching but differs from flang's selected explicit declaration rule.
- [Question 029](029_Composite_trait_member_collisions.md) — Combined trait views require qualification when member names overlap; witness matching must also be defined when a file implements multiple traits with the same member name.

# Answers

## A. Match same-name top-level functions after an explicit declaration

`impl PlatformApi for #File;` checks the file's existing top-level `newline` and uses it as the trait witness. The root must first create its own top-level alias, such as `comp newline = api.newline`, before declaring conformance. This keeps `impl` blocks simple but makes explicit remapping to a differently named function unclear.

## B. Require every witness inside the `impl` block

```flang
impl PlatformApi for #File {
    comp newline = #File.(inherent).newline;
}
```

Every required member must be named in the block, even if a same-name top-level function already exists. This makes witness selection explicit and supports remapping, but repeats each platform file's own names.

## C. Match same-name members by default; allow explicit witness bindings

A bare `impl PlatformApi for #File;` uses compatible same-name top-level members. An `impl` block may supply `comp newline = api.newline` to bind a different function value or override the default match. The root can use the selected file's function directly without writing a wrapper body. The compiler must define precedence when both an explicit witness and a same-name top-level member exist, and whether explicit witnesses also become top-level names.

# Selected answer

**B.** Every non-default trait member must have an explicit witness inside the `impl` block, even when a compatible same-name top-level function exists. A bare `impl PlatformApi for #File;` does not satisfy a trait with required members. The witness may bind an existing function value; exact syntax and the visibility of witnesses outside the trait view remain open.
