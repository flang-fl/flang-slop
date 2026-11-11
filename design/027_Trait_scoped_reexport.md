# Question

How does a root file expose the shared trait functions of its selected platform file under the root's own top-level names?

[Question 025, answer E](025_Platform_interface_conformance.md) says that `windows.fl`, `linux.fl`, and `root.fl` explicitly declare that they implement `PlatformApi`. [Question 022, answer C](022_Conditional_module_files.md) permits conditional re-export. The remaining choice is how the root's trait members refer to the selected file's members. All syntax here is provisional.

## Illustrative goal

```flang
// root.fl
impl PlatformApi for #this_file;

comp Selected = if target.os == .windows {
    #import("windows.fl")
} else {
    #import("linux.fl")
};

// What makes root.newline() and root.write_console(...) exist?
```

# Research

- [OCaml manual: including structure components](https://ocaml.org/manual/5.3/modules.html) — `include` re-exports another structure's definitions in the current structure. It provides a precedent for bringing selected members into a root namespace without hand-written forwarding functions. A signature can separately restrict the visible components.
- [OCaml manual: module signatures](https://ocaml.org/manual/5.3/modtypes.html) — A structure can have extra members while matching a signature. This supports a trait-scoped projection that exposes only the required API, though OCaml signatures are not flang traits.
- [Rust Reference: use declarations](https://doc.rust-lang.org/reference/items/use-declarations.html) — Public `use` creates re-exported name bindings, including selected names and glob imports. These are aliases rather than new forwarding function bodies, and Rust's `use` does not itself synthesize a trait implementation.
- [OCaml manual: first-class modules](https://ocaml.org/manual/5.3/firstclassmodules.html) — A module can be packed as a value constrained by a module type. This is relevant to a compile-time binding of either platform file behind one interface, though OCaml uses explicit packing and unpacking.
- [Rust Reference: `impl Trait`](https://doc.rust-lang.org/reference/types/impl-trait.html) — Rust's opaque `impl Trait` denotes a concrete hidden type, and return branches must resolve to one concrete type. Flang's proposed `comp api: impl PlatformApi` has a compile-time-selected file, so the annotation could be a static trait constraint; its exact meaning is a separate typing question.
- [Question 016](016_Compiler_introspection_and_messages.md) — User-defined declaration generation at compile time is still undecided. Option C depends on enough compile-time reflection and emission to implement it in a library.

# Answers

## A. A built-in trait-scoped re-export

```flang
#reexport_impl(PlatformApi, from = Selected);
```

The compiler checks `Selected` against `PlatformApi` and introduces exactly the trait's required top-level names into the root. Their signatures and behavior come from `Selected`; extra public helpers are not exposed through this operation. The root's explicit `impl PlatformApi` declaration is checked against the result. This is concise and gives uniform diagnostics, but adds a dedicated language operation. It still needs collision and generic-signature rules.

## B. Explicit forwarding functions

```flang
fn newline() -> String { Selected.newline() }
fn write_console(text: String) { Selected.write_console(text); }
```

The root writes every function. It may instead put `comp if` inside each function to choose a platform. This needs no declaration-generation facility and allows adaptation of platform APIs, but repeats signatures and forwarding code. The root's trait declaration detects omissions or mismatches.

## C. A user-defined compile-time forwarding utility

```flang
comp forward_trait(PlatformApi, from = Selected, into = #this_file);
```

A standard-library or user-defined compile-time function inspects `PlatformApi` and creates forwarding declarations in the root. This avoids a special built-in for this exact use case and can be customized. It requires compile-time code to create declarations, with rules for expansion order, name conflicts, generated diagnostics, and visibility. Question 016 has not decided whether those abilities exist.

## D. Re-export the entire selected file namespace

```flang
#reexport(Selected);
```

The root's trait check verifies required functions, while re-export publishes everything that is otherwise eligible. This is simple and resembles an ordinary namespace include, but platform-only helpers may appear on one target and not another. A separate visibility rule would be needed if the public API must contain only the shared trait members.

## E. Bind the selected file's function values into the root's trait implementation

```flang
comp api: impl PlatformApi = comp if target.os == .windows {
    #import("windows.fl")
} else {
    #import("linux.fl")
};

impl PlatformApi for #File {
    comp newline = api.newline;
    comp write_console = api.write_console;
}
```

This is the user's proposed shape. Functions are values, so the root names the selected implementation's function values directly. No wrapper body or declaration generator is needed. The root still lists each member explicitly, and its `impl` is checked against `PlatformApi`. Selecting the file at compile time permits direct calls to the chosen functions in an AOT build. This does not decide whether `impl PlatformApi` in a binding is an opaque type, a compile-time existential, or a constraint on a known concrete file type; the syntax and precise type rule remain open.

# Selected answer

**E, qualified.** The root selects a platform file at compile time and explicitly binds that file's function values as members of its own `PlatformApi` implementation. No forwarding function bodies are needed. The user has not chosen exact binding syntax or the meaning of `impl PlatformApi` as a binding type. A future utility for generating these bindings remains possible, but is not needed for this pattern.
