# Question

After a file binds a function value as a trait witness, can callers write `file.function()` directly, or must they name the trait?

[Question 030, answer B](030_Trait_witness_binding.md) requires every witness inside an explicit `impl` block. [Question 027, answer E](027_Trait_scoped_reexport.md) uses such bindings in a root file to expose the selected platform API. The remaining rule is how callers find these members on a file's namespace. Visibility and export rules for individual declarations remain open under [Question 024](024_File_import_visibility.md).

## Example

```flang
// root.fl
impl PlatformApi for #File {
    comp newline = api.newline;
}

// caller.fl
comp platform = #import("root.fl");
platform.newline(); // Valid? If so, which lookup rule finds it?
```

# Research

- [Rust Reference: associated functions](https://doc.rust-lang.org/reference/items/associated-items.html#associated-functions-and-methods) — A trait's associated function can be called through the implementing type when lookup identifies the function, or through a fully qualified trait path. This is a precedent for option B, though Rust's file modules are not implementing types.
- [Rust book: disambiguating trait members](https://doc.rust-lang.org/book/ch20-02-advanced-traits.html#disambiguating-between-methods-with-the-same-name) — Same-name trait functions may require a qualified path. This matches the collision rule selected in [Question 029](029_Composite_trait_member_collisions.md).
- [OCaml manual: module signatures and ascription](https://ocaml.org/manual/5.3/modules.html) — A signature controls which components are visible through an ascribed module. This is background for keeping trait-view members separate from unrelated file members.

# Answers

## A. Trait qualification is always required

Witnesses belong only to their trait implementation. A caller uses an explicit trait-qualified path such as `platform.(PlatformApi).newline()`. `platform.newline()` works only if the file separately declares or re-exports a top-level `newline`. This makes provenance clear but adds another alias for the root's intended public API.

## B. File lookup includes trait witnesses when unambiguous

`platform.newline()` finds the unique accessible trait witness named `newline`. If two implemented traits contribute `newline`, the call requires trait qualification, following [Question 029](029_Composite_trait_member_collisions.md). The witness remains associated with its trait rather than becoming a separate inherent file declaration. A rule is still needed if the file also has an inherent top-level `newline`.

## C. Witness bindings become ordinary top-level file members

The `comp newline` inside `impl` also creates `platform.newline` as an inherent member. This makes the root API straightforward, but two traits using the same name now collide at declaration time, possibly even when neither is accessed. The language would need a way to avoid promotion for one of them.

# Selected answer

**B.** Ordinary file member lookup includes accessible trait witnesses when the member name is unambiguous. A witness remains associated with its trait and does not create a separate inherent file declaration. If multiple traits contribute the same name, access requires trait qualification under [Question 029](029_Composite_trait_member_collisions.md). A clash with an inherent top-level declaration remains open.
