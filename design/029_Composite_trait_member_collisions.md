# Question

When a compile-time view uses `impl TraitA + TraitB` and both traits expose a member with the same name, what does `view.name` mean?

[Question 028, answer B](028_Comptime_impl_binding.md) says a compile-time `impl` binding exposes the combined members of all listed traits, while hiding unrelated concrete members. Overlapping names need a rule even if the traits require identical signatures. This question concerns name lookup through that combined view, not trait implementation syntax in general.

## Example

```flang
trait Logger { fn flush(); }
trait Storage { fn flush(); }

comp service: impl Logger + Storage = #import("service.fl");
service.flush(); // Which trait member?
```

The two trait requirements might be implemented by the same function value or by distinct functions. The answer should say whether that difference affects lookup. All syntax is provisional.

# Research

- [Rust book: disambiguating same-name trait methods and associated functions](https://doc.rust-lang.org/book/ch20-02-advanced-traits.html#disambiguating-between-methods-with-the-same-name) — Rust permits different traits to provide the same method name and uses explicit qualification when the intended member is ambiguous. Receiver-free associated functions may need the implementer and trait named together.
- [Rust Reference: `impl Trait` bounds](https://doc.rust-lang.org/reference/types/impl-trait.html) — Multiple trait bounds can constrain one hidden concrete type. This is a precedent for the `+` composition syntax, though flang's compile-time view has its own member lookup rules.
- [OCaml manual: module signatures](https://ocaml.org/manual/5.3/modtypes.html) — Signatures specify named components; combining interface requirements raises the same broad question of duplicate component names. The manual is background for signature composition, not a direct prescription for flang's `+` behavior.

# Answers

## A. Ambiguous names require trait qualification

`service.flush` is rejected when both traits contribute `flush`, even if signatures or implementations happen to match. A qualified access such as `service.(Logger).flush` or `service.(Storage).flush` chooses one requirement. This is predictable and preserves each trait's identity, but adds syntax at collision sites.

## B. Merge identical members; qualify distinct ones

If both traits' `flush` requirements have compatible signatures and resolve to the same function value, `service.flush` is accepted. Otherwise the caller must qualify the trait. This reduces noise when composition repeats a shared operation, but the compiler must define what counts as the same function value and how generic signatures compare.

## C. Reject the composition when names overlap

`impl Logger + Storage` is invalid whenever both traits contain `flush`, regardless of implementation. This keeps lookup simple but prevents composition of independently designed traits with common names, even if the caller would only access them through qualified paths.

## D. Resolve overloads from call signatures or context

The compiler may pick `flush` based on argument and result types, requiring qualification only if overload resolution remains ambiguous. This allows same-name operations to coexist, but call meaning can change when signatures evolve and a function value such as `comp f = service.flush` may have no argument context.

# Selected answer

**A.** If two traits in a combined `impl TraitA + TraitB` view expose the same member name, unqualified access through the view is ambiguous and rejected, even when signatures or implementations match. The caller must qualify the trait. The exact qualification syntax remains open.
