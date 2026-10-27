# Question

Should flang use a consistent sigil for compiler-provided operations and constructs, and if so, should it be `@` or `#`?

This question affects more than the spelling of a compile-time conditional in [Question 017](017_Comptime_if_syntax.md). Possible examples below include type introspection and compile-time diagnostics, whose APIs are not yet designed.

# Research

- [Zig language reference: built-in functions](https://ziglang.org/documentation/master/#Builtin-Functions) — Zig uses `@` for compiler built-ins, such as `@typeInfo` and `@compileError`. Its compile-time conditional is still written with ordinary `if` plus compile-time context, so a built-in marker need not prefix every compile-time construct.
- [Odin language overview: conditional compilation](https://odin-lang.org/docs/overview/#conditional-compilation) — Odin uses `#` in compiler-provided facilities such as `#config`, but writes compile-time conditionals with `when`. This likewise separates the marker convention from the conditional spelling.
- [Question 017](017_Comptime_if_syntax.md) — Compares `comp if`, `@if`, and `#if` spellings.
- [Question 016](016_Compiler_introspection_and_messages.md) — Compiler introspection may need a visible namespace or marker.

# Answers

## A. Reserve `@` for compiler-provided operations

```flang
@if target_has_simd { /* ... */ }
comp fields = @type_info(Player).fields;
```

Built-ins are easy to recognize and cannot be mistaken for ordinary user functions. This uses punctuation in control flow and requires a rule for whether users may define `@` names.

## B. Reserve `#` for compiler-provided operations

```flang
#if target_has_simd { /* ... */ }
comp fields = #type_info(Player).fields;
```

The marker also makes built-ins recognizable. Readers may associate `#if` with textual preprocessing, so the language should make clear that both branches still parse and only the selected branch is type-checked.

## C. No universal sigil

```flang
comp if target_has_simd { /* ... */ }
comp fields = compiler.type_info(Player).fields;
```

Keywords and an explicit `compiler` namespace can identify operations by role. This avoids reserving punctuation but does not give all compiler-provided constructs one visual marker.

# Selected answer

Pending. The user is considering `@` and `#`; neither has been chosen.
