# Question

How can `if number < 0 { -1 } else { 1 }` produce a value if its braces must obey exactly the same value rule as other blocks?

The desired spelling and `if` as an expression are recorded in [Question 006](006_If_values.md). The original explicit-break and fallthrough choices are recorded in [Questions 002](002_Blocks_as_values.md) and [005](005_Block_fallthrough.md). The user prefers one block rule everywhere, so a branch-only final-expression rule would not meet that goal. Under the original fallthrough rule, `{ -1 }` could not produce `-1`; selecting B below revises that rule.

# Research

- [Kotlin language specification: code blocks and control structure bodies](https://kotlinlang.org/spec/statements.html#code-blocks) — A control structure body can take the value of its last expression. The [Kotlin `if` documentation](https://kotlinlang.org/docs/control-flow.html#if-expression) shows a multi-statement branch yielding its last expression.
- [Rust Reference: `if` expressions and block expressions](https://doc.rust-lang.org/reference/expressions/if-expr.html) — Each `if` branch is a block expression; its final expression determines its value. Rust's condition grammar excludes struct expressions in this position, an example of a parsing constraint when parentheses are optional.
- [Zig language reference: `if` and blocks](https://ziglang.org/documentation/master/#if) — `if` selects between expressions, while a labeled multi-statement block can return a value through `break :label value`. This keeps explicit block exits, though a simple `{ -1 }` branch would not mean `-1` under a strict Zig-like block rule.

# Answers

## A. `if` branch bodies have their own final-expression rule — conflicts with the stated preference

```flang
let sign = if number < 0 {
    log("negative");
    -1
} else {
    1
};
```

The last expression in a braced `if` branch is its value. An ordinary named block with no break still yields unit. This preserves the requested syntax and the earlier block rule, at the cost of braces behaving differently in an `if` branch.

## B. All blocks may yield their final expression

```flang
let sign = if number < 0 { -1 } else { 1 };
let forty_two = { 40 + 2 };
```

Braces have one general result rule, including in `if`. A block with no targeted breaks yields its final expression, or unit if none. A block with targeted breaks requires all completing paths to break, preserving the explicit multi-exit rule from Question 005. This prevents mixing implicit fallthrough and explicit exits in the same block.

## C. A braced single expression is a value; longer branches need explicit breaks — conflicts with the stated preference

```flang
let sign = if number < 0 { -1 } else { 1 };

let logged_sign = if number < 0 negative: {
    log("negative");
    break :negative -1;
} else { 1 };
```

The desired short form works as expression grouping. A branch containing declarations or several statements is an ordinary block and needs a named explicit exit to produce a value. This preserves the block rule but creates a sharp syntax distinction between short and long branches.

## D. Keep explicit block results and change the `if` example

```flang
let sign = result: {
    if number < 0 {
        break :result -1;
    } else {
        break :result 1;
    }
};
```

All braces remain ordinary blocks under the decisions already made. `if` still determines which branch runs, and the enclosing named block supplies the value. This gives up the concise `if number < 0 { -1 } else { 1 }` form.

# Selected answer

**B.** Every block follows the same result rule, whether used alone or as an `if` branch. With no targeted break, its final expression is its value; with no final expression, it produces unit. If the block has a targeted break, every path that completes it must break to it. `break` is the explicit mechanism for multiple exits and for exiting an enclosing named block. This revises [Question 005](005_Block_fallthrough.md) and extends [Question 002](002_Blocks_as_values.md).
