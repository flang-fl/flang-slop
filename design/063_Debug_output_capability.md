# Question

Should flang provide `debug_print` that can write without an ordinary output capability and disappears from builds where debug output is disabled? If so, what happens to its argument expressions and type checking?

This is a possible narrow exception to explicit external capabilities in [Question 062](062_External_capabilities.md), which remains undecided. It would be runtime output in enabled builds, not a compile-time compiler message from [Question 016](016_Compiler_introspection_and_messages.md). The language already contemplates debug, release-safe, and fast modes in [Question 041](041_Runtime_integer_overflow_modes.md), so a dedicated `debug_output` build setting may be clearer than using a broad word such as "release". All syntax below is provisional.

# Research

- [Rust `dbg!` documentation](https://doc.rust-lang.org/std/macro.dbg.html): prints to standard error in both debug and release builds; this is not the requested build-only behavior.
- [Rust `debug_assert!` documentation](https://doc.rust-lang.org/std/macro.debug_assert.html): disabled by default in optimized builds; its expression is still type-checked but not executed when disabled.
- [Rust Reference, *Conditional compilation*](https://doc.rust-lang.org/reference/conditional-compilation.html): a separate approach removes source forms under selected build configurations.
- [Miller, Yee, and Shapiro, *Capability Myths Demolished* (2003)](https://papers.agoric.com/assets/pdf/papers/capability-myths-demolished.pdf): an ambient debug channel is still authority, even if restricted to some builds.

# Answers

## A. No ambient debug output

Debug logging uses an explicit logger capability. A compile-time build conditional can omit calls in builds with debug output disabled. This preserves a uniform authority rule, but the logger must be passed or captured where needed.

```flang
if comp build.debug_output { logger.print("value = {}", value) }
```

## B. Built-in debug channel, disabled in selected builds

`debug_print` may use a compiler or host-provided diagnostic sink without an output argument. In enabled builds it formats and writes at runtime; in disabled builds it emits no runtime code and does not evaluate its argument expressions. A strong version still parses and type-checks the call in all builds, so a typo or missing formatting trait is not hidden by building with debug output disabled. This is a deliberate, narrow exception to a strict capability model.

```flang
debug_print("value = {}", value)
```

The build setting, failure policy for the diagnostic sink, evaluation and ownership of arguments, and target-specific destination remain open. The operation could borrow rather than consume values to reduce build-dependent ownership behavior.

## C. Explicit capability at the entry point, implicitly available only in debug builds

An entry point or test host receives a `DebugSink` capability, and the compiler threads it through calls that use `debug_print`. This retains a capability origin but makes passing hidden in those builds. It needs rules for function types, libraries, function values, and calls compiled under different build settings; it approaches typed implicit capabilities from [Question 062](062_External_capabilities.md).

# Selected answer

B, with qualification: a debug-output operation, provisionally called `debug_print`, is an exception to the ordinary rule that output capability must be passed to a function. It may emit runtime diagnostic output in builds where debug output is enabled and is absent from builds where it is disabled. The name, destination (including whether it uses standard error, standard output, or a target-specific sink), build setting, failure behavior, and precise argument evaluation and type-checking rules remain open.
