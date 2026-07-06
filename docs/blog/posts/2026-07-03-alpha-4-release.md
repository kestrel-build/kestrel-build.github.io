---
date: 2026-07-03
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-alpha.4 Released

Alpha.4 is a release about *releases*. The headline is a compatibility fix that
lets the compiler run on the machines people actually have -- plus the tooling
that makes every future release something you can trust.

<!-- more -->

## Runs on the LLVM you already have

Kestrel emits LLVM IR and hands it to `llc`. The catch: LLVM changed its IR
between versions, and Debian 12 and Ubuntu 22.04 LTS -- what a huge number of
people run -- ship **LLVM 14**. Earlier alphas quietly assumed something newer.

Alpha.4 fixes this. The compiler now produces IR that links cleanly on **LLVM 14
through 18**:

- it passes `-opaque-pointers` when it detects LLVM 14, and
- it numbers unnamed IR values the exact way LLVM 14 insists on.

We also added a script that reproduces the precise release environment
(`rust:1.80` / LLVM 14) in a container, so "works on my machine" stops being a
mystery.

## A one-line install

```sh
curl -fsSL https://raw.githubusercontent.com/kestrel-build/kestrel-public/main/install.sh | sh
```

The installer detects your platform, downloads the right binary, and
**verifies its checksum** before putting it on your PATH.

## Every release is proven, not promised

Two new habits back every release from here on:

- **The shipped binary is tested, not just the source.** A public workflow
  downloads the actual release artifact and compiles + runs a suite of real
  programs against it. A status badge shows whether what you can download
  actually works -- independent of our private build pipeline.
- **Release notes come from the changelog.** The notes you read on a release are
  generated straight from `CHANGELOG.md`, so they always match what changed.

## Why alpha.2 and alpha.3 felt quiet

Alpha.2 and alpha.3 built the language core (semantic checking, strings, enums,
structs, methods, arrays) but never fully published -- a gap in the release
toolchain we've now closed. Alpha.4 bundles all of their work plus the release
machinery above. If you're coming from alpha.1, you're getting everything at
once.

## Try it out

- [GitHub Releases](https://github.com/kestrel-build/kestrel/releases)
- [Examples repository](https://github.com/kestrel-build/examples)
- [Documentation](https://kestrel-build.github.io)
