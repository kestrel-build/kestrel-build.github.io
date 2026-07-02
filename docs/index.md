# Kestrel

**Memory-safe. Taint-aware. Effect-typed.**

Kestrel is a systems programming language that makes security properties part of the type system — not an afterthought. It combines C-style familiarity with Rust-level safety guarantees, and adds security dimensions that no other systems language has built in.

## Why Kestrel?

Most security bugs fall into a few known categories: buffer overflows, use-after-free, SQL injection, timing side-channels. Kestrel's type system catches all of these at compile time, without runtime overhead.

- **Memory safety** — no dangling pointers, no use-after-free, automatic deallocation
- **Taint tracking** — user input is tainted at the source; the compiler refuses to let it reach a sink without sanitization
- **Effect types** — functions declare what they do (`@io`, `@network`, `@pure`); untrusted code cannot sneak in side effects
- **Constant-time types** — `secret[T]` values cannot leak through timing

## Install

```bash
curl -fsSL https://kestrel-build.github.io/install.sh | sh
```

## Hello, Kestrel

```kestrel
func main() -> int32 {
    str name = "world"
    printf("Hello, {name}!\n")
    return 0
}
```

## A taste of the safety model

```kestrel
// The compiler tracks that 'input' came from user data
@tainted str input = read_line()

// This would be a compile error — tainted data cannot reach a SQL sink
// db.query("SELECT * FROM users WHERE name = '{input}'")

// Sanitize first, then use
str safe = sanitize(input)
db.query("SELECT * FROM users WHERE name = '{safe}'")
```

## Get started

- [Installation guide](getting-started.md)
- [Language guide](language-guide/overview.md)
- [Security model](security/overview.md)
- [Example programs](https://github.com/kestrel-build/examples)
