---
date: 2026-07-15 16:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-alpha.8: A Standard Library, Real Modules, and a Growing Self-Hosted Compiler

Yesterday Kestrel learned to compile itself. Today's release is about what a
language does the morning after: it gets a standard library, its modules
grow real walls, and the self-hosted compiler keeps absorbing the language
it is written in — hash maps, floating point, string interpolation, proper
error messages with line numbers.

<!-- more -->

## The standard library, written in Kestrel

There is now a [std repository](https://github.com/kestrel-build/std) with
four implemented, tested modules — `string`, `math`, `collections`, and
`io` — all pure Kestrel. Even `sqrt` is Newton's method in Kestrel, no C
library in sight:

```kestrel
import std.string.{trim, to_upper, split}
import std.math.{gcd, sqrt}

func main() -> void {
    println(to_upper(trim("  kestrel  ")))   // KESTREL
    List[str] parts = split("a,b,c", ",")    // 3 parts
    println("gcd: {gcd(48, 36)}")            // 12
    println("sqrt2: {sqrt(2.0)}")            // 1.41421
}
```

And the compiler knows how to find it: point `KESTREL_STD` at the std tree
(or ship a `std/` directory next to the binary) and `import std.*` resolves
like any local module. Everything still compiles from source as one
program — no binary library format, no linker surprises.

## Modules with real walls

Until now, `pub` was a promise Kestrel didn't check. The self-hosted
compiler now enforces the whole story, with line-numbered errors:

```
kestrel: line 8:  struct 'Hidden' is private to module 'lib.a' - mark it pub
kestrel: line 7:  struct 'Vis' is not imported by module 'app.b' - add it to an import line
kestrel: line 12: method 'peek' of struct 'Box' is private to module 'lib.a' - mark it pub
kestrel: line 12: field 'v' of struct 'Box' is private to module 'lib.a' - mark it pub
```

Functions, structs, enums, methods, and — new in this release — individual
**fields** are private by default; crossing a module boundary requires `pub`
and an import. And modules are now true namespaces: two modules may each
have their own private `helper()` without colliding — the compiler mangles
emitted names per module and resolves calls to your own module first, then
to what you imported.

## The subset closes in on the language

The compiler-written-in-Kestrel gained, since alpha.7:

- **`HashMap[str, V]`** — including passing maps to functions (borrowed, so
  the callee's inserts persist), returning them (an owned copy), storing
  them in struct fields, and `m2 = m1` deep copies. Valgrind-clean, like
  everything else.
- **`for` loops and compound assignment** (`+=` and friends) — pure
  parse-time sugar; the compiler's own source now uses them.
- **`float64`, string interpolation, and hard diagnostics** — an unknown
  name, a private field, a duplicate function: each is a clear error with a
  line number instead of a silent mis-parse.

Every commit, CI rebuilds the compiler with itself and checks the result is
byte-identical, then runs **19 programs through the self-built compiler** —
including its own three unit-test suites and eight programs that must be
*rejected* with exactly the right error message.

## Getting alpha.8

Binaries for x86_64 and aarch64 Linux are on the
[GitHub release](https://github.com/kestrel-build/kestrel/releases/tag/v1.0.0-alpha.8),
with SHA256SUMS. The std library lives in the
[std repository](https://github.com/kestrel-build/std); set `KESTREL_STD` to
its checkout to use `import std.*` today.

Next: per-module types, richer std (the `kernel` bindings are waiting on
freestanding codegen), and the long road of teaching the self-hosted
compiler the rest of its own language.
