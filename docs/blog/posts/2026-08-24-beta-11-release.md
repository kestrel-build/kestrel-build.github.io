---
date: 2026-08-24 18:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-beta.11: The Compiler Catches More of Your Mistakes

The whole promise of Kestrel is that the compiler catches your mistakes before
your users do. beta.11 makes it catch a few more — the shipped compiler now
rejects the same bad programs the reference compiler always has, and explains
them the same way.

<!-- more -->

## A little background

Kestrel is written in Kestrel. There are really two compilers: the original one
(written in Rust, the "reference"), and the self-hosted one (written in Kestrel,
which is what you download). For a long time the self-hosted compiler could
*build* everything the reference could — but it was quietly a bit more lenient:
a handful of clearly-wrong programs slipped through it that the reference would
have stopped.

For a language whose pitch is "the compiler is on your side," being *less* strict
than the reference is exactly the wrong direction. beta.11 closes the gap.

## What it now catches

Four kinds of mistake that used to slip through the shipped compiler now stop it:

**Returning the wrong type.** A function that promises to return an `int32` but
hands back a `true`:

```kestrel
func f() -> int32 {
    return true      // error: return type mismatch
}
```

**A `match` that forgets a case.** If you match on an enum, you have to handle
every variant (or add a `case _`). Miss one, and the compiler says so — instead
of silently letting a value fall through with nowhere to go:

```kestrel
match (color) {
    case Color.Red: paint_red()
    // error: match is not exhaustive — Green and Blue aren't handled
}
```

**A contract that isn't a yes/no question.** `@requires` and `@ensures` are
boolean conditions; anything else is now rejected:

```kestrel
@requires(n)         // error: a `@requires` contract must be a `bool` expression
func f(int32 n) -> int32 { return n }
```

**A struct built with a field missing.** Leave one out and the compiler names it,
instead of a confusing punctuation error:

```kestrel
Point p = Point { x: 1 }   // error: struct literal for `Point` is missing field `y`
```

## Clearer wording, too

A few messages were reworded to match the reference exactly — an unknown name now
reads `` undefined variable `x` ``, and a type that needs a `cast[T]()` says "no
implicit conversions." Small, but it means every tool and tutorial describes the
same error the same way.

## The proof

There's a public suite of example programs — real code and deliberately-broken
code — that every release is checked against. As of beta.11 the shipped,
self-hosted compiler passes **all 55 of them**, the broken ones included. What
the reference rejects, the compiler you download now rejects too.

## Get it

```bash
curl -fsSL https://raw.githubusercontent.com/kestrel-build/kestrel/main/install.sh | sh
```

Nothing in your working code needs to change — if your programs were correct,
they still compile. beta.11 only turns a few *incorrect* programs from "compiles
anyway" into "here's what's wrong."
