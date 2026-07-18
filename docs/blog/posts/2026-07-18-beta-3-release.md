---
date: 2026-07-18 12:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-beta.3: Signed, for Real

Beta.2 promised signed releases. Beta.3 delivers them — this is the first
Kestrel release you can cryptographically verify.

<!-- more -->

## What changed

The build pipeline now GPG-signs every tagged release. Each release carries two
new files next to the binaries: a `SHA256SUMS.asc` detached signature and the
public key, `kestrel-signing-key.asc`. The signing key lives in the CI secret
store and never touches the source tree.

That is the whole release, and it is an honest one. Beta.2 announced signing but
shipped just before the pipeline was finished, so it went out unsigned — a
footnote we added to that post. Beta.3 is the fix: signing is on, and it stays
on for every release from here.

## Verify a download

Grab the binary, `SHA256SUMS`, `SHA256SUMS.asc`, and `kestrel-signing-key.asc`
from the release, then:

```bash
gpg --import kestrel-signing-key.asc
gpg --verify SHA256SUMS.asc SHA256SUMS   # the checksums are authentic
sha256sum -c SHA256SUMS                  # the binaries match the checksums
```

A `Good signature from "KestrelBuild"` means the bytes you downloaded are the
exact bytes we built. (`gpg` may also print a note that the key is not
"trusted" — that only means you have not personally vouched for it yet. The
signature is still valid.)

## Why bother this early?

Because trust is easier to build than to retrofit. A verifiable download chain
is the kind of thing you want in place *before* anyone runs Kestrel on something
that matters — not bolted on afterward. Everything else from beta.2 still holds:
the trait system is complete, memory is managed for you, and the compiler still
compiles itself.
