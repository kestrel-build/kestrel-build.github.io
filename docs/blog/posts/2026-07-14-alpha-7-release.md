---
date: 2026-07-14 18:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-alpha.7: Kestrel Now Compiles Itself

This is the release we have been building toward since alpha.1: **Kestrel is
self-hosting**. There is now a Kestrel compiler written in Kestrel, it
compiles its own source code, and the compiler it produces is byte-for-byte
identical to the one that built it. That loop — a compiler building itself,
and getting the same answer — is the oldest proof of seriousness a language
can offer. C did it in 1973. Now Kestrel has done it too.

And it is fast: the self-hosted compiler builds a 7,000-line benchmark
**about twice as fast as `gcc -O0` builds the same program in C**.

<!-- more -->

## What self-hosting means (and why you should care)

Think of a language as a set of tools. The question every new language must
answer is: *are the tools good enough to build real things?* The most honest
test is to build the hardest thing the language's authors know how to build —
the language's own compiler — using nothing but the language itself.

That is what this release does. The pipeline works like this:

1. A frozen "seed" compiler (the original Rust implementation) builds
   `stage1` — the Kestrel compiler written in Kestrel.
2. `stage1` compiles its own source code, producing `stage2`.
3. `stage2` compiles the same source again, producing `stage3`.
4. We check that `stage2` and `stage3` are **byte-identical**.

If anything in the compiler were wrong — a mis-parsed expression, a
miscompiled loop, a dropped declaration — the copies would drift apart. They
don't. The fixpoint holds, and our CI re-proves it on every single commit,
along with 13 test programs (including the compiler's own unit-test suites
and a set of programs that must be *rejected* with the right error messages)
compiled and run by the self-built compiler.

The Rust implementation is now the *bootstrap seed*: it is kept only so a
fresh machine can build stage1. New compiler work happens in Kestrel.

## Faster than gcc

Along the way the self-compile went from 103 seconds to about 1 second. Two
things were responsible, and both are lessons in how much allocation behavior
matters:

- The code generator built its output with `body = body + chunk` — which
  re-copies the entire growing buffer on every append. Tens of thousands of
  appends made that O(n²): 70 of the 103 seconds were spent copying the same
  bytes over and over. The fix is the new `List[str].join()`: collect chunks
  in a list (each push is cheap) and stitch them together once at the end, in
  a single pass.
- The backend invoked LLVM at its default optimization level, produced
  assembly text, and recompiled the C runtime from source on *every* link.
  Now it goes straight to an object file with fast instruction selection and
  links against a cached runtime object.

The result, measured on identical 500-function programs (~7,000 lines each)
written in C and in Kestrel, end-to-end from source to runnable binary:

| Compiler | Time |
|----------|------|
| `gcc -O0` (C) | 0.81 s |
| **Kestrel, self-hosted** | **0.42 s** |
| Kestrel (Rust seed) | 0.58 s |

All three binaries print the same answer. (Honest footnote: both compilers
are at the no-optimization level, and Kestrel's self-hosted compiler does
less analysis than gcc. But "compiles its own language about 2× faster than
gcc compiles C" is a real number, and it is the floor, not the ceiling.)

## The language grew, because writing a compiler demands it

Self-hosting is a forcing function: every gap in a language shows up the
moment you try to write real code in it. This release ships everything the
compiler-in-Kestrel needed, so you get it too:

**`for` over lists** — the third form of Kestrel's one loop now works on
`List[T]`:

```kestrel
List[str] words = list_new()
words.push("kes")
words.push("trel")
for (w in words) {
    println(w)
}
for (i, w in words) {
    println("{i}: {w}")
}
```

The binding is a fresh copy of each element, and anything heap-owning is
freed automatically at the end of every iteration — even on `break` or
`continue`. Valgrind-clean, like every memory feature in Kestrel.

**`List[str].join()`** — flatten a list of strings in one pass. If you ever
find yourself writing `s = s + chunk` in a loop, use a list and `join()`
instead; your program stops being accidentally quadratic:

```kestrel
List[str] parts = list_new()
parts.push("Hello, ")
parts.push("Kestrel")
parts.push("!")
str greeting = parts.join()
```

**`fail(msg)`** — print to stderr and exit(1). The abort primitive every
command-line tool needs for fatal errors; the self-hosted compiler uses it
for its own diagnostics:

```kestrel
if (value < 0) {
    fail("value must be non-negative, got {value}")
}
```

## The compiler is also a program you can read

One more thing worth saying plainly. The Kestrel-in-Kestrel compiler — a
lexer, a recursive-descent parser, an LLVM IR code generator, and a driver —
is about 5,000 lines of ordinary Kestrel: structs with methods, enums with
payloads, `match`, lists, strings. It manages all of its memory through the
language's automatic ownership (NOVA); the generated compiler runs
leak-clean under Valgrind. When it finds a mistake in your program it says
things like:

```
kestrel: line 12: function 'secret' is private to module 'lib.a' - mark it pub
kestrel: line 3: unknown name 'typo'
```

A compiler is not a magic artifact. It is a program, and now it is a Kestrel
program.

## Everything else in alpha.7

The full changelog is long — this is the biggest release since alpha.1 — but
highlights beyond the headline:

- `kestrel stat`: line counts by language and kind across a source tree,
  including the self-hosting percentage.
- Compound assignment (`+=`, `-=`, `*=`, `/=`, `%=`) and `for` loops in the
  self-hosted subset, string interpolation, `float64` (including
  `List[float64]`), and module visibility with line-numbered diagnostics.
- The automated bootstrap check (`scripts/bootstrap.sh`) runs in CI on every
  commit: fixpoint + 13 programs through the self-built compiler in ~8 s.

## Getting alpha.7

Grab a binary from the
[GitHub release](https://github.com/kestrel-build/kestrel/releases/tag/v1.0.0-alpha.7)
(x86_64 and aarch64 Linux, with SHA256SUMS), or use the installer. Examples
live in the [examples repository](https://github.com/kestrel-build/examples).

What's next: growing the self-hosted compiler toward the full language —
finer-grained visibility, richer collections — and the standard library,
written in Kestrel, of course.
