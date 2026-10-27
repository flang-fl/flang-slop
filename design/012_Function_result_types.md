# Question

Must a named function declare its result type, or may the compiler infer it from the body?

This asks only about result types, not parameter annotations, generics, or whether functions are public. The visibility spelling in examples is provisional. Functions already return the body's final value when there is no explicit `return` ([Questions 010](010_Function_body_results.md) and [011](011_Early_function_exit.md)).

# Research

- [Rust Reference: functions](https://doc.rust-lang.org/reference/items/functions.html) — A named function has an explicit result type when it returns something other than unit; omitting the result type means unit.
- [Milner, “A Theory of Type Polymorphism in Programming” (1978)](https://www.pure.ed.ac.uk/ws/files/15143545/1_s2.0_0022000078900144_main.pdf) — Presents a type inference method for ML expressions. This is foundational evidence that static typing need not require an annotation on every result, though flang may have language features beyond the paper's core calculus.
- [OCaml language manual: function definitions](https://ocaml.org/manual/5.3/expr.html) — Functions can be defined without an explicit result type; a result type constraint may be added when desired.
- [Kotlin documentation: return types](https://kotlinlang.org/docs/functions.html#return-types) — Return types can be inferred for single-expression functions, while block-bodied functions generally declare them. This shows a syntax-based middle ground, though flang already uses blocks for function bodies.

# Answers

## A. Named functions declare non-unit result types

```flang
fn double(number: Int) -> Int {
    number * 2
}
```

Omitting `-> Type` would mean unit. Callers can learn a function's result type from its signature without reading its body. Simple functions repeat a type that the compiler could infer.

## B. Infer result types when omitted

```flang
fn double(number: Int) {
    number * 2
}
```

An annotation remains allowed, but omission means inference, not unit. This reduces repetition. Recursive functions, separate compilation, or unclear inference may still need an annotation; the exact boundary would require another rule.

## C. Infer private results; require public result types

```flang
fn double_local(number: Int) {
    number * 2
}

pub fn double(number: Int) -> Int {
    double_local(number)
}
```

Internal code stays concise, while the public interface states its result type. This depends on a future visibility and module design and may make changing a private function's visibility require more annotations.

# Selected answer

**A.** Named functions declare a non-unit result type explicitly. Omitting the result type means unit. This does not yet decide whether local variable types can be inferred.
