# Question

How does code access the payload of a tagged union when its active alternative is not known at compile time?

[Question 051, answer A](051_Union_variant_construction.md) requires construction to name an alternative. [Question 045, answer A](045_Enums_and_tagged_unions.md) requires exhaustive matching. Reading a payload raises a separate question: must code establish the active alternative through a pattern, or can it project a named alternative directly?

```flang
comp Message = union {
    Request: Bytes,
    Response: Bytes,
    Failure: ErrorInfo,
}

fn handle(message: Message) {
    // Could an expression such as `message.Request` read a payload here?
    // If so, what happens when `message` is not Request?

    match message {                         // Provisional pattern syntax.
        Message.Request(bytes) => process(bytes),
        Message.Response(bytes) => send(bytes),
        Message.Failure(error) => report(error),
    }
}
```

The question concerns **reading** a payload. Whether a pattern binding copies, borrows, or moves it, and whether assigning a payload can change the active alternative, are separate questions. The field and pattern spellings here are illustrative.

# Research

- [Rust Reference: match expressions](https://doc.rust-lang.org/reference/expressions/match-expr.html) and [patterns](https://doc.rust-lang.org/reference/patterns.html) — A variant pattern checks the case and binds its fields in the selected arm. The match arm's binding mode determines whether a payload is copied, moved, or borrowed. This makes the successful case check part of payload access.
- [Swift language guide: enumerations](https://docs.swift.org/latest/documentation/the-swift-programming-language/enumerations/) — Associated values are extracted with `switch` case patterns or an `if case` pattern. This illustrates a concise conditional form without treating the payload as an unconditional field.
- [Zig language reference: unions and tagged unions](https://ziglang.org/documentation/master/#Tagged-union) — Zig permits direct field access and `switch` capture. Its reference says accessing a non-active union field is safety-checked illegal behavior; this is a different failure policy from returning an optional payload.
- [Question 049, answer B](049_Union_tag_representation.md) — The compiler may encode the tag in the payload representation. Any access rule must use the union's semantic alternative, rather than assuming a separate tag field exists.

# Answers

## A. Access through a successful pattern only

A `match` arm, conditional pattern, or equivalent destructuring form checks the alternative and binds its payload. A plain expression like `message.Request` is invalid when `message` has union type, even after a separate tag comparison. A user-defined helper can perform a match and return an optional payload if desired. This keeps all built-in access tied to the check but can repeat a pattern for a simple query.

```flang
if case Message.Request(bytes) = message { // Illustrative conditional pattern.
    process(bytes);
}
if message is Message.Request {            // Illustrative separate tag test.
    process(message.Request);               // Invalid under A; use a pattern.
}
```

The benefit is a simple rule: the syntax that checks the alternative also introduces the payload. Changing a union later cannot turn a formerly safe direct read into a possible mismatch. The cost is that callers must use a pattern even when they already tested the tag, and a reusable one-case query needs a helper.

## B. Allow direct access when control flow proves the alternative

Patterns always work. A direct access such as `message.Request` also works where the compiler can prove that `Request` is active, for example inside the true branch of a tag test. Elsewhere it is a compile-time error. This can make code shorter, but requires rules for when assignments, aliases, and calls invalidate that proof. The tag-test spelling and proof rules remain open.

```flang
if message is Message.Request { // Illustrative tag test.
    process(message.Request);  // Allowed: this branch proves the tag.
}
// `message.Request` is invalid here: the tag is not proven.
```

The benefit is that a tag test and later payload use can be written separately, which may read naturally when the branch does several things. The cost is a flow-sensitive compiler rule. For example, if `message` can be reassigned or mutated through an alias inside the branch, the compiler must know when the proof no longer holds. Calls and concurrency may complicate that rule once ownership and aliasing are designed. A pattern binding avoids many of these questions by naming the payload at the check.

## C. Provide a safe conditional projection

Patterns always work. A separate expression, provisionally `message.Request?`, checks the alternative and returns an optional payload: present for `Request`, absent otherwise. Plain `message.Request` remains invalid without proof. This makes one-case queries concise and avoids a runtime failure, but commits the language to an optional-result form and to how payload ownership flows through it.

```flang
match message.Request? { // Illustrative projection and optional syntax.
    Some(bytes) => process(bytes),
    None => {},
}
```

The benefit is an ordinary value representing either a payload or a mismatch. Callers can pass it to helpers or compose it with other optional operations. The cost is another built-in operation alongside patterns. Its result may need to borrow the payload rather than copy or move it, and that choice depends on the language's later ownership rules. For a simple branch, it may be more ceremony than a conditional pattern.

## D. Permit checked direct projection with failure on mismatch

Patterns always work. `message.Request` checks the active alternative at runtime and yields the payload if it matches; a mismatch stops execution. This is concise when the programmer expects a specific alternative, but a mistaken expectation becomes a runtime failure. A later explicit fallible projection could still be added.

```flang
fn process_expected_request(message: Message) {
    process(message.Request); // Stops execution if this is Response or Failure.
}
```

The benefit is compact code at boundaries where a specific case is guaranteed by the surrounding protocol, even if the compiler cannot prove it. The cost is that the function's type does not show its precondition, and an unexpected case becomes a runtime failure. To make this answer precise, flang would need to decide whether the check is guaranteed in every build and how it reports the failure. A pattern or explicit fallible operation remains preferable when mismatch is ordinary input.

# Selected answer

**A.** Reading a tagged union payload requires a successful alternative pattern, such as a `match` arm or conditional pattern. Direct projection from a union value, such as `message.Request`, is invalid, even after a separate tag test. The exact pattern syntax and whether binding copies, moves, or borrows the payload remain open.
