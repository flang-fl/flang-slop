# Question

How should flang request that an `if` condition be evaluated during compilation?

In [Question 015, answer A](015_Runtime_control_of_comptime_state.md), a runtime condition may not control a `comp mut` update, while a compile-time condition may. The spelling below is provisional. Whether a compile-time `if` can also introduce declarations at file scope is a separate question.

# Research

- [Zig language reference: compile-time expressions](https://ziglang.org/documentation/master/#Compile-Time-Expressions) — `comptime` can force an expression, including its control flow, to be evaluated during compilation; a runtime-dependent condition then produces an error. The reference also says Zig implicitly inlines an `if` whose condition is known at compile time.
- [D language specification: `static if`](https://dlang.org/spec/version.html#staticif) — An explicit `static if` condition must be evaluable during compilation. Its unused branch need only be syntactically valid, and it can conditionally include declarations. D's branch scoping differs from ordinary `if`, so adopting the spelling would not require adopting all of its semantics.
- [Zig language reference: built-in functions](https://ziglang.org/documentation/master/#Builtin-Functions) — Zig reserves `@`-prefixed names for compiler built-ins, a precedent for visibly marking compiler-provided operations.
- [Odin language overview: conditional compilation](https://odin-lang.org/docs/overview/#conditional-compilation) — Odin uses `when` for compile-time conditions and `#`-prefixed compiler facilities such as `#config`; the marker and conditional spelling can be separate design choices.
- [Question 015](015_Runtime_control_of_comptime_state.md) — Records why a compile-time conditional is needed for `comp mut` updates.
- [Question 019](019_Builtin_sigil.md) — Explores whether compiler-provided constructs use `@`, `#`, or another convention.

# Answers

## A. Prefix the `if` with `comp`

```flang
comp mut x = 0;
comp if feature_enabled {
    x++;
}
somearray[x] = 5;
```

The prefix explicitly requires the condition and branch choice to be known during compilation. It reuses the `comp` marker from compile-time bindings.

## B. Put `comp` inside the condition position

```flang
comp mut x = 0;
if comp feature_enabled {
    x++;
}
somearray[x] = 5;
```

This emphasizes that the condition must be known during compilation while retaining `if` as the first word.

## C. Infer the stage from a known condition

```flang
comp mut x = 0;
if feature_enabled {
    x++;
}
somearray[x] = 5;
```

If `feature_enabled` is known during compilation, the compiler chooses the branch then. A runtime value would make the `x++` invalid. This is concise, but the meaning of an `if` depends on how its condition is classified.

## D. Use `@if`

```flang
comp mut x = 0;
@if feature_enabled {
    x++;
}
somearray[x] = 5;
```

The spelling visibly identifies a compiler-provided conditional and is compact. It may fit a broader `@` convention for built-ins, but that convention is still undecided.

## E. Use `#if`

```flang
comp mut x = 0;
#if feature_enabled {
    x++;
}
somearray[x] = 5;
```

This could fit a broader `#` convention for built-ins. The language would need to distinguish it from textual preprocessing: the inactive branch still must parse as flang syntax under [Question 018, answer A](018_Comptime_if_branch_checking.md).

# Selected answer

**Pending after reconsideration.** A (`comp if`) was previously selected, but the user has proposed D (`@if`) as another possible spelling and is still considering whether `@` or `#` should mark built-ins. The agreed semantics remain: the condition must be known during compilation, and [Question 018, answer A](018_Comptime_if_branch_checking.md) governs checking of the unchosen branch.
