# Question

What kinds of members may a trait require from a file or other implementing type?

The selected platform example uses only functions. [Question 025](025_Platform_interface_conformance.md) left constants, types, generics, and associated types open. [Question 030, answer B](030_Trait_witness_binding.md) requires an explicit witness for each required member, so widening the set of member kinds also widens what an `impl` block must be able to bind. All syntax below is provisional.

## Example

```flang
trait PlatformApi {
    fn newline() -> String;
    comp path_separator: String;
    type FileHandle;
}

impl PlatformApi for #File {
    comp newline = #File.(inherent).newline;
    comp path_separator = "\\";
    type FileHandle = WindowsHandle;
}
```

If `FileHandle` is abstract through the trait, a caller using `impl PlatformApi` could manipulate handles through trait operations without relying on their platform-specific representation. Whether type identities may differ across target builds is a separate compatibility question.

# Research

- [Rust Reference: traits](https://doc.rust-lang.org/reference/items/traits.html) — Rust traits can require associated functions, constants, and types. Required associated items are supplied by implementations; associated functions can be receiver-free or methods.
- [OCaml manual: module signatures](https://ocaml.org/manual/5.3/modtypes.html) — Signatures can specify values, types, exceptions, modules, and more. A matching structure may have additional components, while abstract type specifications hide implementation details.
- [Go specification: interface types](https://go.dev/ref/spec#Interface_types) — Go's basic interfaces describe method sets; embedded interfaces compose requirements. This is a narrower precedent focused on methods rather than constants or associated types.
- [Question 028](028_Comptime_impl_binding.md) — A compile-time `impl Trait` view exposes only the declared trait members, so the chosen member kinds also determine what that view can reveal.

# Answers

## A. Functions only

Traits require callable operations, including receiver-free functions and perhaps methods once method syntax is decided. Constants and types stay ordinary file members and cannot be abstracted through a trait view. This keeps witness checking and generic use simpler, but platform APIs with different handle types need another abstraction mechanism.

## B. Functions and constant values

Traits may require functions and compile-time or immutable values of declared types. A platform path separator can be accessed through the trait view, but a platform-specific type such as `FileHandle` remains outside it. This covers configuration-style APIs without associated-type rules.

## C. Functions, constants, and associated types

Traits may require all three categories, as in the example. A file's `impl` binds each required function value, constant value, and type. The trait view hides concrete representations of abstract associated types. This supports richer cross-platform APIs, but needs rules for type equality, generic constraints, and what callers may do with an abstract type.

## D. Any importable declaration kind

A trait may describe nested namespaces, other traits, compile-time functions, and future declaration forms as well as functions, constants, and types. This treats traits as general file-shape contracts, but makes matching, visibility, and diagnostics substantially broader. It also risks binding the trait system to every future declaration feature.

# Selected answer

**D.** A trait may require any importable declaration kind, including functions, constants, types, nested namespaces, other traits, compile-time functions, and future declaration forms. Implementations must explicitly bind witnesses for required members under [Question 030](030_Trait_witness_binding.md). The matching rules for each kind, including nested declarations and abstract types, remain open.
