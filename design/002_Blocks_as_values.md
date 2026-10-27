# Question

Should a block of code be usable as an expression that produces a value? If so, how does the block choose that value?

This is about a block itself. Whether `if`, loops, or other constructs produce values can be decided separately. All flang syntax below is illustrative, not yet a decision.

# Research

- [Rust Reference: block expressions](https://doc.rust-lang.org/stable/reference/expressions/block-expr.html) — A block runs its statements, then evaluates an optional final expression. That expression supplies the block's value and type; a block without one produces `()`.
- [Scala documentation: blocks](https://docs.scala-lang.org/tour/basics.html#blocks) — A block's last expression is its result. The same rule applies to a method body.
- [OCaml language manual: expressions](https://ocaml.org/manual/5.3/expr.html) — OCaml uses expression sequencing: `e1; e2` evaluates both and gives the value of `e2`. Its `begin ... end` groups an expression. This shows that sequencing can produce a value without curly braces.
- [Go language specification: blocks and return statements](https://go.dev/ref/spec#Blocks) — A block is a sequence of declarations and statements. Functions return results with a `return` statement; the block itself has no result value. See also [return statements](https://go.dev/ref/spec#Return_statements).
- [Zig language reference: blocks](https://ziglang.org/documentation/master/#Blocks) — Blocks are expressions; a labeled block can produce a chosen value through `break :label value`.

### The same calculation in existing languages

All five examples compute `6`. Their real syntax makes the distinction clearer than provisional flang syntax:

```rust
let six = {
    let two = 2;
    two * 3 // no semicolon: this is the block's value
};
```

```scala
val six = {
  val two = 2
  two * 3 // last expression is the block's value
}
```

```ocaml
let six =
  let two = 2 in
  two * 3
```

OCaml's local `let ... in` is an expression; it does not need a `{ ... }` block. For sequencing, `e1; e2` yields `e2`.

```go
func six() int {
    two := 2
    return two * 3 // the function returns explicitly
}
```

```zig
const six = blk: {
    const two = 2;
    break :blk two * 3; // exits the labeled block with this value
};
```

Rust and Scala illustrate answer A, Go illustrates B, and Zig illustrates an explicit value exit similar to C. OCaml demonstrates another route: make local binding and sequencing expressions without making braces central.

# Answers

## A. The final expression is the value

```flang
let six = {
    let two = 2;
    two * 3
};
```

The final expression determines the block's type. We would still need to decide what an empty block or a block ending with a statement produces, and how a semicolon changes the result.

## B. Blocks only sequence statements

```flang
let two = 2;
let six = two * 3;
```

A block groups work and establishes scope, but cannot appear where a value is expected. A function would use an explicit `return` to produce a result. This gives statements and expressions distinct roles.

## C. Named blocks produce values through an explicit break

```flang
let six = calculation: {
    let two = 2;
    break :calculation two * 3;
};
```

The `break` exits the named block immediately and supplies its result. It can appear before the end of the block, allowing different paths to provide a value. The exact label syntax remains open. The current rule for block completion without a targeted `break` is in [Question 007](007_If_branch_values.md).

# Selected answer

**C, subsequently extended by [Question 007, answer B](007_If_branch_values.md).** Named blocks support an explicit `break` to produce a value or exit early. Blocks without a break targeting them can also produce the value of their final expression. The label spelling remains open.

The related idea of named `break` and `continue` for loops leads to [Question 003](003_Loop_values.md).
