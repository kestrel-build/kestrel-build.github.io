# Language Guide Overview

Kestrel is a statically typed systems language with C-style brace syntax and no semicolons. This guide walks through the core language features.

## Design goals

- **Readable** — curly-brace blocks, no semicolons
- **Safe by default** — memory safety, taint tracking, effect types built into the type system
- **C-compatible** — call any C library with zero overhead
- **Single binary** — one tool for build, run, test, format, lint

## Topics

- [Variables and Types](variables-types.md)
- [Functions](functions.md)
- [Control Flow](control-flow.md)
- [Structs and Classes](structs-classes.md)
- [Memory Safety](memory-safety.md)
- [Error Handling](error-handling.md)
- [Modules](modules.md)
