---
date: 2026-07-22 16:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-beta.7: We Pointed the Tools at Ourselves

beta.6 shipped `kestrel fmt`, `lint`, `doc`, and `test`. The natural next step was
to run them across our own Kestrel code — the examples and the standard library —
and wire that into CI so those repositories stay clean on every change. Doing that
found a real bug in the formatter. beta.7 fixes it.

<!-- more -->

## The bug: a formatter must never change meaning

The one rule a formatter cannot break is this: reformatting code must never change
what it does. It may move whitespace around, but the program has to mean exactly
the same thing afterward.

`kestrel fmt` broke that rule on multi-line strings. A string can span several
lines:

```kestrel
str report = "
    Name: {name}
    Score: {score}
"
```

The formatter worked one line at a time and re-indented each line to its brace
depth. But the indented lines here are *inside a string* — that leading whitespace
is part of the text. Re-indenting them silently rewrote the string's contents.

The fix teaches the formatter to track when it is inside a string that continues
onto the next line — the same way it already tracks multi-line block comments. A
line that begins inside a string is now left exactly as written, while the code
around it still gets tidied. `kestrel fmt` is idempotent again, and safe on every
example we have.

## Why this happened — and why that's the point

We didn't find this by staring at the formatter. We found it by *using* it: turning
`kestrel fmt --check` and `kestrel lint` loose on the whole examples repository and
the standard library. One file — a demo of multi-line string interpolation — was
exactly the shape that tripped the bug.

That is the whole argument for a language shipping its own tools and then using
them on itself. The examples and the standard library are the code new users read
first; keeping them consistently formatted and lint-clean matters. And the act of
enforcing that is what pulls bugs like this into the light.

## Staying solid from here

Both the [examples](https://github.com/kestrel-build/examples) and the standard
library now run a style check on every push: `kestrel fmt --check` on every file,
and `kestrel lint` on every file that parses. If a contribution drifts out of
format or picks up a lint, CI says so before it lands.

## Get it

```bash
curl -fsSL https://raw.githubusercontent.com/kestrel-build/kestrel/main/install.sh | sh
```

Every release is checksummed and signed. If you're already on beta.6, this is a
drop-in upgrade — the only change is a `kestrel fmt` that no longer touches the
inside of your multi-line strings.
