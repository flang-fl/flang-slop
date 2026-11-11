# Question

Given an `enum T` value, when can a program retrieve the `T` value associated with its active option?

[Question 045, answer A](045_Enums_and_tagged_unions.md) says a runtime enum value stores only a compact option identifier, while each option's associated `T` value is known at compile time. For example:

```flang
comp Colors = enum Color {
    RED = Color.new(255, 0, 0);
    GREEN = Color.new(0, 255, 0);
    BLUE = Color.new(0, 0, 255);
}

fn paint(choice: Colors) {
    let color: Color = choice.value; // Proposed access spelling; is this allowed?
    draw(color);
}
```

If `choice` is known during compilation, the compiler knows the associated `Color`. If `choice` is known only at runtime, the compiler would need to generate a choice-dependent lookup or branches to produce `Color`. A further complication is that `T` might be a compile-time-only kind of value, such as a type or a compiler object, with no runtime representation. The exact access spelling and compiler lowering are separate questions.

# Research

- [Swift language reference: raw-value enums](https://docs.swift.org/latest/documentation/the-swift-programming-language/declarations/) — A Swift raw-value enum has a `rawValue` property, so code can retrieve the raw value from an enum case. Swift restricts eligible raw-value types; flang's proposed `enum T` is broader and must address types that cannot exist at runtime.
- [Zig language reference: `@tagName` and `@typeInfo`](https://ziglang.org/documentation/master/) — Zig exposes an enum's active name through `@tagName` and its fields through compile-time type information. These are examples of distinct runtime-facing and compile-time inspection paths; neither directly specifies arbitrary associated `T` values for flang.
- [Rust book: `match` on enum variants](https://doc.rust-lang.org/book/ch06-02-match.html) — An exhaustive `match` can compute a runtime value from a runtime-known enum case. Flang could use the same mechanism explicitly or generate equivalent choice-dependent code behind an associated-value access operation.
- [Question 046, answer A](046_Duplicate_enum_values.md) — Distinct enum options may hold equal `T` values. Access from option to value is therefore unambiguous, even though a reverse lookup may not be.

# Answers

## A. Access at either stage when the result can exist there

An enum option known at compile time yields its associated `T` value during compilation. A runtime-known enum value may also yield `T` when `T` has a runtime representation; the compiler generates a lookup or equivalent branches. If `T` exists only at compile time, accessing it from a runtime-known enum value is a compile-time error. The spelling, cost model, and exact rules for runtime-representable types remain open.

## B. Associated values are compile-time metadata only

Direct access to an associated `T` value requires the enum option itself to be known during compilation. Runtime code must explicitly match every option and construct whatever runtime result it needs. This makes staging and cost visible, but repeats information already present in the enum declaration.

## C. Every enum's `T` must be runtime-representable

Restrict `enum T` so all associated values can be returned from a runtime enum value. Access always works, including when the option is runtime-known. This gives a simple promise to users but excludes enums of types, functions with compile-time-only identity, and other compile-time-only data.

# Selected answer

**A.** An enum option known at compile time yields its associated value during compilation. A runtime-known enum value can yield its associated value when the associated type has a runtime representation; the compiler generates a choice-dependent lookup or equivalent code. Accessing a compile-time-only associated value from a runtime-known option is a compile-time error. Access spelling and lowering remain open.
