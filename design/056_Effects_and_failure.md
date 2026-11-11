# Question

How does a function declare operations beyond computing a value, and how do recoverable failures move through calls?

This concerns I/O, mutation, allocation, and failure. [Question 044](044_Error_propagation_and_inference.md) already explores error inference. Syntax below is provisional.

# Research

- [Moggi, *Notions of Computation and Monads* (1991)](https://doi.org/10.1016/0890-5401(91)90052-4): separates values from effectful computations.
- [Plotkin and Pretnar, *Handling Algebraic Effects* (2013)](https://arxiv.org/abs/1312.1399): models operations and caller-selected handlers.
- [Bauer and Pretnar, *Programming with Algebraic Effects and Handlers* (2012)](https://arxiv.org/abs/1203.1539): develops these ideas as language constructs.

# Answers

## A. Result values for failure; other effects mostly implicit

Recoverable failure uses `Result<T, E>` in signatures. I/O and mutation need no general effect annotation. Simple signatures do not reveal every possible side effect.

```flang
fn load(path: Path) -> Result<Text, IoError> { /* ... */ }
```

## B. Track selected effects in function types

Function types state important effects, such as I/O or failure. Pure functions are distinguishable. The design must decide effect composition and public API rules.

```flang
fn load(path: Path) -> Text !{io, fail<IoError>} { /* ... */ }
```

## C. General effect operations and handlers

Functions request abstract operations; a surrounding handler defines their meaning. This is flexible but adds substantial semantic and implementation work.

```flang
fn load(path: Path) -> Text !{Read} { perform Read(path) }
handle load(path) with TestFileSystem
```

# Selected answer

A, with qualification: recoverable failures are represented by explicit result values such as `Result<T, E>`. I/O, mutation, and similar operations do not require a general effect annotation in every function type. This does not decide how functions obtain authority to use `stdin`, `stdout`, files, or other external resources; see [Question 062](062_External_capabilities.md). Convenient error propagation and public error-type inference remain open in [Question 044](044_Error_propagation_and_inference.md).
