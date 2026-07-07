---
date: 2026-07-02 09:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-alpha.1 Released

The first alpha release of the Kestrel compiler is here. This marks a major milestone: the complete modular rewrite now compiles programs end-to-end, from source code all the way to a linked binary.

<!-- more -->

## What is Kestrel?

Kestrel is a memory-safe, taint-aware systems programming language. It combines C-style familiarity with compile-time safety guarantees that go beyond what any existing systems language offers: memory safety, taint tracking, effect types, constant-time enforcement, and more -- all checked at compile time with zero runtime overhead.

## What works in alpha.1

The full compilation pipeline is operational:

**Source -> Lex -> Parse -> Semantic Analysis -> IR Lowering -> LLVM Codegen -> Link -> Binary**

Features you can use today:

- Functions with parameters and return values
- `if` / `else` control flow
- Integer and float arithmetic
- String literals and interpolation
- `println` output
- Type checking across the full type system
- Borrow checking for memory safety

Here is a working Kestrel program you can compile right now:

```kestrel
func fibonacci(int32 n) -> int32 {
    if (n <= 1) {
        return n
    }
    return fibonacci(n - 1) + fibonacci(n - 2)
}

func main() -> int32 {
    int32 result = fibonacci(10)
    printf("fib(10) = %d\n", result)
    return 0
}
```

## 19-crate architecture

The compiler is splits into 19 independent crates, each with a clear responsibility:

| Layer | Crates |
|-------|--------|
| Foundation | `kestrel-common`, `kestrel-error`, `kestrel-ast`, `kestrel-symboltable` |
| Frontend | `kestrel-lexer`, `kestrel-parser`, `kestrel-semantic`, `kestrel-security` |
| Backend | `kestrel-irgen`, `kestrel-optimizer`, `kestrel-codegen`, `kestrel-linker` |
| Runtime | `kestrel-runtime` |
| Tools | `kestrel-formatter`, `kestrel-lint`, `kestrel-tester`, `kestrel-doc` |
| Orchestration | `kestrel-driver`, `kestrel-cli` |

This structure makes each component independently testable and replaceable. It also lays the groundwork for the compiler to eventually compile itself.

## 1,307 tests passing

The test suite covers every stage of compilation. Lexer, parser, semantic analysis, type checking, borrow checking, code generation -- all tested independently and in integration. Every commit runs the full suite.

## What comes next

The path from alpha to beta:

- **Loops**: `while` and `for` loop codegen
- **More types**: arrays, structs, enums
- **NOVA memory management**: automatic deallocation without a garbage collector
- **Taint tracking**: the security type system that makes Kestrel unique
- **Self-hosting**: the compiler compiling itself

## Try it out

- [GitHub Releases](https://github.com/kestrel-build/kestrel/releases) -- download the compiler
- [Examples repository](https://github.com/kestrel-build/examples) -- working programs to learn from
- [Documentation](https://kestrel-build.github.io) -- language guide and reference
- [Getting Started](../../getting-started.md) -- installation and first program
