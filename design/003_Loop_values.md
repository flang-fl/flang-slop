# Question

If loops can have names targeted by `break` and `continue`, should a `break` also be able to give a loop a value? What value does a loop have when it finishes normally?

The names and punctuation in these flang examples are provisional. `continue :outer` would start the next iteration of `outer`; `break :outer` would exit it.

# Research

- [Zig language reference: `while`, `for`, and labeled loops](https://ziglang.org/documentation/master/#while) — `while` and `for` can be expressions. `break` can provide a result; `else` supplies one on normal completion. Labels let nested `break` and `continue` target an outer loop. See also [Zig's `for` examples](https://ziglang.org/documentation/master/#for).
- [Rust Reference: loops and breakable expressions](https://doc.rust-lang.org/reference/expressions/loop-expr.html) — Labels can target an outer loop with `break` or `continue`. A bare `loop` can receive a value from `break`, while `while` and `for` do not produce non-unit results. Rust also permits `break` with a value from a labeled block.
- [Question 002: blocks as values](002_Blocks_as_values.md) — Establishes the chosen direction for named blocks, but does not decide loop results.

# Answers

## A. Every loop may yield a value

```flang
let index = 0;
let found_at = search: while index < items.length {
    if items[index].matches() {
        break :search index;
    }
    index += 1;
} else {
    -1
};
```

Like Zig, a normal end needs a result too when the loop is used as a value. An `else` branch is one possible way to supply it. The surrounding syntax is illustrative, not settled.

## B. Only an unconditional loop may yield a value

```flang
let answer = search: loop {
    let candidate = next_candidate();
    if candidate.ready() {
        break :search candidate;
    }
};
```

Like Rust, a loop that can end on its own would be used for effects or wrapped in a named value-producing block. An unconditional loop has no normal end to account for.

## C. Loop labels only control the iteration

```flang
outer: while more_rows() {
    while more_columns() {
        if skip_row() {
            continue :outer;
        }
        if done() {
            break :outer;
        }
    }
}
```

Named `break` and `continue` work, but loops never produce a useful value. A separate variable or named block can collect a result.

# Discussion

## Tradeoffs of A and B

**A, like Zig:** One loop can express both "found a result early" and "finished without finding one." The types of the `break` value and normal-completion value must agree when the loop is used as a value. This adds a meaningful `else` to loops and more control-flow paths for readers and type checking to track. A loop with no `else` could produce unit on normal completion, so a value-carrying `break` would need another matching result path.

**B, like Rust:** Only an unconditional `loop` can return a non-unit result, so it has no normal-completion case to explain. `while` and `for` remain simpler. A search over a finite collection needs a wrapper block, a result variable, or a different expression (such as an iterator operation). The distinction between loop kinds can feel arbitrary when the same search is easy to write as a `for` loop.

# Selected answer

**A.** `while` and `for` may produce values via `break`, and `else` supplies the value on normal completion. The exact syntax and the type rule for loops used only for effects remain open. Whether `break value` can omit a target is [Question 004](004_Unnamed_breaks.md).
