# Question

What syntax explicitly selects a file's top-level member or a particular trait witness when their names collide?

[Question 029, answer A](029_Composite_trait_member_collisions.md) requires qualification when two traits contribute the same name. [Question 032, answer B](032_Inherent_vs_trait_member_lookup.md) also requires it when a top-level file member and a trait witness collide. The syntax must work both at call sites and inside an `impl` block that binds an existing top-level function as a witness. All examples below are provisional.

## Example requiring qualification

```flang
// windows.fl
fn newline() -> String { "\r\n" }

impl PlatformApi for #File {
    comp newline = /* explicitly select the top-level newline */;
}

// Elsewhere, explicitly select PlatformApi.newline on windows.fl.
```

# Research

- [Rust book: fully qualified syntax](https://doc.rust-lang.org/book/ch20-02-advanced-traits.html#using-fully-qualified-syntax) — Rust writes `<Type as Trait>::function` to specify a trait implementation's receiver-free associated function. The full path is explicit but verbose.
- [Rust Reference: qualified paths](https://doc.rust-lang.org/reference/paths.html#qualified-paths) — Formalizes paths that name an associated item through a type and trait. Flang can adopt a different spelling while preserving the same disambiguation idea.
- [OCaml manual: module ascription](https://ocaml.org/manual/5.3/modules.html) — A module expression can be viewed through a particular module type, hiding other members. This is a precedent for qualifying by interface view rather than by a special member-path form.
- [Question 028, answer B](028_Comptime_impl_binding.md) — Compile-time `impl Trait` annotations already create restricted trait views; a qualification syntax might reuse that concept.

# Answers

## A. Qualify the member segment

```flang
comp newline = #File.(inherent).newline;
comp trait_newline = #File.(PlatformApi).newline;
```

The receiver remains the same; the parenthesized segment selects the member namespace. This is compact and works for a single member without creating a new binding. The exact keyword for top-level members could differ from `inherent`.

## B. Cast or ascribe to a member view

```flang
comp newline = (#File as inherent).newline;
comp trait_newline = (#File as impl PlatformApi).newline;
```

This reuses the idea of restricted compile-time views and can select several members through one bound view. It needs a defined `inherent` view and a rule that ascription changes lookup without creating runtime dispatch or data conversion.

## C. Use a fully qualified path

```flang
comp newline = <#File as inherent>::newline;
comp trait_newline = <#File as PlatformApi>::newline;
```

This resembles Rust's explicit associated-item paths. It makes the type and origin visible at the access site and works without a temporary view. The punctuation is heavier and may conflict with flang's eventual generic or comparison syntax.

# Selected answer

**A, qualified.** A parenthesized segment on the member path selects its origin: `file.(Trait).name` selects a trait witness, while a corresponding `file.(inherent).name` selects a top-level member. The precise keyword for top-level members and the spelling of the file self-reference remain open. The qualifier changes static name lookup; it does not imply a runtime cast or dispatch.
