---
date: 2026-07-22 12:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-beta.6: Safer by Default, and a Toolbox to Match

Beta.5 finished null safety. Beta.6 hardens the everyday paths — arithmetic,
assumptions, cleanup, and module boundaries — and hands you the tools every
project reaches for on day one.

<!-- more -->

## Integer overflow traps instead of wrapping

`int32 x = 2147483647; x + 1` used to silently become `-2147483648`. That is the
quiet start of a whole class of security bugs — a wrapped size calculation that
allocates a buffer that is too small. In beta.6 it stops the program instead:

```
integer overflow: add of two int32 values
```

Every `+`, `-`, and `*` on an integer is checked, signed and unsigned, in
expressions and compound assignments. The in-range path stays cheap (an add plus
a jump-on-overflow). When you *want* modular arithmetic — hashes, checksums,
ring buffers — say so:

```kestrel
uint32 hash = wrapping_mul(hash, 16777619)
```

## Contracts: state your assumptions, and hold them

`@requires` and `@ensures` attach a precondition and a postcondition to a
function. Both are checked at run time; inside `@ensures`, `result` is the
returned value. A broken assumption halts at the boundary that stated it.

```kestrel
@requires(n >= 0)
@ensures(result >= 1)
func factorial(int32 n) -> int32 {
    if (n <= 1) { return 1 }
    return n * factorial(n - 1)
}
```

## `defer`, block-scoped

`defer` schedules cleanup that runs when its block exits — on the happy path, an
early `return`, a `break`/`continue`, or a `throw` — last-in-first-out, and
before the block's locals are freed. It is block-scoped, so a `defer` in a loop
body fires once per iteration:

```kestrel
for (i in 0..2) {
    defer println("close {i}")
    println("open {i}")
}
// open 0 / close 0 / open 1 / close 1 / open 2 / close 2
```

## Copies are deep copies — no aliasing surprises

Copying a struct, list, or map now duplicates the heap it owns, so the copy is
fully independent of the original. Mutating one never touches the other, and each
is freed exactly once. (This also closed a double-free that could happen when two
names shared one buffer.)

## Module boundaries are real

A declaration is **private to its module** unless you mark it `pub`. Reaching for
a private name from another module is a compile error — your public surface is
exactly what you chose to expose, and everything else stays yours to change.

## One binary, four tools

The toolchain now formats, lints, documents, and tests:

```bash
kestrel fmt --check src/*.kst   # canonical formatting (CI-friendly)
kestrel lint src/*.kst          # naming + style checks
kestrel doc src/vector.kst      # Markdown API reference from /// comments
kestrel test math_test.kst      # run the file's @test functions
```

`kestrel fmt` is idempotent and preserves your comments; `kestrel doc` pulls the
prose from `///` comments straight into the reference.

## Get it

```bash
curl -fsSL https://raw.githubusercontent.com/kestrel-build/kestrel/main/install.sh | sh
```

As always, every release is checksummed, signed, and verified against the public
[example suite](https://github.com/kestrel-build/examples) before it ships.
