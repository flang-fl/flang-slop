# Question

Can an `if` itself produce a value, or is it only a control-flow statement?

Named blocks already provide a way to choose a value through `break` ([Question 002](002_Blocks_as_values.md)). This question asks whether a simple conditional should need such a block. The flang syntax below is illustrative.

# Research

- [Rust Reference: `if` expressions](https://doc.rust-lang.org/reference/expressions/if-expr.html) — `if` evaluates to the chosen branch's value. Without `else`, its skipped path has unit value, so all possible outcomes must have compatible types.
- [Zig language reference: `if`](https://ziglang.org/documentation/master/#if) — `if` can select between two expressions, including values used in an assignment. Zig does not need a separate ternary operator.
- [OCaml manual: conditional expressions](https://ocaml.org/manual/5.3/expr.html) — `if ... then ... else ...` is an expression; when `else` is omitted it defaults to unit.
- [Go specification: `if` statements](https://go.dev/ref/spec#If_statements) — `if` controls statement blocks and does not itself produce a value.

# Answers

## A. `if` can produce a value

```flang
let sign = if number < 0 { -1 } else { 1 };
```

This is the preferred surface syntax, with optional parentheses around the condition. An `if` used for a non-unit value needs an `else`, and both branches need compatible types. Its braces follow the ordinary block result rule selected in [Question 007](007_If_branch_values.md). An `if` used only for effects could omit `else`.

## B. `if` only controls statements

```flang
let sign = result: {
    if (number < 0) {
        break :result -1;
    }
    break :result 1;
};
```

There is one value-exit mechanism: a named block with `break`. This avoids a second value-producing control form, but simple conditionals require more ceremony.

# Selected answer

**A.** `if` may produce a value. Parentheses around its condition are optional. The preferred form is `if number < 0 { -1 } else { 1 }`; its branches use the same block value rule as other blocks, selected in [Question 007](007_If_branch_values.md).
