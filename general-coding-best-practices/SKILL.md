---
name: general-coding-best-practices
description: 'General coding best practices inspired by "A Philosophy of Software Design" (John Ousterhout) and "Clean Code" (Robert C. Martin). Use when: designing function structure, deciding whether to extract a function, organizing pipeline/orchestrator functions, designing class responsibilities, or placing error handling and logging.'
---

# General Coding Best Practices

## AI Behaviour
1. Present me the questions about code change with the `vscode_askQuestions` tool.

## Function Design
- The main function must call other functions, acting as a pipeline orchestrator.
- Use try-catch to handle errors only in the main function. Always add a log entry in the catch block by default.
- If a query is small and simple, such as fetching only the user id, first name, and email, keep it inline in the main function; don't encapsulate it in a separate function.
- Prefer a few deep functions (rich behavior behind a simple signature) over many shallow ones that just rename or forward a single call.
- Each function should do one thing, but "one thing" means one coherent responsibility, not the smallest possible line count.
- Keep parameter counts low; group related parameters into a DTO/object instead of adding more arguments.
- Avoid hidden side effects: a function should not silently mutate state outside its own scope.

## Class Design
- A class should have one reason to change (Single Responsibility Principle).
- Hide implementation details behind a small, simple public interface (information hiding); keep internal complexity out of the interface.
- Avoid "classitis": don't split a class into many tiny classes if that only adds indirection without reducing real complexity.
- Avoid god classes: if a class accumulates unrelated responsibilities, split it along those responsibilities.

## Comments
- Write comments only to explain what is not obvious from reading the code itself (the "why", not the "what"). Don't restate what the code already shows.
- Prefer good names and clear structure over comments; use a comment when the code cannot express the reasoning by itself (e.g., a non-obvious design tradeoff).

## References

These two books sometimes disagree; combine them rather than following either dogmatically:
- **"A Philosophy of Software Design", John Ousterhout** — favors deep modules, information hiding, and being skeptical of over-decomposition.
- **"Clean Code", Robert C. Martin** — favors small functions, single responsibility, and expressive naming.
- Where they conflict (e.g., function size), prioritize whichever reduces overall cognitive complexity for the reader in context, not a fixed line-count rule.

