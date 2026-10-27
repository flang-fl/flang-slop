# Question

Can a source file restrict who may import it directly while letting a root file re-export selected declarations to outside callers? If so, is the boundary a directory, a logical library, or an explicitly named root file?

The user suggested a `#no_import`-like marker on sibling files, or a publicity scope limited to their folder that a root's re-export could widen. The wording is provisional: a literal ban on *all* imports would also prevent the root from importing the file. [Question 023, answer E](023_Compilation_unit_size.md) establishes that files are importable namespaces and root files may aggregate them by explicit code.

## Illustrative behavior

```text
math/
  root.fl
  vector.fl
app.fl
```

`math/vector.fl`:

```flang
#import_scope(.directory); // illustrative: direct imports allowed from math/ only
fn dot(a: f32, b: f32) -> f32 { a * b }
```

`math/root.fl`:

```flang
#reexport(#import("vector.fl"));
```

`app.fl`:

```flang
comp math = #import("math/root.fl");
// math.dot is available through the root's public path

comp vector = #import("math/vector.fl"); // direct import rejected
```

Whether every declaration in `vector.fl` is re-exportable, or only explicitly public ones, is a separate visibility question. The snippets do not settle the `#` marker or `#import` spelling.

# Research

- [Go documentation: `internal` packages](https://go.dev/doc/modules/layout#package-or-command-with-supporting-packages) — An `internal` directory restricts which code may import its packages, allowing an outer package to expose a stable API while keeping implementation packages inaccessible to external users. This is directory-based import visibility, though the exact boundary differs from the proposed same-folder rule.
- [Rust Reference: visibility and re-exports](https://doc.rust-lang.org/reference/visibility-and-privacy.html#re-exporting-and-visibility) — A public re-export can expose an item through a public path even when its original module path is private. This is a precedent for access being checked through the chosen path rather than permanently attached to the item's original file.
- [Odin overview: exported names and `private` attributes](https://odin-lang.org/docs/overview/#exported-names) — Odin distinguishes package-private and file-private declarations. This is a precedent for scoped visibility, though it governs declarations rather than direct import of an individual file.
- [Zig language reference: compilation model](https://ziglang.org/documentation/master/#Compilation-Model) — A Zig module has a root source file and may contain other source files. This is a precedent for identifying a group through its root, though Zig's import visibility rules are not the proposal here.
- [Question 022, answer C](022_Conditional_module_files.md) — Establishes the re-export operation being considered.

# Answers

## A. Directory-scoped direct imports; root may re-export

The example above is valid: `root.fl` can import `vector.fl` because it is in `math/`; `app.fl` cannot import `vector.fl` directly, but may use names re-exported by `root.fl`. This matches the user's folder-scope idea.

**Benefits:** The rule is visible from the file tree, needs no library manifest or owner declaration, and makes a small folder a natural encapsulation boundary. A compiler can decide access from source paths alone.

**Costs:** Moving a file changes its import permissions. A root in `math/` could not import a restricted implementation in `math/platform/` under a literal same-folder rule; the language would need an ancestor-directory or subtree variant. Generated files, symlinks, and alternate path spellings also need a canonical-directory rule. Directory layout may become an API design constraint rather than just an organization choice. Go's `internal` rule is directory-based but uses an ancestor subtree, not literal same-folder access.

## B. Logical-library-scoped direct imports; root may re-export

A file can be imported from anywhere within its declared library, regardless of directory. The root may publish names through a re-export.

**Benefits:** Files can move between folders without changing access, and a root can use implementation files in subdirectories. Tests, generated files, and platform code can share internals if they belong to the same library. This aligns access with the intended API boundary rather than physical layout.

**Costs:** Flang must define library identity and membership, perhaps through a build manifest or root declaration. A large library gives many files access to internals, which can invite unintended coupling. Moving code *between libraries* changes access even when the path stays similar. Error messages need to explain library membership, which may be less obvious than a directory path.

## C. A hard `#no_import` marker with a special load operation

Ordinary `#import("vector.fl")` is forbidden even to `root.fl`; a privileged operation such as `#load` can consume it for re-export. This gives a very explicit implementation-file category, but creates two ways to load code and raises questions about which parts of the loaded file are callable internally.

## D. All files are importable; visibility is only on declarations

Outside code may import `vector.fl`, but sees no members unless they are publicly visible. The root can explicitly re-export eligible members. This keeps import rules simple, but does not prevent external code from depending directly on members that the file does expose.

## E. A root file owns a logical library scope

```flang
// math/platform/windows.fl — illustrative syntax
#import_scope(root = "../root.fl");
```

Only the named root and files it authorizes can directly import this file; the root may re-export selected names. This makes the user's “mother file” the explicit boundary, even when implementation files are in subdirectories. It avoids a separate library manifest but requires each restricted file to name or derive an owner, and rules are needed for nested roots and circular ownership. This is a design combination of A and B, not a claim about a referenced language.

An alternative E spelling lets the root declare membership once, while implementation files refer to that logical library:

```text
math/
  root.fl
  platform/windows.fl
  platform/linux.fl
app.fl
```

`math/root.fl`:

```flang
#library("math", files = #source_files_recursive("."));

comp if target.os == .windows {
    #reexport(#import("platform/windows.fl"));
} else {
    #reexport(#import("platform/linux.fl"));
}
```

`math/platform/windows.fl`:

```flang
#import_scope(.library); // direct imports allowed only from files in "math"
fn newline() -> String { "\r\n" }
```

`app.fl`:

```flang
comp math = #import("math/root.fl"); // allowed
comp windows = #import("math/platform/windows.fl"); // rejected
```

An outside caller could use `math.newline()` through the root's re-export. The `files` set establishes membership for access checks; it would not by itself import or type-check every listed file. The root could compute a different set, including files outside its directory, so membership is defined by the root rather than by physical location. `#library`, `#source_files_recursive`, and `#import_scope` are illustrative names, not selected syntax. Whether the compiler accepts members listed by multiple roots and how it checks cycles remain open.

# Selected answer

**E, qualified.** The user likes a logical library scope owned by a root file: implementation files can restrict direct imports to members of that library, while the root can re-export selected names for outside callers. The root may compute membership programmatically. The exact directives, membership verification, and visibility of individual declarations remain open.
