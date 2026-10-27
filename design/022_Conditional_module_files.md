# Question

How can a library expose one stable module name while selecting its implementation from different source files during compilation?

The user first considered a rare compile-time file inclusion directive, perhaps spelled `#include("file")`, inside a compile-time conditional. They then selected answer C: import a separate implementation unit and re-export its public members through a stable module name. [Question 023, answer E](023_Compilation_unit_size.md) establishes file-level imports with explicit root-file composition for larger public namespaces. The `#` marker, `#import` versus `#load`, and conditional spelling remain open ([Questions 017](017_Comptime_if_syntax.md) and [019](019_Builtin_sigil.md)).

## Selected direction: conditional re-export

`platform.fl` (illustrative syntax):

```flang
comp if target.os == .windows {
    #reexport(#import("platform/windows.fl"));
} else {
    #reexport(#import("platform/linux.fl"));
}
```

The proposed `#import` would produce a struct-like namespace with members for the declarations in the imported file or module. `#reexport` would expose its public members through `platform`, so a caller uses `platform.newline()` regardless of target. The implementation files are separate importable units; the chosen one is not textually pasted into `platform.fl`.

## Earlier inclusion sketch (not selected)

`platform.flang`:

```flang
comp if target.os == .windows {
    #include("platform/windows.flang");
} else {
    #include("platform/linux.flang");
}
```

`platform/windows.flang`:

```flang
fn newline() -> String { "\r\n" }
fn write_console(text: String) { windows_write_console(text); }
```

`platform/linux.flang`:

```flang
fn newline() -> String { "\n" }
fn write_console(text: String) { linux_write_console(text); }
```

`main.flang`:

```flang
import platform;

fn main() {
    platform.write_console("hello" + platform.newline());
}
```

The selected file's declarations would be exposed as `platform.newline` and `platform.write_console`; callers would not import `windows` or `linux` directly. The example assumes two targets and leaves handling of other targets open. The `String`, `import`, and target-query spellings are illustrative.

# Research

- [GCC manual: C `#include` operation](https://gcc.gnu.org/onlinedocs/gcc-7.2.0/cpp/Include-Operation.html) — C preprocessing inserts the included file's token stream into the including file. It can even include incomplete syntactic fragments, which is more permissive than this proposal necessarily needs.
- [Clang documentation: modules](https://clang.llvm.org/docs/Modules.html) — Contrasts textual inclusion with semantic imports; repeated parsing and macro-sensitive inclusion are costs of the textual C model. This motivates deciding whether a flang `#include` would merge declarations without literal token substitution.
- [Zig language reference: compilation model and `@import`](https://ziglang.org/documentation/master/#Compilation-Model) — Shows a compile-time choice that discovers a platform-specific file; Zig's imports still treat files as separate namespaces. This is similar file selection but differs from merging declarations into one public module.
- [Odin language overview: packages](https://odin-lang.org/docs/overview/#packages) — Multiple files in one package share a package name. This is a precedent for several files contributing to one module without using C-style textual inclusion.
- [Question 020](020_Comptime_if_file_scope.md) — Asks whether compile-time conditionals directly insert declarations at file scope; a selected implementation file is another way to present one stable API.

# Answers

## A. Include complete declarations into the current module

`#include("platform/windows.flang")` parses the chosen file as flang declarations and adds them to `platform`'s scope. The file must contain complete declarations. This gives the requested public name and avoids raw token substitution, while requiring rules for duplicate inclusion, relative paths, cycles, and declaration visibility.

## B. Include the chosen file textually, like C

The selected file's tokens are inserted at the directive site before parsing that part of the module. This can include arbitrary fragments and is very flexible. It also makes parsing, formatting, diagnostics, and inclusion order more sensitive to surrounding source text.

## C. Choose a separate importable unit and re-export it as `platform`

Each platform implementation is a separate importable unit. `platform.fl` selects one and forwards its public declarations. This preserves a boundary between the public module and implementation, but needs rules for re-export collisions, visibility, and what counts as an importable unit. The user proposes a `#reexport(#import("windows.fl"))`-like form; exact spelling is open.

# Selected answer

**C.** Select a separate importable implementation and re-export its public members under a stable module name. The user proposes that import produce a struct-like namespace and that a re-export operation forward its members. `#reexport(#import("windows.fl"))` is illustrative: `#import` versus `#load` and the `#` sigil are undecided. [Question 023, answer E](023_Compilation_unit_size.md) establishes file-level imports and explicit root-file composition for larger public namespaces.
