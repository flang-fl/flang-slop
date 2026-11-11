# Question

Is a payload-carrying union alternative's constructor itself a function value?

[Question 051, answer A](051_Union_variant_construction.md) requires construction to name the alternative, but leaves this point open. [Question 027](027_Trait_scoped_reexport.md) already establishes that ordinary functions are values. Consider applying the same named alternative to many payloads:

```flang
comp Message = union {
    Request: Bytes,
    Response: Bytes,
    Failure: ErrorInfo,
}

let make_request = Message.Request; // Is this a function value?
let message: Message = make_request(bytes);
```

The example assumes provisional `Message.Request(bytes)` construction syntax. For this question, a payload alternative has one payload type, so its constructor would have type `Bytes -> Message`. A payload-free alternative would be a `Message` value, not a zero-argument function, unless a later question chooses otherwise. The runtime representation of function values is separate.

# Research

- [Swift language reference: enumeration declarations](https://docs.swift.org/latest/documentation/the-swift-programming-language/declarations/) — Swift explicitly permits taking a reference to an associated-value case as a function. Its example binds `Number.integer` with type `(Int) -> Number` and passes it to `map`.
- [Rust Reference: enumeration items](https://doc.rust-lang.org/reference/items/enumerations.html) — Tuple-like variants define constructors in Rust's value namespace, while unit-like variants are values. This distinguishes a constructor from an already constructed fieldless case.
- [Rust Reference: function item types](https://doc.rust-lang.org/reference/types/function-item.html) — Referring to a tuple-like variant constructor yields a zero-sized function item value that can be called or converted to a function pointer. Flang could expose a constructor as a function value without requiring a stored function pointer for every use.
- [Question 051, answer A](051_Union_variant_construction.md) — Naming the alternative remains required whether the constructor is called immediately or passed as a value.

# Answers

## A. Every payload constructor is a function value

`Message.Request` is a function value of type `Bytes -> Message`. It can be bound, passed, returned, or called like any other function. The construction still names the alternative, possibly at the place where the constructor value was obtained rather than the call site.

```flang
let make_request = Message.Request;
let message: Message = make_request(bytes);
```

This fits the existing decision that functions are values and makes mapping or adapting collections of payloads convenient. It may require decisions about generic constructors and function identity, but those questions also affect ordinary functions.

## B. Constructors can only be used directly to construct

`Message.Request(bytes)` constructs a value, but `Message.Request` alone is invalid. To pass the operation around, the programmer writes an explicit function or closure. The spelling below is illustrative because closure syntax is not selected.

```flang
let make_request = fn(bytes: Bytes) -> Message {
    Message.Request(bytes)
};
```

This keeps a clear distinction between a case name and a function. It costs a wrapper for a common operation, and the wrapper can obscure the original constructor name in diagnostics or tooling.

## C. Require explicit conversion to a function value

Direct construction works. A separate operation, provisionally `constructor(Message.Request)`, produces a `Bytes -> Message` function value when one is needed.

```flang
let make_request = constructor(Message.Request); // Provisional operation.
let message: Message = make_request(bytes);
```

This makes the change of role explicit while avoiding a handwritten wrapper. It adds a special operation and a distinction that may feel arbitrary if ordinary named functions can already be used as values.

# Selected answer

**A.** Every payload-carrying union alternative's constructor is a function value. For example, `Message.Request` has type `Bytes -> Message` and may be bound, passed, returned, or called like an ordinary function. The alternative remains explicitly named when that constructor value is obtained. The treatment of payload-free alternatives and the runtime representation of constructor function values remain open.
