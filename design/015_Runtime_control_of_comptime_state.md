# Question

What happens when runtime control flow would change a `comp mut` value whose next use must be known during compilation?

```flang
fn write(condition: bool) {
    comp mut x = 0;
    if condition { x++; }
    somearray[x] = 5;
}
```

`condition` is a runtime value and `somearray` is a runtime array. The compiler cannot know a single value of `x` at the final write without a further rule. This follows [Question 014, answer B](014_Comptime_mutable_scope.md).

# Research

- [Zig language reference: compile-time variables](https://ziglang.org/documentation/master/#Compile-Time-Variables) — Every load and store of a `comptime` variable must happen during compilation; a violation is a compile error. Zig also permits functions containing both compile-time and runtime operations.
- [Zig language reference: compile-time expressions](https://ziglang.org/documentation/master/#Compile-Time-Expressions) — A forced compile-time control-flow expression errors if it depends on a runtime value. This is a precedent for requiring the stage boundary to be explicit.
- [Question 014](014_Comptime_mutable_scope.md) — Establishes function-local `comp mut` mixed with emitted runtime writes.

# Answers

## A. Reject runtime control over compile-time mutation

`if condition { x++; }` is an error because `condition` is unknown during compilation. Write the runtime choice in ordinary runtime code instead:

```flang
fn write(condition: bool) {
    if condition {
        somearray[1] = 5;
    } else {
        somearray[0] = 5;
    }
}
```

The stage of each `comp mut` update stays clear. This may require rewriting code that could theoretically be specialized automatically.

If the condition itself is evaluated during compilation, the update is allowed. The syntax for requesting a compile-time `if` is [Question 017](017_Comptime_if_syntax.md).

## B. Split and specialize both runtime paths

The compiler tracks `x = 1` on the true path and `x = 0` on the false path, then emits a write in each path. This is convenient, but joins, loops, and side effects would need rules to control code duplication and compilation cost.

## C. Promote `x` to runtime state

`x` becomes a runtime local when a runtime condition controls its mutation. The source remains concise, but `comp mut` would no longer guarantee that its loads and stores happen during compilation.

# Selected answer

**A.** A runtime-dependent branch cannot update `comp mut` state. A compile-time `if` may update it because the compiler knows which branch executes. The syntax for that form remains undecided.
