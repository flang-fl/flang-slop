# Question

What size source unit should `#import` (or `#load`) identify: one file, a directory of files, an explicitly defined group, or an entire library? Can a single-file root programmatically assemble a larger public namespace?

In [Question 022, answer C](022_Conditional_module_files.md), an import is intended to produce a struct-like namespace whose public members can be re-exported. The user has not decided whether that namespace comes from a single file or a larger module. The syntax below is illustrative; the `#` marker and `import` versus `load` remain open.

There are three related but distinct boundaries: **import identity** (what a path names), **name scope** (which declarations share a namespace), and **compiler work unit** (what is parsed, checked, cached, or emitted together). They need not coincide. This question compares simple designs where they mostly coincide; a later question can separate them if that proves better. A compiler could also cache declarations or emit LLVM modules more finely than its source-level import unit.

# Research

- [Zig language reference: namespaces, compilation model, and source files](https://ziglang.org/documentation/master/#Compilation-Model) — Every Zig file has a struct-like namespace, while a Zig module is a collection of files with a root file. This demonstrates that file namespaces and module-level compilation identity can be different.
- [Zig language reference: `@import`](https://ziglang.org/documentation/master/#import) — Importing a source file returns its struct-like namespace. This is a direct precedent for the user's proposed file-level import result, though automatic directory aggregation would be an additional flang feature.
- [Odin language overview: packages](https://odin-lang.org/docs/overview/#packages) — An Odin package is a directory of files sharing a package declaration. An import refers to the package, and declarations in its files share the package namespace.
- [Go language specification: packages](https://go.dev/ref/spec#Packages) — A Go package consists of one or more source files whose declarations belong to the same package. The [Go modules reference](https://go.dev/ref/mod) separately groups packages for distribution, showing that package and distribution units can differ.
- [Rust Reference: crates and source files](https://doc.rust-lang.org/reference/crates-and-source-files.html) — A crate is Rust's unit of compilation and linking, and it contains a tree of module scopes that may span several files. This is a larger compilation boundary than a single file.
- [Question 022](022_Conditional_module_files.md) — Conditional re-export needs a clear answer about what an import returns.

# Answers

## A. One source file per importable unit

```flang
comp math = #import("math.fl");
```

The file directly supplies the namespace and can be checked or cached independently. It is easy to find where a member comes from and to select one platform file. Larger libraries need many imports or an explicit facade that re-exports several files; splitting a library into files also splits its namespace unless another mechanism merges them. Fine-grained caching is possible but not automatic.

## B. One directory package per importable unit

```flang
comp math = #import("math/");
```

All eligible files in `math/` contribute to one namespace. Related declarations can share names without a re-export facade, and adding a file can be simple. The file layout now defines package membership; adding or editing one file may force reconsidering a wider unit, and platform-specific files need a selection rule so both variants do not define the same names.

## C. An explicit multi-file module per importable unit

```flang
comp math = #import("math.module"); // module definition lists its source files
```

A module root or manifest selects files independently of the directory layout. This can keep the public import path stable while allowing a module to span directories and choose platform-specific files. It adds configuration and duplicate ways to find a declaration: the directory tree and the module definition. The compiler must define how conditional file lists affect module identity and caching.

## D. One whole library or crate per importable unit

```flang
comp graphics = #import("graphics"); // whole library with internal modules
```

The compiler handles the library as one large unit while internal modules organize names. This gives broad context for checking and optimization and a clear artifact boundary. A small change can invalidate a large unit unless the compiler uses finer-grained incremental analysis; independent compilation and parallelism across parts of the library are harder to expose.

## E. Files are importable units; a root file assembles a public module

Each source file retains its own namespace as in A. A designated root file imports and re-exports its sibling files, possibly by enumerating the directory at compile time:

```flang
// math/root.fl — syntax sketch, not a decision on `comp for` or built-in names
comp files = #source_files_in(#current_dir());
comp for path in files {
    comp if path != #current_file() && path.ends_with(".fl") {
        #reexport(#import(path));
    }
}
```

Callers import `math/root.fl` and see a combined `math` namespace, while individual files can still be imported directly where their visibility permits it ([Question 024](024_File_import_visibility.md)). This gets a customizable multi-file public API without making the directory itself the primitive import unit or requiring a separate manifest. **Aggregation happens only because the library developer wrote this loop.** A root file could instead explicitly list imports, filter the directory, or re-export nothing. For a root that does scan and re-export all siblings, adding or renaming a sibling can change its public API without editing the root; collisions between re-exported names need a rule. The compiler must track the directory listing as a build dependency and enumerate files deterministically, or incremental builds could become stale or order-dependent. Those are design consequences of the proposal, not claims about Zig or Odin.

# Selected answer

**E, qualified.** A source file is an importable unit and exposes a struct-like namespace to permitted callers. A library developer may explicitly write a root file that programmatically imports and re-exports other files, including by scanning its directory. The language does not automatically aggregate or re-export siblings. Who may import a file is [Question 024](024_File_import_visibility.md); the `#import`/`#load` spelling, directory-query API, re-export collision rules, and compiler cache or code-generation granularity remain open.
