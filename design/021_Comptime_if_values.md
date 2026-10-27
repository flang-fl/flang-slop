# Question

Does a compile-time conditional produce the selected branch's value, just as an ordinary `if` does?

The spelling `comp if` below is provisional ([Question 017](017_Comptime_if_syntax.md)). [Question 007, answer B](007_If_branch_values.md) says ordinary blocks yield their final expression, and [Question 010, answer A](010_Function_body_results.md) says a function with no `return` yields its body's value.

```flang
fn newline() -> &'static str {
    comp if windows {
        "\r\n"
    } else {
        "\n"
    }
}
```

The `&'static str` type is an illustrative spelling from the user, not a decision about flang's string or lifetime types.

# Research

- [Zig language reference: compile-time expressions](https://ziglang.org/documentation/master/#Compile-Time-Expressions) — Zig can force evaluation of an expression at compile time; an `if` whose condition is known during compilation selects one branch while skipping analysis of the other.
- [Rust Reference: `if` expressions](https://doc.rust-lang.org/reference/expressions/if-expr.html) — An `if` is an expression whose value comes from the selected block. This is the model already chosen for ordinary flang `if` expressions.
- [D language specification: `static if`](https://dlang.org/spec/version.html#staticif) — D's `static if` is a conditional compilation construct that can select declarations or statements; it is a precedent for treating compile-time selection as a distinct form rather than requiring it to mirror expression `if`.
- [Question 020](020_Comptime_if_file_scope.md) — The user's `newline` example is a reason to distinguish body-level selection from file-level declaration selection.

# Answers

## A. Same result rule as ordinary `if`

The selected branch yields its final expression. In the example, the function body yields the selected string, so no explicit `return` is needed. If either branch uses `return` targeting the function, [Question 011](011_Early_function_exit.md) requires explicit returns on all completing function paths.

## B. Compile-time conditional is only a selection statement

The example cannot produce a result directly. A separate compile-time value binding or explicit returns in each branch would be needed. This keeps conditional compilation separate from expression evaluation, but makes simple cases more verbose.

# Selected answer

Pending. The user's example suggests A, but they asked how the existing return rules apply rather than explicitly choosing this additional rule.
