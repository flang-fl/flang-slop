# Question

When a compile-time conditional's condition is known during compilation, what checks apply to the unchosen branch? The `comp if` spelling in this example is provisional; [Question 017](017_Comptime_if_syntax.md) now also considers `@if` and `#if`.

```flang
comp if target_has_simd {
    use_simd_only_type();
} else {
    use_portable_type();
}
```

Suppose `target_has_simd` is false and `use_simd_only_type` does not exist on this target. Must the program still fail? The spelling is being reconsidered in [Question 017](017_Comptime_if_syntax.md).

# Research

- [D language specification: `static if`](https://dlang.org/spec/version.html#staticif) — An unchosen branch must be syntactically valid but need not be semantically valid. This allows target-specific names to appear in code that is not selected.
- [Zig language reference: compile-time expressions](https://ziglang.org/documentation/master/#Compile-Time-Expressions) — Zig describes compile-time known `if` conditions as selecting one branch while skipping analysis of the other. Its compile-time evaluation rules also reject runtime-dependent values where compilation needs a known result.
- [Question 017](017_Comptime_if_syntax.md) — Explores spellings for an explicit compile-time conditional.

# Answers

## A. Parse both branches; analyze only the chosen branch

The unchosen branch must have valid syntax, but unknown names and type errors inside it are ignored for this compilation. This supports platform-specific code. For example, a Windows-only function can appear in the Windows branch without needing a declaration on Linux. Because both branches form syntax trees, a formatter and editor can still understand their structure. The cost is that errors in dormant code may appear only when a configuration selects it. This is the approach specified for D's `static if` and described for Zig's compile-time known `if`.

## B. Parse and type-check both branches

The compiler rejects unknown names or type errors in either branch, even when one cannot execute in this compilation. Its strongest benefit is finding mistakes in both implementations in one build: a typo in the other platform's branch is caught immediately. It also makes compiler diagnostics less dependent on build configuration. The cost is that target-specific APIs or types would need declarations on every target, even when never used there; a separate mechanism would be needed to express conditional availability.

## C. Skip even parsing the unchosen branch

Only the chosen branch needs valid syntax. This is useful if conditional regions may contain foreign-language text, generated fragments, or syntax introduced by a compiler plugin. The cost is that the compiler still needs a way to find the end of the skipped branch, and ordinary parsers, formatters, and editors cannot reliably understand it. Syntax errors may remain hidden until another configuration selects that branch. This would move `comp if` closer to a preprocessor boundary than to an ordinary `if` block.

# Selected answer

**A.** Parse both branches, but name-resolve and type-check only the active branch. The inactive branch must be valid flang syntax; errors that depend on declarations or types are reported only if that branch is selected. This does not yet decide whether a compile-time conditional may introduce declarations at file scope or whether its braces create a normal block scope.
