---
date: 2026-07-22 18:00:00
authors:
  - kestrel
categories:
  - Release
  - Milestone
---

# Kestrel v1.0.0-beta.8: FFI, `unsafe` — and the Language Is Frozen

This is a milestone release. beta.8 adds the last two things the language was
missing — calling C, and touching raw memory — and with them the v1 feature set
is **complete**. From here, the Rust compiler is **frozen**: bug fixes only, no
new language features, until it is replaced entirely by Kestrel.

<!-- more -->

## The escape hatch: FFI and raw pointers

A memory-safe language still has to talk to the world, and the world is written
in C. beta.8 adds `extern "C"` for calling C functions, and raw pointers —
`ptr[T]`, address-of `&x`, dereference `*p`, and store-through `*p = v`.

The key design choice is that these live behind a wall. Taking an address is
fine, but **dereferencing a raw pointer, writing through one, or calling a C
function is only allowed inside an `unsafe { }` block:**

```kestrel
extern "C" {
    func abs(int32 n) -> int32
}

func main() -> void {
    int32 x = 41
    unsafe {
        ptr[int32] p = &x
        *p = *p + 1            // read and write through the pointer
    }
    println("x is now: {x}")   // 42

    unsafe {
        println("abs(-7) = {abs(0 - 7)}")
    }
}
```

Everything outside `unsafe` still carries the full guarantee. The unsafe blocks
are exactly where a reviewer looks first — the boundary is explicit, small, and
searchable.

## The bigger news: the language is feature-complete

FFI + `unsafe` was the last feature on the list. Take stock of what Kestrel is
today: memory safety without a garbage collector, null safety, checked integer
overflow, taint tracking, effect and capability typing, constant-time and
crypto-sensitivity checks, contracts, structured concurrency with static
deadlock-order checking, generics and traits, modules with real visibility,
and a built-in formatter, linter, doc generator, and test runner — all in one
signed binary. And now C interop and a raw-memory escape hatch.

That is the v1 language. **We are done adding to it.**

## What "frozen" means, and what happens next

Kestrel has been able to compile *itself* for a while — the self-hosting
bootstrap holds a byte-for-byte fixpoint. What it couldn't do was compile every
*other* program, because the self-hosted compiler only implements a subset of the
language while the Rust compiler kept growing.

Freezing the language stops that chase. Now the self-hosted Kestrel compiler can
catch up to a fixed target, feature by feature, until it can compile everything
the Rust one can. When it can, **the Rust compiler gets deleted** — and the first
release candidate ships with zero Rust in it. Kestrel, built entirely by Kestrel.

That is the whole point of the project, and today is the day the finish line
stopped moving. The checklist from here is public and bounded. See you at rc.1.

## Get it

```bash
curl -fsSL https://raw.githubusercontent.com/kestrel-build/kestrel/main/install.sh | sh
```

Signed and checksummed, as always.
