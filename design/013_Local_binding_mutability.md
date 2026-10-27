# Question

Can a local name be assigned a new value after its initial binding? Does the same mutability permission also control mutation of data through that binding?

This concerns reassignment of the name, such as `counter = counter + 1`, and mutation through the binding, such as `point.x = 2`. Whether aliases to the same data can bypass a binding's restriction is a separate question. All flang syntax below is provisional.

# Research

- [Rust Reference: variables](https://doc.rust-lang.org/reference/variables.html) — Local bindings are immutable unless marked `mut`; `let mut counter = 0` permits reassignment.
- [Zig language reference: variables](https://ziglang.org/documentation/master/#Variables) — `const` and `var` explicitly distinguish bindings that cannot and can be reassigned.
- [OCaml documentation: mutability and references](https://ocaml.org/docs/mutability-imperative-control-flow#references) — A `let` name is immutable. Mutable state is available through reference cells, whose contents can change while the binding itself stays fixed.
- [WG14 C committee draft N843: type qualifiers](https://open-std.org/jtc1/sc22/wg14/www/docs/n843.htm) — C permits ordinary objects to be modified and provides `const` qualification to restrict modification, illustrating a mutable-default approach.

# Answers

## A. Immutable by default; explicitly mutable bindings

```flang
let limit = 10;
let mut counter = 0;
counter += 1;

let mut point = Point(1, 2);
point.x = 3;
```

Most names stay fixed; reassignment and direct mutation through a binding are visible where it is declared. A plain `let point` would prohibit `point.x = 3`. The language still needs separate rules for aliases and references.

## B. Mutable by default; explicitly immutable bindings

```flang
let counter = 0;
counter += 1;
const limit = 10;
```

Reassignment is concise, especially in loops, but a reader cannot tell from `let` alone whether the name will change later.

## C. Bindings never change; mutable state lives in cells or objects

```flang
let counter = Cell(0);
counter.set(counter.get() + 1);
```

Names have one meaning throughout their scope. Mutation is explicit in the type and access operations, at the cost of extra indirection or syntax for a simple counter.

# Selected answer

**A, extended to mutation through the binding.** `let` forbids both rebinding the name and mutating data through that binding. `let mut` permits both. Whether data reachable through aliases remains mutable, and how references express that permission, are separate questions. The user also wants corresponding compile-time forms; see [Question 014](014_Comptime_mutable_scope.md).
