# Question

When a function's body reaches its closing brace, does the body's value become the function's return value, or must the function use an explicit `return`?

Ordinary blocks already yield their final expression ([Question 007](007_If_branch_values.md)). The function declaration syntax below is illustrative.

# Research

- [Rust Reference: function bodies](https://doc.rust-lang.org/reference/items/functions.html#function-body) — The function body's tail expression is returned to the caller. An explicit `return` exits earlier.
- [Zig language reference: functions](https://ziglang.org/documentation/master/#Functions) — The examples use explicit `return` statements to provide function results.
- [OCaml language manual: function expressions](https://ocaml.org/manual/5.3/expr.html) — A function's body is an expression; applying the function evaluates that expression to produce its result.

# Answers

## A. The function body returns its value

```flang
fn double(number: Int) -> Int {
    number * 2
}
```

This applies the ordinary block rule to functions. An explicit `return value` could still exit early; whether flang has such a keyword is a separate question. A non-unit function whose body completes with unit would be a type error.

## B. Every non-unit function result uses explicit `return`

```flang
fn double(number: Int) -> Int {
    return number * 2;
}
```

The return point is visible, but a function body differs from other blocks: its final expression is not enough to return a value. The compiler must check that every completing path in a non-unit function returns.

# Selected answer

**A, qualified by [Question 011](011_Early_function_exit.md).** When a function has no `return` targeting it, its body's final expression is returned to the caller, following the ordinary block rule. A non-unit function whose body completes with unit is a type error. Once the function uses `return`, all paths completing the function must use explicit `return`.
