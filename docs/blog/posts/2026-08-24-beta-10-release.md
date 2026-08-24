---
date: 2026-08-24 12:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-beta.10: Build Once, Run Anywhere

Most of the time you build a program on the same kind of machine you run it on —
write it on your laptop, run it on your laptop. But sometimes the machine you are
building *for* is not the one in front of you: a Raspberry Pi, a router, an old
server, a phone. beta.10 makes that a single command.

<!-- more -->

## One command, eleven CPUs

```bash
kestrel build --target aarch64 -o app main.kst
```

That builds a program for a 64-bit ARM chip — from an ordinary x86 laptop. No
extra steps, no fiddling with toolchains by hand.

It works because the compiler doesn't hard-wire a particular CPU into your
program until the very last moment. Think of it like a writer who drafts an idea
in plain terms first, then hands it to a translator to put into whatever language
the reader needs. `--target` just names the language.

Eleven of them ship in this release:

`x86_64`, `aarch64` (64-bit ARM), `ppc64le`, `riscv64`, `armv7` (32-bit ARM),
`s390x`, `i686` (32-bit x86), `powerpc` (32-bit), `mips`, `mipsel`, and
`mips64el`.

That list covers every combination that matters: 32-bit and 64-bit, and both
byte orders (the two ways a CPU can lay out numbers in memory — "little-endian"
and the older "big-endian"). Each one is checked the honest way — we cross-build
a real program *and run it* on an emulator of that chip, on every change, so a
target on this list will not quietly stop working.

## A profile for writing, a profile for shipping

A build you're still working on and a build you're ready to hand out want
different things. So `kestrel build` now has two modes.

Plain `kestrel build` is the **draft**: fast, and it keeps all the labels on — the
function names and debug markers a debugger needs when something goes wrong. It's
the manuscript with notes in the margins.

`kestrel build --release` is the **finished book**: optimized, and stripped of
everything a stranger doesn't need. It also removes the small tells that would
reveal the program was written in Kestrel — the internal names and messages are
compiled out — so a release binary looks like any other C program. (To be clear:
this hides *information*; the safety and security protections are on in every
build, always.)

You can set the default and tweak the details in a `kestrel.toml` file, so your
team just types `kestrel build` and gets the right thing:

```toml
[build]
profile = "draft"

[profile.release]
opt-level = "2"
strip = true
```

## A bug we're glad we caught

Getting those eleven CPUs right turned up a real one. On a 32-bit, big-endian
chip, a program would crash the moment it allocated memory.

The cause is a nice illustration of why "the same code" isn't the same on every
machine. When the program asked the system for a chunk of memory, it passed along
the *size* as a 64-bit number. But on a 32-bit chip, the size is only expected to
be 32 bits — half as wide. On a big-endian machine, the half the system actually
read was the *wrong* half: zero. So the program asked for a zero-sized box and
then tried to put things in it. Boom.

The fix was to always hand the size across through a small helper that knows the
right width for the machine it's on. With that in place, all eleven targets run
correctly.

## Also new

- `kestrel dump-tokens` and `kestrel dump-ast` show you exactly how the compiler
  reads your file — the first tools you reach for when a program parses in a way
  you didn't expect.
- `kestrel build --verify-ir` runs a quick self-check on the compiler's internal
  work before it produces a binary, catching malformed output early.
- A new fuzzer runs on every change, generating random programs and checking that
  the two compilers (the original and the self-hosted one) agree on every one.

## Get it

```bash
curl -fsSL https://raw.githubusercontent.com/kestrel-build/kestrel/main/install.sh | sh
```

Binaries for all six shipped architectures are attached to the
[GitHub release](https://github.com/kestrel-build/kestrel/releases/latest), each
checksummed and signed. And once it's installed, that one binary can cross-compile
your programs to any of the eleven — see
[Cross-compilation](../../reference/cross-compilation.md).
