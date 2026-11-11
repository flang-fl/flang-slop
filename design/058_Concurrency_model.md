# Question

What is the core model for independent execution, communication, and shared mutable state?

This depends on [Question 054](054_Memory_and_aliasing.md). DiamondFire may need a different execution substrate; this asks what the language promises rather than which scheduler first ships. Syntax below is provisional.

# Research

- [Hoare, *Communicating Sequential Processes* (1978)](https://www.cs.cmu.edu/~crary/819-f09/Hoare78.pdf): treats parallel processes and explicit message exchange as program structures.
- [Boehm and Adve, *Foundations of the C++ Concurrency Memory Model* (2008)](https://doi.org/10.1145/1375581.1375591): motivates precise rules for atomics, ordering, and data races in shared memory.
- [Jung et al., *RustBelt* (2018)](https://people.mpi-sws.org/~dreyer/papers/rustbelt/paper.pdf): studies ownership safety in concurrent code.

# Answers

## A. Shared-memory tasks with checked data-race freedom

Tasks may share memory; safe mutation requires synchronization or proven disjoint access. This interacts deeply with aliasing.

```flang
spawn { synchronized(counter) { counter += 1 } }
```

## B. Isolated tasks communicating by messages

Each task owns mutable state; sending transfers or copies data. This simplifies local reasoning but may need a special path for efficient shared data.

```flang
let worker = spawn_worker()
worker.send(move job)
```

## C. No built-in concurrent execution initially

The core defines sequential execution. Target-specific libraries may offer tasks or threads later, with their safety and portability contracts left open.

# Selected answer

Pending.
