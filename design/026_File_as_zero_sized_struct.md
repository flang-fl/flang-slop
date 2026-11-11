# Question

If an imported file is literally represented as a struct, what does that mean at runtime?

The user wants empty structs to be a language feature and sees imported files as empty structs. A file's top-level declarations belong to its static namespace; possible runtime values representing a file may matter later for dynamic dispatch. This question fixes the basic representation without choosing a dynamic-dispatch mechanism or the syntax for constructing such values.

## Illustrative model

```flang
// windows.fl
fn newline() -> String { "\r\n" }

// root.fl
comp Windows = #import("windows.fl");
let text = Windows.newline();

// A future dynamic-dispatch feature might also allow a value
// representing Windows as an implementation of a trait.
```

`Windows` names the imported file's type or static namespace here; the exact meaning of `comp`, import results, and type/value paths remains open. The file's empty runtime representation does not make its functions empty or remove their generated code. A dynamic trait value might need dispatch metadata even if the underlying file value carries no data.

# Research

- [Rust Reference: unit-like struct types](https://doc.rust-lang.org/reference/types/struct.html#unit-like-structs) — A fieldless struct type has one value. This is the closest precedent for a nominal, data-free file representation, though Rust does not model files as structs.
- [Rustonomicon: zero-sized types](https://doc.rust-lang.org/nomicon/exotic-sizes.html#zero-sized-types-zsts) — Fieldless structs can occupy no storage; operations that produce or store them can often be removed. Zero size does not mean the type cannot have a value.
- [Rust Reference: trait objects](https://doc.rust-lang.org/reference/types/trait-object.html) — Rust's dynamic trait values use dispatch metadata alongside a pointer to a concrete value. Flang could choose a different representation; this source only shows why a zero-sized implementer does not imply zero-cost dynamic dispatch.
- [Question 023](023_Compilation_unit_size.md) — A source file was already selected as an importable struct-like namespace. This question makes the struct analogy more literal.
- [Question 025](025_Platform_interface_conformance.md) — Explores making the file itself implement a trait and forwarding that implementation through a root file.

# Answers

## A. A file is a zero-sized struct with a static namespace

An imported file is a struct with a static namespace for its top-level functions, types, and constants. The file carries no runtime fields, so a value representing it has no runtime data. A future dynamic-dispatch feature may allow such values to serve as trait implementers. Whether file types have nominal or structural identity, whether values can be explicitly constructed, and how a file type differs from a user-written empty struct remain separate questions.

## B. A file is only a namespace; an empty struct can represent it separately

Imports yield a namespace, while a declared empty struct implements traits when needed. This keeps file identity out of the value type system, but requires an adapter type for the platform-interface pattern.

## C. A file namespace is modeled internally as a struct but has no language-level value

The compiler may reuse struct machinery for imports and trait checking, but flang code cannot construct or pass the file as a value. Dynamic dispatch would require a separate representation or a later language change.

# Selected answer

**A, qualified.** The user wants empty structs as a language feature and imported files to be literal empty structs at runtime. They do not expect ordinary per-instance data, while leaving open future runtime values for dynamic dispatch. Exact construction, type identity, type/value namespace, and dispatch rules remain undecided.
