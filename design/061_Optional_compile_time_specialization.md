# Question

Should a function be able to use an argument at compile time when it happens to be compile-time-known, while still accepting a runtime value? If so, how is that choice exposed and guaranteed?

This differs from a required `comp` parameter, which rejects runtime values. It also differs from ordinary optimization: a source-level compile-time branch may use compile-time-only operations and can follow [Question 018](018_Comptime_if_branch_checking.md)'s selected-branch checking. Function-local compile-time state already has mixed-stage rules in [Questions 014–015](014_Comptime_mutable_scope.md). Syntax below is provisional.

# Research

- [Jones, Gomard, and Sestoft, *Partial Evaluation and Automatic Program Generation* (1993)](https://www.cs.utexas.edu/~novak/jonesgomardsestoft.pdf): describes specialization from inputs known at an earlier stage and residual code for later inputs.
- [Zig language reference, *Compile-Time Parameters* and `@inComptime`](https://ziglang.org/documentation/master/#comptime): required `comptime` parameters differ from a query about the current evaluation context; the latter is not a general test of whether each runtime argument happened to be constant.
- [C++ draft, `if consteval`](https://eel.is/c%2B%2Bdraft/stmt.if): selects a branch according to constant-evaluation context, which is different from testing one argument's compile-time availability.
- [Rust `const_eval_select` documentation](https://doc.rust-lang.org/std/intrinsics/fn.const_eval_select.html): experimental context-sensitive selection notes the semantic concern when compile-time and runtime paths produce different behavior.

# Answers

## A. No source-level optional compile-time branch

Ordinary function arguments are runtime values unless declared `comp`. The optimizer may fold computations or specialize calls when it can, without the program observing that choice. To require compile-time computation, the programmer uses a required `comp` parameter or a separate compile-time function.

```flang
fn matches(pattern: str, text: str) -> bool {
    runtime_match(pattern, text)
}
// The compiler may optimize matches("abc", text).
```

## B. Stage-polymorphic parameter with a compile-time branch

An optional marker such as `comp?` says the function accepts either a compile-time-known or runtime value. A stage test such as `if comp pattern` is decided during call specialization, never at runtime. A compile-time-known call gets a generated version using the first branch; a runtime call uses the other. The stage choice must follow specified front-end rules rather than LLVM optimization accidents. This is conceptually an inferred compile-time boolean for the argument.

```flang
fn matches(pattern: comp? str, text: str) -> bool {
    if comp pattern {
        comp machine = compile_pattern(pattern)
        machine.matches(text)
    } else {
        runtime_match(pattern, text)
    }
}
```

This requires rules for first-class function values, separate compilation, code-size limits, and whether both paths must have equivalent observable behavior. A direct call with a compile-time-known argument would be guaranteed to specialize; an indirect call through an ordinary runtime function value might use the runtime variant unless a specialized function value was formed first.

## C. Separate explicit compile-time and runtime entry points

The library offers a required-`comp` specialized API and a runtime fallback. Callers choose which contract they want. This avoids a stage-polymorphic function type but duplicates part of the API and may need a wrapper for ergonomic calls.

```flang
fn matches_comp(comp pattern: str, text: str) -> bool { /* specialized */ }
fn matches_runtime(pattern: str, text: str) -> bool { /* runtime */ }
```

# Selected answer

B, with qualification: a function may mark an argument as optionally compile-time-known and select a compile-time branch when that argument is known during call specialization, while retaining a runtime branch for runtime values. A direct call with a compile-time-known argument is guaranteed to take the compile-time path according to language-defined stage rules, not backend optimization. The exact `comp?` and `if comp` spelling, rules for indirect calls and first-class function values, and whether the branches must have equivalent observable behavior remain open.
