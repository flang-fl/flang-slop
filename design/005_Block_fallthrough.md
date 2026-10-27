# Question

What happens when execution reaches the closing brace of a named block without a `break` targeting that block?

Named blocks can already produce values through explicit `break` ([Question 002](002_Blocks_as_values.md)). This question asks whether reaching the end produces a value, is an error, or uses the final expression. Flang syntax below remains provisional.

# Research

- [Zig language reference: blocks and empty blocks](https://ziglang.org/documentation/master/#Blocks) — A labeled block can return a value with `break :label value`. The reference also documents an empty block as producing Zig's `void` value. This informed the original fallthrough question; the current flang rule is recorded below.
- [Rust Reference: labeled block expressions](https://doc.rust-lang.org/reference/expressions/loop-expr.html#labeled-block-expressions) — A labeled block may exit through a labeled `break` or reach its final expression. Its type accounts for the break values and that final expression; without a final expression, the fallthrough path has unit type.
- [Rust Reference: block expressions](https://doc.rust-lang.org/reference/expressions/block-expr.html) — A block's final expression is its value; a block without one produces unit. This is the implicit-result alternative.

# Answers

## A. Reaching the end produces a unit value

```flang
work: {
    log("done");
} // completes with a unit-like value
```

If the block has no `break` targeting it, reaching the end produces unit. The user's chosen qualification is stricter than merely checking compatible types: once any path breaks to this block, every path that completes the block must break to it. A path that reaches `}` is an error even if its hypothetical unit value would match the break value. The exact unit type remains a separate design question.

## B. Every path through a named block must break explicitly

```flang
result: {
    log("done");
    break :result;
}
```

Even a named block used only to group work must state its exit. This is uniform but adds ceremony to effect-only blocks.

## C. The final expression supplies the fallthrough value

```flang
let answer = result: {
    if ready() {
        break :result 42;
    }
    0
};
```

This resembles Rust's labeled blocks. It makes a default result concise, but combines an explicit `break` rule with an implicit final-expression rule.

# Selected answer

**A, with a qualification, was selected initially and then revised by [Question 007, answer B](007_If_branch_values.md).** The current rule is: with no break targeting it, a block produces its final expression, or unit if there is none. If any path breaks to the block, all paths that complete the block must use a break targeting it; reaching `}` is then an error. A path that exits the function or never completes does not need a break to this block.
