# Question

How can a root file require its Windows and Linux implementations to provide one stable public API, so platform re-export does not accidentally expose different functions or signatures?

The user suggested a trait or interface that each implementation must satisfy. [Question 022, answer C](022_Conditional_module_files.md) selects conditional re-export, [Question 023, answer E](023_Compilation_unit_size.md) makes each file an importable namespace, and [Question 026, answer A](026_File_as_zero_sized_struct.md) makes each file a zero-sized struct. The answers below compare ways to apply a shared API contract.

The intended behavior is that a root file exposes `newline` and `write_console` on either target with the same signatures. Whether a trait also requires constants, types, generics, or associated types remains open. The selected approach is illustrated in answer E; the alternative projection approach is answer A.

**Cross-target limit:** [Question 018, answer A](018_Comptime_if_branch_checking.md) checks only the active compile-time branch. A Windows build would check `windows.fl`, and a Linux build would check `linux.fl`; one build would not automatically prove that both satisfy the interface. Checking both would require builds for both targets or a separate compiler feature that analyzes inactive implementations under their own target environments.

# Research

- [OCaml manual: module types and signatures](https://ocaml.org/manual/5.3/modtypes.html) — A signature specifies value, type, and module components. A structure matches it when it provides all required names with compatible types; it may provide more. This closely matches checking an imported namespace against a declared API.
- [Rust Reference: traits](https://doc.rust-lang.org/reference/items/traits.html) — A trait is an interface implemented by types, with associated functions, constants, and types. Applying this model directly to imported file namespaces would be an additional flang design choice.
- [Rust Reference: implementations](https://doc.rust-lang.org/reference/items/implementations.html) — A type's implementation can contain associated functions callable without an instance. This supports the user's idea that top-level file functions could fulfill trait functions if a file is treated as a type or struct. Rust itself does not make imported files implement traits.
- [OCaml manual: structures](https://ocaml.org/manual/5.3/moduleexamples.html) — A module structure groups definitions and exposes them through a qualified name. This is a precedent for treating a file's top-level declarations as one namespace, although an OCaml structure is not literally a record or struct value.
- [Go specification: interface types](https://go.dev/ref/spec#Interface_types) — A type implements an interface through its method set without an explicit `implements` declaration. This is a precedent for structural conformance, though Go interfaces are for values and methods rather than file namespaces.
- [Zig language reference: `@hasDecl`](https://ziglang.org/documentation/master/#hasDecl) — Compile-time code can ask whether a namespace has a public declaration. The reference cautions that ad hoc presence checks can silently change behavior after renames, motivating a single explicit contract check.
- [Question 018](018_Comptime_if_branch_checking.md) — Establishes that inactive compile-time branches are not type-checked in the current build.

# Answers

## A. A module interface checks and projects the imported namespace

An `interface PlatformApi` declaration names required members. Each selected implementation is checked against it, and the root re-exports only those members. Extra platform-specific functions may exist internally without changing the public API. This is a direct fit for file namespaces, but introduces a new interface construct or extends an existing one to namespaces. Diagnostics must identify missing and mismatched members.

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

## E. Each imported file is a struct-like entity that can implement a trait

An imported file is itself an entity to which a trait can apply. It explicitly declares that it implements the trait and binds a witness for every required member under [Question 030, answer B](030_Trait_witness_binding.md). Existing top-level function values can serve as those witnesses. The platform files need no wrapper types. The root file explicitly implements the trait too, binding function values from the selected implementation under [Question 027, answer E](027_Trait_scoped_reexport.md). For example (all syntax provisional):

```flang
trait PlatformApi {
    fn newline() -> String;
    fn write_console(text: String);
}

// windows.fl
fn newline() -> String { "\r\n" }
fn write_console(text: String) { /* Windows implementation */ }
impl PlatformApi for #this_file {
    comp newline = #this_file.(inherent).newline;
    comp write_console = #this_file.(inherent).write_console;
}

// root.fl
comp platform: impl PlatformApi = comp if target.os == .windows {
    #import("windows.fl")
} else {
    #import("linux.fl")
};

impl PlatformApi for #this_file {
    comp newline = platform.newline;
    comp write_console = platform.write_console;
}
```

Here the root binds function values from the selected file, with no wrapper function bodies. The exact syntax for function-value members and how `impl PlatformApi` constrains the selected file are still open.

The file's basic zero-sized struct representation is selected in [Question 026](026_File_as_zero_sized_struct.md). Trait conformance is explicitly declared, as the user selected. A single target build still checks only its selected implementation under [Question 018](018_Comptime_if_branch_checking.md). Dynamic dispatch over file values is a future question; a zero-sized file value can still require dispatch metadata.

The illustrative `PlatformApi` functions have no receiver. In Rust, such associated functions are not dispatched through a trait object; Rust's dynamic trait calls use methods with a receiver. Flang could later allow a runtime interface value to carry a table of receiver-free functions, or require receiver-bearing methods for dynamic dispatch. That choice is not implied by selecting this option.

# Selected answer

**E, qualified.** Imported files are zero-sized structs under [Question 026](026_File_as_zero_sized_struct.md). Each platform file must explicitly declare that it implements the shared trait and bind witnesses for its required members under [Question 030, answer B](030_Trait_witness_binding.md). The root file must explicitly implement the same trait, binding the selected platform's function values as its own members under [Question 027, answer E](027_Trait_scoped_reexport.md). The syntax, extra-member visibility, and dynamic-dispatch rules remain open. Under [Question 018](018_Comptime_if_branch_checking.md), an ordinary build checks only its selected platform branch.
