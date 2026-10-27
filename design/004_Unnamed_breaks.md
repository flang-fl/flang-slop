# Question

When `break` carries a value, may it omit the name of its target? If so, which construct does it exit?

This is separate from the decided ability of [named blocks](002_Blocks_as_values.md) and [loops](003_Loop_values.md) to produce values.

# Research

- [Zig language reference: blocks and loops](https://ziglang.org/documentation/master/#Blocks) — A block uses a label and `break :label value` to return a value. A `while` or `for` loop can use an unnamed `break value`; labels target an outer loop. See the [`while` examples](https://ziglang.org/documentation/master/#while).
- [Rust Reference: loops and breakable expressions](https://doc.rust-lang.org/reference/expressions/loop-expr.html) — An unnamed `break` targets the innermost loop. `break value` works in an unconditional `loop`; a labeled `break` can target an outer loop or labeled block.

# Answers

## A. Omit the target only for the innermost loop

```flang
let answer = loop {
    if ready() {
        break 42;
    }
};
```

An unnamed `break` or `continue` always targets the innermost loop. Breaking from a named block, or an outer loop, requires its name. This is concise for the common case and keeps the target predictable.

## B. Require a name whenever a break carries a value

```flang
let answer = search: loop {
    if ready() {
        break :search 42;
    }
};
```

Value exits are always visually explicit. Even the simplest loop needs a name if it produces a value.

## C. An unnamed value break targets the nearest value-producing construct

```flang
let answer = result: {
    if ready() {
        break 42;
    }
};
```

This saves names in blocks too, but adding a nested loop or another value-producing construct could silently change the target of an existing `break 42`.

# Selected answer

**A.** An unnamed `break` or `continue` targets the innermost loop. `break value` is allowed there. A named block or outer loop must be targeted explicitly. The exact label punctuation remains undecided.
