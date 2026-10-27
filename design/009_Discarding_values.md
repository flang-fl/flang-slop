# Question

When an expression produces a value that the program does not need, may it simply stand alone as a statement, or must the discard be explicit?

This does not change [Question 008](008_If_without_else.md): an `if` without `else` must have unit type, even if the surrounding context would discard its value. The semicolons and `_` spelling below are provisional.

# Research

- [Rust Reference: expression statements](https://doc.rust-lang.org/reference/statements.html#expression-statements) — An expression statement evaluates an expression and ignores its result. A semicolon can turn an expression into a statement, including a call that returns a value.
- [Zig language reference: ignored expression values](https://ziglang.org/documentation/master/#void) — A non-`void` expression cannot be ignored by simply writing it as a statement; `_ = expression` explicitly discards its value. `void` expressions may stand alone.
- [Go specification: expression statements](https://go.dev/ref/spec#Expression_statements) — Calls and receive operations may appear as expression statements, while arbitrary expressions generally cannot. This illustrates a restricted middle ground.

# Answers

## A. A statement position may discard any expression value

```flang
calculate(); // result is ignored
```

This is concise and lets the semicolon indicate that the value is unused. Accidentally ignoring a meaningful result may be easy unless warnings or a "must use" mechanism are added later.

## B. Non-unit values require an explicit discard

```flang
_ = calculate(); // result is intentionally ignored
```

A standalone expression must have unit type. This catches accidental loss of a value, at the cost of extra notation when the caller genuinely does not care about it.

## C. Only selected expression forms may stand alone

```flang
calculate(); // a call is allowed; `1 + 2;` is not
```

This permits familiar call statements while rejecting obviously unused arithmetic. The rule depends on syntax rather than just type, so a call returning an important result can still be silently ignored.

# Selected answer

**B.** A standalone expression must have unit type. A non-unit value requires an explicit discard, provisionally written `_ = expression;`. This does not relax the rule for an `if` without `else` from [Question 008](008_If_without_else.md).
