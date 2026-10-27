# Question

How can a root file require its Windows and Linux implementations to provide one stable public API, so platform re-export does not accidentally expose different functions or signatures?

The user suggested a trait or interface that each implementation must satisfy. The specific relationship between a trait on a type and an interface for an imported file namespace is still open. [Question 022, answer C](022_Conditional_module_files.md) selects conditional re-export, and [Question 023, answer E](023_Compilation_unit_size.md) makes each file an importable struct-like namespace.

## Illustrative desired effect

`platform/root.fl` (all syntax provisional):

```flang
interface PlatformApi {
    fn newline() -> String;
    fn write_console(text: String);
}

comp if target.os == .windows {
    #reexport_as(#import("windows.fl"), PlatformApi);
} else {
    #reexport_as(#import("linux.fl"), PlatformApi);
}
```

The compiler checks that the selected implementation has both functions with compatible signatures. Only members named by `PlatformApi` are exposed through `platform`, even if a platform file defines extra helpers. Whether interfaces may also specify constants, types, generics, or associated types is open.

**Cross-target limit:** [Question 018, answer A](018_Comptime_if_branch_checking.md) checks only the active compile-time branch. A Windows build would check `windows.fl`, and a Linux build would check `linux.fl`; one build would not automatically prove that both satisfy the interface. Checking both would require builds for both targets or a separate compiler feature that analyzes inactive implementations under their own target environments.

# Research

- [OCaml manual: module types and signatures](https://ocaml.org/manual/5.3/modtypes.html) — A signature specifies value, type, and module components. A structure matches it when it provides all required names with compatible types; it may provide more. This closely matches checking an imported namespace against a declared API.
- [Rust Reference: traits](https://doc.rust-lang.org/reference/items/traits.html) — A trait is an interface implemented by types, with associated functions, constants, and types. Applying this model directly to imported file namespaces would be an additional flang design choice.
- [Go specification: interface types](https://go.dev/ref/spec#Interface_types) — A type implements an interface through its method set without an explicit `implements` declaration. This is a precedent for structural conformance, though Go interfaces are for values and methods rather than file namespaces.
- [Zig language reference: `@hasDecl`](https://ziglang.org/documentation/master/#hasDecl) — Compile-time code can ask whether a namespace has a public declaration. The reference cautions that ad hoc presence checks can silently change behavior after renames, motivating a single explicit contract check.
- [Question 018](018_Comptime_if_branch_checking.md) — Establishes that inactive compile-time branches are not type-checked in the current build.

# Answers

## A. A module interface checks and projects the imported namespace

The `interface PlatformApi` sketch above names required members. Each selected implementation is checked against it, and the root re-exports only those members. Extra platform-specific functions may exist internally without changing the public API. This is a direct fit for file namespaces, but introduces a new interface construct or extends an existing one to namespaces. Diagnostics must identify missing and mismatched members.

## B. A trait is implemented by a platform type

```flang
trait PlatformApi {
    fn newline() -> String;
    fn write_console(text: String);
}

// Each platform file defines a type implementing PlatformApi.
```

The root selects the implementation type and exposes trait operations. This reuses a general type abstraction if flang has traits, and can support stateful implementations. It adds wrapper types or forwarding where the platform API naturally consists of free functions in a file namespace. Whether static trait methods alone can be called through a selected type needs rules.

## C. A compile-time structural assertion checks each namespace

```flang
comp windows = #import("windows.fl");
comp assert_namespace_matches(windows, PlatformApi);
```

A library can define its own check with introspection, without a dedicated module-interface feature. This is flexible for unusual APIs, but diagnostics and the exact set of exported names depend on user code; it may not automatically hide extra platform-specific members.

## D. Re-export without a shared contract

Each target exposes whatever its selected implementation provides. This allows intentionally different platform APIs and keeps the language simpler, but a caller can compile on one target and fail on another due to a missing or differently typed function.

# Selected answer

Pending. The user is interested in an interface or trait requirement, but has not selected the mechanism or whether the interface filters extra public members.
