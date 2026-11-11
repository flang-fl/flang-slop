# Question

When a trait requires a nested namespace, how does an implementation prove that the nested namespace has the required members?

[Question 034, answer D](034_Trait_member_kinds.md) allows traits to require any importable declaration kind, including nested namespaces. [Question 030, answer B](030_Trait_witness_binding.md) still requires an explicit witness for each required member. The open point is whether the nested witness must itself explicitly implement a named trait, whether the parent implementation can ask for recursive structural checking, or whether only one exact namespace identity is accepted. All syntax is provisional.

## Example

```flang
trait FileOps {
    type Handle;
    fn open(path: String) -> Handle;
}

trait PlatformApi {
    namespace files: FileOps;
}

impl PlatformApi for #File {
    comp files = #import("platform_files.fl");
}
```

Should `platform_files.fl` have to declare `impl FileOps` itself? Or is it enough for the parent compiler check to inspect its members?

# Research

- [OCaml manual: module type specifications](https://ocaml.org/manual/5.3/modtypes.html) — A module signature can require a nested module with a module type, and a structure matches a signature when it supplies all specified names with compatible types. This is a precedent for recursively checking a nested namespace's shape.
- [OCaml manual: module ascription](https://ocaml.org/manual/5.3/modules.html) — Ascription checks a module against a module type and hides extra components through that view, analogous to a nested trait view.
- [Rust Reference: trait associated types and bounds](https://doc.rust-lang.org/reference/items/associated-items.html#associated-types) — An associated type can have trait bounds, requiring the chosen type to satisfy another trait. This is a precedent for a nested named requirement, though Rust associated types are not file namespaces.
- [Question 025](025_Platform_interface_conformance.md) — The project chose explicit trait implementation declarations for files, which may or may not also be required for nested namespace witnesses.

# Answers

## A. The nested namespace must explicitly implement its named trait

`platform_files.fl` declares `impl FileOps` and binds all required witnesses. The parent binds that file as `files`; the compiler checks the child trait implementation. This keeps conformance explicit at every level and reuses the existing trait rules. It requires a child file that only serves one parent to carry its own `impl` block.

## B. The parent can request structural checking of the nested namespace

The parent explicitly binds `files`; the compiler recursively checks that the chosen namespace supplies `FileOps` members, even if `platform_files.fl` does not declare `impl FileOps`. This avoids child boilerplate and supports imported namespaces outside the author's control, but creates a second, implicit conformance path alongside [Question 025](025_Platform_interface_conformance.md).

## C. The trait requires one exact nested namespace identity

The trait names a specific nested file or namespace identity, and the witness must refer to that exact entity. This is simple to compare and preserves identity, but does not allow Windows and Linux files with different internal types to provide interchangeable nested APIs.

# Selected answer

**A.** A nested namespace bound as a trait witness must explicitly declare that it implements the required named trait. In the example, `platform_files.fl` declares `impl FileOps` and binds its required witnesses; the parent `impl PlatformApi` binds that file as `files`, and the compiler checks its declared `FileOps` implementation.
