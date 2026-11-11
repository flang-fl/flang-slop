# Question

When a file has both a top-level function and a trait witness with the same name, what does `file.name` find?

[Question 030, answer B](030_Trait_witness_binding.md) requires an explicit witness even when a matching top-level function exists. [Question 031, answer B](031_Trait_witness_lookup_on_files.md) allows ordinary file lookup to find unambiguous trait witnesses. The platform-file example now contains both kinds of member, so lookup needs a priority rule. All syntax is provisional.

## Example

```flang
// windows.fl
fn newline() -> String { "\r\n" }

impl PlatformApi for #File {
    comp newline = #File.(inherent).newline;
}

// caller.fl
comp windows = #import("windows.fl");
windows.newline(); // Top-level function or trait witness?
```

The witness initializer explicitly refers to the pre-existing top-level function under [Question 033, answer A](033_Member_origin_qualification.md); the keyword `inherent` is provisional. In this example both paths lead to the same function value, but another implementation could bind a different witness. The lookup rule should work in both cases.

# Research

- [Rust Reference: method-call candidate search](https://doc.rust-lang.org/reference/expressions/method-call-expr.html#candidate-search) — Rust considers inherent methods before visible trait methods for a given candidate receiver type, though auto-borrow and auto-deref can complicate the final result. Flang's receiver-free file functions need their own simpler rule.
- [Rust book: same-name inherent and trait functions](https://doc.rust-lang.org/book/ch20-02-advanced-traits.html#disambiguating-between-methods-with-the-same-name) — Demonstrates that a type's inherent function may be selected by a short path while a fully qualified path selects the trait function.
- [Question 029](029_Composite_trait_member_collisions.md) — Two trait contributions with the same name are ambiguous and need trait qualification, even if they resolve to the same function value.

# Answers

## A. The top-level member wins

`windows.newline` names the inherent top-level function. A trait-qualified path selects `PlatformApi.newline`. This keeps ordinary file declarations stable if a trait implementation is later added, but a caller who intended the trait witness must qualify it even when the function values are identical.

## B. Any collision is ambiguous

`windows.newline` is rejected whenever both an inherent declaration and a trait witness have that name. The caller must qualify the inherent or trait path. This treats all collisions uniformly with [Question 029](029_Composite_trait_member_collisions.md), but adding a trait implementation can break previously valid ordinary member access.

## C. Identical function values merge; different values are ambiguous

`windows.newline` works in the example because the witness is exactly the top-level function value. If the witness refers to a different function, the short path is ambiguous. This avoids noise in the common self-witness case, but the compiler must define function-value identity, including generic instantiations and aliases.

# Selected answer

**B.** A top-level member and a trait witness with the same name make ordinary `file.name` access ambiguous, even if they refer to the same function value. The caller must explicitly qualify which member origin is intended. The qualification syntax, including how an `impl` block refers to an existing top-level function without ambiguity, remains open.
