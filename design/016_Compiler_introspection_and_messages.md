# Question

How much of the compiler's type and program model should compile-time flang code be able to inspect, and may it participate in compilation through a message loop?

The user wants Jai-like introspection and a compiler message loop explored. These are related, but need not be adopted together. Syntax below is illustrative only.

# Research

- [Jonathan Blow, *Jai Demo and Design Explanation* (keynote)](https://www.youtube.com/watch?v=IdpD5QIVOKQ) — Blow demonstrates compiler workspaces and a loop receiving compiler messages and forwarding them to plugins. This is a firsthand demonstration of the interaction model the user mentioned.
- [Zig language reference: `@typeInfo`](https://ziglang.org/documentation/master/#typeInfo) — Exposes type structure as a compile-time value; the reference includes inspection of enum field names. This is a narrower form of introspection that does not itself expose compiler events.
- [Question 014](014_Comptime_mutable_scope.md) — Establishes compile-time state inside mixed-stage functions, which introspection and compiler interaction could use.

# Answers

## A. Type introspection only

Compile-time code can inspect types and declarations, but cannot observe or modify compilation as a process.

```flang
comp fields = type_info(Player).fields;
```

The API is smaller and easier to make deterministic. Compiler plugins would require a separate host API.

## B. Type introspection plus read-only compiler messages

Compile-time code can inspect types and subscribe to compiler events, such as a type becoming available, but cannot change the program being compiled through those messages.

```flang
comp for message in compiler.messages() {
    if message is TypeChecked { inspect(message.declaration); }
}
```

This enables analysis tools and diagnostics; event ordering, error handling, and build reproducibility still need rules.

## C. Type introspection plus interactive compiler plugins

Compile-time code can receive compiler messages and respond with actions, potentially including generated declarations, diagnostics, or compilation options.

```flang
comp for message in compiler.messages() {
    if message is TypeChecked { compiler.emit(generate_for(message.type)); }
}
```

This is closest to the Jai-inspired direction. It offers broad metaprogramming power, but requires explicit boundaries for when code may be inserted, how plugins are ordered, and how incremental builds replay or invalidate their effects. Those are design questions rather than settled properties.

# Selected answer

Pending. The user has asked to explore this area, not selected an option.
