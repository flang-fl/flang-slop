# Question

Can a compile-time conditional select declarations at file scope, and where do the selected names become visible?

The conditional's spelling is still open in [Question 017](017_Comptime_if_syntax.md); `comp if` below is only a placeholder. Both branches must parse, and only the selected branch is type-checked, per [Question 018, answer A](018_Comptime_if_branch_checking.md).

```flang
comp if target_has_simd {
    fn dot(a: Vec, b: Vec) -> f32 { simd_dot(a, b) }
} else {
    fn dot(a: Vec, b: Vec) -> f32 { portable_dot(a, b) }
}

fn main() {
    dot(left, right);
}
```

Should the call to `dot` in `main` find the selected declaration?

## User's simpler alternative

For a shared function signature with only a small body difference, the user proposes placing the compile-time conditional inside one function:

```flang
fn newline() -> &'static str {
    comp if windows {
        "\r\n"
    } else {
        "\n"
    }
}
```

This avoids any file-scope conditional if the compile-time conditional can yield a value ([Question 021](021_Comptime_if_values.md)). The `&'static str` spelling is the user's illustration; flang has not selected string or lifetime syntax. The question here is whether file scope should *also* support selecting whole declarations, such as types, imports, foreign bindings, or functions with entirely different implementations. Those could be useful, but they do not make the `newline` case require file-scope conditionals.

The two selected `fn` declarations need not have the same signature by construction: only the active branch is analyzed. A common signature is useful when callers should work unchanged across targets. Different signatures would make callers target-specific or require another common wrapper.

The user also proposed keeping wholly different implementations in separate files and selecting one as the contents of a common library module. [Question 022](022_Conditional_module_files.md) develops that example. This could remove much of the need for a file-scope conditional that directly encloses declarations.

# Research

- [D language specification: `static if`](https://dlang.org/spec/version.html#staticif) — D permits `static if` at module and other declaration scopes. Its selected branch introduces declarations into the surrounding scope, even when written with braces.
- [Rust Reference: conditional compilation](https://doc.rust-lang.org/reference/conditional-compilation.html) — Rust can select whole items with `#[cfg(...)]`, so a platform-specific function can be present in one build and absent in another. This achieves conditional file-level declarations without making an `if` block transparent.
- [Question 007, answer B](007_If_branch_values.md) — Ordinary flang blocks share one result rule; whether declaration scope also stays uniform is relevant here.
- [Question 022](022_Conditional_module_files.md) — An alternative way to select complete platform implementations from separate files.

# Answers

## A. Selected declarations enter the surrounding file scope

The `dot` call works. The conditional acts as a declaration selector, and inactive declarations do not exist in this build. This is concise for target-specific APIs, but its braces have different name-scoping behavior from ordinary blocks.

## B. Selected declarations remain inside a branch namespace

The selected `dot` exists, but must be accessed through an explicit namespace or export mechanism. This preserves a strong boundary around the branch, at the cost of extra naming machinery and less direct conditional declarations.

## C. Compile-time conditionals are not allowed at file scope

The example is invalid. File-level alternatives would use a separate conditional declaration feature, such as attributes on each `fn`. This keeps the conditional focused on expression and statement contexts, but creates another mechanism for a common platform-selection task.

## D. File-scope conditionals may select module operations, but not declare names directly

A file-scope conditional may choose an operation such as `#reexport(#import("windows.fl"))` ([Question 022, answer C](022_Conditional_module_files.md)). The selected implementation remains a separate importable unit; its declarations are not spliced into the conditional's block. This supports the user's platform-module example while keeping direct file-scope declaration injection unavailable. The exact syntax and whether other module operations are allowed remain open.

# Selected answer

Pending.
