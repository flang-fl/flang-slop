# Question

What does a debug or release-safe build do when a runtime integer overflow check detects overflow?

[Question 041, answer B](041_Runtime_integer_overflow_modes.md) says that ordinary integer overflow must fail in those builds, while fast builds without checks give it no guaranteed behavior. "Fail" still needs a precise execution rule. The mechanism also needs to make sense for a future DiamondFire target, where ending a plot execution may differ from aborting a native process. The syntax and target mapping are provisional.

```flang
fn increment(value: i32) -> i32 {
    value + 1
}

// What happens when increment receives the maximum i32 value in a checked build?
```

# Research

- [Zig language reference: illegal behavior and integer overflow](https://ziglang.org/documentation/master/) — A detected runtime safety violation triggers a panic. Zig's root source can provide a panic handler, while its default handler terminates execution. This shows a trap-like path with configurable reporting.
- [Rust Reference: panic](https://doc.rust-lang.org/reference/panic.html) — Rust can use an unwinding panic handler, which may be caught, or an aborting handler. This shows that "panic" alone does not settle recoverability or cleanup.
- [Rust Reference: operator overflow](https://doc.rust-lang.org/reference/expressions/operator-expr.html#overflow) — Rust's checked integer overflow invokes panic, establishing a precedent for routing arithmetic checks through a general failure mechanism.
- [Question 041](041_Runtime_integer_overflow_modes.md) — Fixes the modes that must detect overflow; this question chooses what detection does.

# Answers

## A. Stop the current execution without language-level recovery

Overflow triggers a trap or abort appropriate to the target. Native code might terminate the process or thread; DiamondFire code might stop the current plot execution. The language cannot catch this failure as a value. This is simple, but offers little control over cleanup or reporting.

## B. Raise a language-level panic that can be caught

Overflow enters a general panic mechanism. A caller or boundary can catch it if the target supports the necessary unwinding or equivalent propagation; otherwise it reaches a default handler. This allows recovery and common reporting, but requires defined cleanup and cross-target panic behavior.

## C. Propagate a typed error through the expression and function result

Checked arithmetic produces a recoverable error that callers must handle or propagate. This makes overflow part of ordinary control flow, but would change the type or effect of expressions containing basic arithmetic and require syntax for propagation.

# Selected answer

**A, qualified.** When a debug or release-safe runtime check detects overflow in an ordinary integer operation, it stops the current execution without language-level recovery. The target determines how that stop is implemented; the exact diagnostics and cleanup behavior remain open. Code that wants to handle overflow uses an explicit fallible arithmetic operation such as a provisional `try_add`, which returns `Result<T, E>` rather than triggering the ordinary operator's failure path. The error type is open in [Question 043](043_Fallible_arithmetic_result.md).
