# Question

May `if` omit `else`? If it does, does the whole `if` produce a unit value, or is it permitted only when used for effects?

Flang's `if` can produce a value and its branches follow the ordinary block rule ([Questions 006](006_If_values.md) and [007](007_If_branch_values.md)). The precise unit type and spelling are still open.

# Research

- [Rust Reference: `if` expressions](https://doc.rust-lang.org/reference/expressions/if-expr.html) — An `if` without `else` produces `()` when its condition is false; the expression must have a consistent type across paths.
- [OCaml manual: conditionals](https://ocaml.org/manual/5.3/expr.html) — An omitted `else` defaults to `else ()`, so a one-armed conditional has unit result.
- [Kotlin language specification: conditional expressions](https://kotlinlang.org/spec/expressions.html#conditional-expressions) — An `if` missing an `else` has `Unit` type and is permitted only as a statement.

# Answers

## A. Omitted `else` means unit on the false path

```flang
if ready {
    log("ready");
}
```

This can be used wherever a unit value is accepted. Its true branch must also have a compatible unit result. `if ready { 42 }` would be a type error unless the result is explicitly discarded.

## B. `else` is always required

```flang
if ready {
    log("ready");
} else {}
```

Every conditional shows both paths. This is uniform for value and effect uses, but adds empty `else` branches for common one-sided actions.

## C. Omit `else` only when the result is discarded

```flang
if ready {
    log("ready");
}
```

The form is convenient for effects, but cannot initialize a variable or appear where a value is required, even if that value would be unit. This treats use context as part of the validity rule.

# Selected answer

**A, with an explicit type condition.** An `if` may omit `else` only when its true branch produces unit. The missing false branch also produces unit, so the whole `if` has unit type. If either branch would produce a non-unit value, an `else` is required, even when the surrounding context would discard the result.
