# Question

Can a value hide its concrete type at runtime, and what may safe code do with it?

The design seed leaves `Any` open. This separates explicit dynamic values from implicit dynamic typing. Syntax below is provisional.

# Research

- [Abadi et al., *Dynamic Typing in a Statically Typed Language* (1991)](https://research.google/pubs/dynamic-typing-in-a-statically-typed-language-2/): combines an explicit tagged `Dynamic` type with checked inspection.
- [Siek and Taha, *Gradual Typing for Functional Languages* (2006)](https://web.stanford.edu/class/cs242/materials/old/siek06__gradual.pdf): studies static and dynamic program regions joined by runtime checks.
- [Kennedy and Syme, *.NET Generics* (2001)](https://www.microsoft.com/en-us/research/wp-content/uploads/2001/01/designandimplementationofgenerics.pdf): shows how retained runtime type information interacts with generic execution.

# Answers

## A. No universal runtime type value in the core

Programs use closed unions or trait views. A later library could add type erasure. Heterogeneous data requires more manual design initially.

## B. Explicit `Any` with checked inspection

Packing retains a runtime type identity; safe inspection or downcast checks it. Ordinary statically typed code stays static. Packable types and ownership need rules.

```flang
let value: Any = pack(42)
if let n: i32 = value.downcast<i32>() { use(n) }
```

## C. Gradual typing with implicit dynamic boundaries

Dynamic values flow through ordinary expressions with runtime checks. This changes type-error behavior and performance expectations across the language.

```flang
let value: Any = external_data()
let result: i32 = value + 1 // implicit runtime check
```

# Selected answer

Pending.
