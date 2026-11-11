# Question

Must construction of a tagged union value explicitly name its alternative, or may the compiler choose an alternative from the payload's type?

[Question 045, answer A](045_Enums_and_tagged_unions.md) says a union's alternatives have distinct names and may carry data. These remain distinct even if their payload types happen to be the same:

```flang
comp Message = union {
    Request: Bytes,
    Response: Bytes,
    Failure: ErrorInfo,
}

let bytes: Bytes = read_bytes();
let message: Message = Message.Request(bytes); // Explicit tag, provisional syntax.
// Should `let message: Message = bytes;` ever choose a tag automatically?
```

The construction syntax is not yet selected. This question concerns whether the alternative must be named, not whether a constructor can be passed as a function value or whether the payload itself needs a conversion.

# Research

- [Swift language reference: enumeration declarations](https://docs.swift.org/latest/documentation/the-swift-programming-language/declarations/) — A case carrying a value acts as a constructor function, such as `Number.integer(3)`. The case name remains explicit and can itself be referenced as a function value.
- [Rust Reference: enum variant expressions](https://doc.rust-lang.org/reference/items/enumerations.html) — Rust constructs data-carrying variants through the named variant's constructor or struct expression. The name identifies the chosen variant independently of its payload type.
- [Zig language reference: anonymous union literals](https://ziglang.org/documentation/master/#anonymous-union-literals) — Zig can infer the surrounding union type from context, but its literal still names the active field, such as `.{ .int = 42 }`.
- [Question 049, answer B](049_Union_tag_representation.md) — Physical tag optimization does not remove the semantic distinction between alternatives.

# Answers

## A. Always name the alternative

Construct `Message.Request(bytes)` or an equivalent explicitly tagged form. `Bytes` never converts to `Message` merely because one variant currently accepts it. Adding another `Bytes` alternative does not change existing construction sites. This is more verbose for simple one-payload unions, but the programmer always states the intended case.

## B. Infer the alternative when the payload type is unique

In a context expecting `Message`, a `Bytes` value becomes the sole `Bytes` alternative if exactly one exists. If multiple alternatives accept `Bytes`, construction is ambiguous and must name the case. This is concise initially, but adding a same-typed variant can make previously valid code fail. Conversions and generic payloads may make uniqueness rules more complex.

## C. Require an explicit union conversion, but infer its unique alternative

An operation such as `Message.from(bytes)` chooses an alternative only when exactly one accepts the payload. A bare `Bytes` value never converts implicitly. This signals construction without spelling a variant name, but still creates ambiguity when variants share a payload type and may require a rule about which payload conversions participate.

# Selected answer

**A.** Constructing a tagged union value must name its alternative explicitly. A payload value does not implicitly become a union value, even when only one alternative currently accepts its type. The exact construction syntax and whether alternative constructors are function values remain open.
