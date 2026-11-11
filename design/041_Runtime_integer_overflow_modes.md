# Question

What should ordinary runtime integer overflow do in a build mode without the mandatory checks of debug and release-safe builds?

The user has decided that overflow in runtime-dependent integer arithmetic must fail in debug and release-safe builds. In contrast, arithmetic whose numeric inputs are all known during compilation uses exact intermediate values and checks the final result, as chosen in [Question 039](039_Numeric_expression_context.md) and [Question 040](040_Typed_constants_in_exact_arithmetic.md). This question concerns a runtime value whose sum exceeds its concrete integer type. Build-mode names and the exact form of failure remain provisional.

```flang
fn increment(value: i32) -> i32 {
    value + 1
}

// A runtime call with value == 2147483647 overflows i32.
```

# Research

- [Zig language reference: optimization modes and integer overflow](https://ziglang.org/documentation/master/) — Zig enables runtime safety checks in debug and safe modes and disables them by default in fast and small modes. Ordinary integer overflow is illegal behavior; explicit wrapping and saturating operators exist. Its compile-time evaluator diagnoses known typed overflow.
- [Rust Reference: operator overflow](https://doc.rust-lang.org/reference/expressions/operator-expr.html#overflow) — Rust checks and panics on integer overflow in debug builds, with compiler options controlling overflow checks. This shows that checking policy can vary by build configuration.
- [Question 040](040_Typed_constants_in_exact_arithmetic.md) — Establishes the difference between exact compile-time arithmetic and runtime-dependent typed arithmetic, and records mandatory failure in debug and release-safe builds.

# Answers

## A. Wrap in fast builds

Unchecked ordinary arithmetic uses modular two's-complement wrapping in a fast build. This is predictable but means the same source expression can fail in a safe build and return a wrapped value in a fast build.

## B. Treat overflow as invalid behavior in fast builds

The compiler may omit overflow checks and optimize under the assumption that overflow never happens. Programs that overflow have no guaranteed result in fast builds; explicit wrapping operations would be needed for intentional modular arithmetic. This permits more optimization but makes mistakes harder to diagnose.

## C. Keep overflow failure in every build mode

Ordinary runtime arithmetic always checks overflow and fails, including fast builds. This preserves one behavior across modes, at the cost of checks where the compiler cannot prove an operation safe. A separate explicit wrapping operation could still exist.

# Selected answer

**B.** Debug and release-safe builds check runtime-dependent integer arithmetic and fail on overflow. In a fast build without mandatory checks, ordinary integer overflow has no guaranteed behavior; the compiler may optimize assuming it does not happen. Intentional modular arithmetic requires an explicit wrapping operation rather than relying on overflow of ordinary operators. The spelling of that operation and the exact failure mechanism in checked builds remain open; the latter is [Question 042](042_Checked_overflow_failure.md).
