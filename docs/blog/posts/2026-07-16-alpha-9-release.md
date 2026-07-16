---
date: 2026-07-16 12:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-alpha.9: Exceptions, Real Interpolation, and a Bigger Standard Library

Alpha.9 fills in three things you reach for constantly: exception handling in
the self-hosted compiler, string interpolation that works *everywhere* (not
just inside `println`), and two more standard-library modules. Two of the
changes started as bug hunts — and the bugs they turned up were the good kind,
the kind that were quietly wrong for a while.

<!-- more -->

## try / catch / throw

The compiler-written-in-Kestrel can now handle exceptions:

```kestrel
func risky(int32 n) -> int32 {
    if (n < 0) {
        throw("negative input")
    }
    return n * 2
}

func classify(int32 n) -> str {
    str out = "?"
    try {
        int32 v = risky(n)
        out = "ok:{v}"
    } catch (str e) {
        out = "err:{e}"
    }
    return out
}
```

`throw` long-jumps to the innermost `try`, and the `catch` binds the thrown
message to its variable. Nested `try`/`catch` and re-throwing both work. It is
built on `setjmp`/`longjmp`, the same mechanism a C program would use — which
comes with the same caveat: don't `return` straight out of a `try` block
(assign a result and return after), and heap allocated on the throw path may
leak. Kestrel's own compiler reports errors by returning diagnostics, not by
throwing, so those limits are fine in practice; the feature exists so the
language's `try`/`catch` can itself be self-hosted.

## Interpolation that works as a value

String interpolation has been in Kestrel since early on — but only ever
*inside* `println`. This release found and fixed the gap: interpolating into a
value you keep.

```kestrel
str name = "kestrel"
str s = "hi {name}"                       // now works
str g = greeting(name, 3)                 // returned from a function
println(greeting("world", 0).len())       // passed along and measured
```

The old code path for "interpolation not in a println" was a stub that
returned only the first literal chunk — so `str s = "hi {name}"` printed `hi `
and, worse, later crashed when the automatic memory manager tried to free what
turned out to be a pointer into static memory. It now formats into a real
owned string. If you ever assigned an interpolated string to a variable and
got a truncated result, this is your fix.

## A bigger standard library

Two new modules land in the [std repository](https://github.com/kestrel-build/std):

- **`std.testing`** — a small unit-test framework (`expect`, `expect_eq_int`,
  `expect_eq_str`, `report`), the same shape the compiler's own tests use. It
  tests itself.
- **`std.os`** — named wrappers over the process and argument built-ins:
  `run` / `run_ok` to shell out, and `argc` / `arg_at` / `args` / `has_flag`
  to read the command line.

```kestrel
import std.testing.{expect_eq_int, report}
import std.os.{run, has_flag}

func main() -> int32 {
    int32 fails = 0
    fails += expect_eq_int(run("true"), 0, "true exits 0")
    return report(fails)
}
```

That brings std to six tested modules — `string`, `math`, `collections`,
`io`, `testing`, `os` — all pure Kestrel, and all resolvable with
`import std.*` once you point `KESTREL_STD` at the checkout.

## Under the hood

Both feature bugs surfaced while writing tests for the new work, and both were
memory-safety issues the compiler's own Valgrind-clean discipline caught
immediately: a thrown string literal was being freed as if it were owned, and
the interpolation stub was freeing static memory. Fixing them made the whole
thing tighter. As always, every commit rebuilds the compiler with itself,
checks the result is byte-identical, and runs a growing corpus — now **20
programs** — through the self-built compiler.

## Getting alpha.9

Binaries for x86_64 and aarch64 Linux are on the
[GitHub release](https://github.com/kestrel-build/kestrel/releases/tag/v1.0.0-alpha.9),
with SHA256SUMS. The standard library is in the
[std repository](https://github.com/kestrel-build/std).

Next: growing the standard library further, and continuing to teach the
self-hosted compiler the rest of its own language.
