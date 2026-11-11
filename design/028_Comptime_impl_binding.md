# Question

In a binding such as `comp api: impl PlatformApi = comp if ...`, what does the `impl PlatformApi` annotation mean?

[Question 027, answer E](027_Trait_scoped_reexport.md) selects a platform file at compile time and binds its function values into the root's trait implementation. The `comp if` chooses one concrete file for each target build under [Question 018](018_Comptime_if_branch_checking.md). The annotation could merely check a trait bound, restrict which members compile-time code may access, or create a type-erased interface value. This question does not settle dynamic dispatch in general.

## Example

```flang
comp api: impl PlatformApi = comp if target.os == .windows {
    #import("windows.fl")
} else {
    #import("linux.fl")
};

impl PlatformApi for #File {
    comp newline = api.newline;
}
```

If `windows.fl` also defines `windows_only_debug`, should `api.windows_only_debug` be accessible in this build? Would `api` still have the exact type of `windows.fl`, or only an interface type?

# Research

- [Rust Reference: `impl Trait`](https://doc.rust-lang.org/reference/types/impl-trait.html) — Rust's return-position `impl Trait` hides one concrete type behind trait operations; each return path must resolve to the same concrete type. A flang `comp if` selects one path before runtime, so this restriction need not apply in the same way, but the distinction between a hidden concrete type and an interface value matters.
- [OCaml manual: first-class modules](https://ocaml.org/manual/5.3/firstclassmodules.html) — A module can be packed as a value constrained by a module type and later unpacked. This is a precedent for a first-class interface value, though the user's binding is explicitly compile-time.
- [OCaml manual: module expression ascription](https://ocaml.org/manual/5.3/modules.html) — Ascribing a module type checks required components and hides components outside that module type. This is a precedent for option B's restricted view.
- [Rust Reference: trait objects](https://doc.rust-lang.org/reference/types/trait-object.html) — Runtime trait objects carry dispatch metadata. Such a representation would be a separate choice from merely requiring a compile-time trait bound.

# Answers

## A. Static trait constraint; concrete file type remains visible

The selected `comp if` branch determines `api`'s concrete file type for this build. The annotation checks that it implements `PlatformApi` but does not erase or hide that type. `api.windows_only_debug` is legal in a Windows build if otherwise visible. The trait's required functions can be bound without runtime dispatch. Code using platform-only members is target-specific and can fail on another target.

## B. Static trait view; extra members are hidden

The selected branch still determines the implementation at compile time, so calls can be statically resolved. The annotation restricts access through `api` to members of `PlatformApi`; `api.windows_only_debug` is rejected even on Windows. Multiple bounds, such as `impl PlatformApi + Foo`, expose the members of both traits. This keeps the root's code against the stated interfaces, but may need a separate syntax to recover the concrete file type when platform-specific operations are intentional. Name collisions between bounds need a rule.

## C. A type-erased interface value

The annotation packages the file behind an interface value with its own identity. `api` exposes only trait members, and the representation could eventually be used at runtime. This unifies static selection with future dynamic dispatch, but commits the language to rules for storing, copying, and dispatching interface values. A compiler might optimize away runtime metadata when `api` is known at compile time, but that is an optimization rather than the language rule.

# Selected answer

**B, qualified.** `comp api: impl PlatformApi` is decided at compile time. The selected implementation must satisfy `PlatformApi`, and only that trait's members are exposed through `api`. `impl PlatformApi + Foo` requires both traits and exposes members from both. This does not itself imply runtime dynamic dispatch. Exact type identity, access to hidden concrete members, and member-name collision rules remain open.
