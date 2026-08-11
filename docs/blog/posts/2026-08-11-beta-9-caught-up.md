---
date: 2026-08-11 18:00:00
authors:
  - kestrel
categories:
  - Release
  - Milestone
---

# Kestrel v1.0.0-beta.9: The Self-Hosted Compiler Caught Up

Two releases ago we froze the language and said the finish line had stopped
moving: the Rust compiler would sit still while the Kestrel-written compiler
caught up to it, feature by feature, until it could build every program the Rust
one could. beta.9 is the release where it caught up.

The self-hosted compiler now compiles **and runs the entire example corpus —
every single program, none skipped.** Everything the reference compiler can
build, the compiler written in Kestrel can build too.

<!-- more -->

## What "caught up" actually means

Kestrel has been able to compile *itself* for a long time. That sounds like the
whole game, but it isn't — the compiler's own source only uses part of the
language, so passing that test just proves the compiler can build one particular
program. The real question is whether it can build *everything else*.

So we kept a bag of test programs — small ones that each lean on a different
corner of the language: strings, maps, slices, generics, traits, concurrency,
error handling, unsafe code, and so on. The rule is simple and unforgiving: the
Kestrel-built compiler has to compile each one, run it, and produce exactly the
output the reference compiler does, down to the exit code.

At the start of this stretch it passed 48 of 65. Today it passes **all 65**, with
the self-hosting fixpoint still holding byte-for-byte. The gap is closed.

## Closing the gap

Most of the remaining programs failed on the same missing piece, so the work was
less "seventeen bugs" and more "a handful of real features." The big ones:

- **Real 64-bit integers.** This was the deep one. The compiler had been quietly
  pretending `int64` was just a 32-bit `int`, which works right up until you
  write a number that doesn't fit — like a five-billion-item count, or a hash
  seed. `int64` is now a true 64-bit type everywhere: in variables, in
  arithmetic, in lists, maps, arrays, and slices. Teaching the compiler this
  without breaking its ability to build *itself* — which uses `int64` in
  hundreds of places — was the trickiest change in the project so far.

- **Unsigned 32-bit integers and overflow that bites.** `uint32` arrived, along
  with the guarantee it exists to protect: a plain `int32` multiply that
  overflows now **stops the program** instead of silently wrapping to a wrong,
  often-exploitable value. If you actually want wrapping — for a hash or a
  checksum — you ask for it by name.

- **Slices.** A borrowed `T[]` window into an array: `nums[1..<4]` hands you a
  view over three elements without copying anything. Length-checked, iterable,
  and cheap to pass around.

- **Cleanup that keeps its promises.** `defer` now works with `println`, and a
  `finally` block runs even when a `return` leaves early — and if there's no
  `catch` to handle an error, the `finally` still runs and the error keeps
  travelling outward instead of vanishing.

Each of these went in with the whole corpus and the self-hosting fixpoint
re-checked at every step, so nothing that already worked quietly broke along the
way.

## What's left

The language is frozen. The self-hosted compiler can now build everything the
Rust compiler can. That leaves exactly one thing between here and the first
release candidate: **making the Kestrel-built compiler the one we actually
ship**, and then deleting the Rust one. rc.1 is Kestrel, built entirely by
Kestrel, with zero Rust inside — and for the first time, that's a packaging job,
not a language one.

The finish line isn't just standing still anymore. We're walking up to it.

## Get it

```bash
curl -fsSL https://raw.githubusercontent.com/kestrel-build/kestrel/main/install.sh | sh
```

Signed and checksummed, as always.
