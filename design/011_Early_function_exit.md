# Question

How should a function exit before reaching its body's final expression?

Functions already return their body value on normal completion ([Question 010](010_Function_body_results.md)). Named blocks and loops already use `break` for early exits ([Questions 002](002_Blocks_as_values.md) and [003](003_Loop_values.md)). The examples below explore whether a function needs another exit mechanism.

# Research

- [Rust Reference: `return` expressions](https://doc.rust-lang.org/reference/expressions/return-expr.html) — `return value` exits the current function immediately. The [function body rule](https://doc.rust-lang.org/reference/items/functions.html#function-body) still permits a final expression on the normal path.
- [Zig language reference: functions](https://ziglang.org/documentation/master/#Functions) — Zig examples use `return value` to exit functions, including from inside conditional branches.
- [OCaml language manual: function expressions](https://ocaml.org/manual/5.3/expr.html) — A function body is an expression whose evaluated value becomes the call's result. A source-level `return` construct is not part of the function-expression grammar, illustrating a structured-expression alternative.

# Answers

## A. Add `return` for early function exit

```flang
fn classify(number: Int) -> Int {
    if number < 0 {
        return -1;
    }
    return 1;
}
```

`return` specifically exits the current function, including from nested blocks or loops. The user's added rule requires every path that completes this function to use `return` once any `return` targeting it appears. A final expression alone is then not a valid result path. Paths that never complete do not need a `return`.

## B. Let `break` target the function by name

```flang
fn classify(number: Int) -> Int {
    if number < 0 {
        break :classify -1;
    }
    break :classify 1;
}
```

This reuses the named-exit mechanism. It also subjects the function body to the rule that, once a targeted break exists, all completing paths must break. Naming collisions and anonymous functions would need rules.

## C. No direct early function exit

```flang
fn classify(number: Int) -> Int {
    if number < 0 { -1 } else { 1 }
}
```

Control flow must be expressed through the final expression or an inner named block. This avoids a function-specific exit construct, but can make deeply nested searches or error paths more cumbersome.

# Selected answer

**A, with a qualification.** `return` exits the current function. If at least one `return` targeting that function appears, every path that completes it must use an explicit `return`, including the path at the end of the body. Otherwise, the body's final expression supplies the result as chosen in [Question 010](010_Function_body_results.md). A `return` inside a nested function applies to that nested function, not the outer one.
