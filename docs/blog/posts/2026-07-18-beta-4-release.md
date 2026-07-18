---
date: 2026-07-18 18:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-beta.4: Null Safety, the Kotlin Way

The billion-dollar mistake, refused at compile time. Beta.4 adds **nullable
types** — a value that might be `null` is a *different type*, and the compiler
makes you check before you use it.

<!-- more -->

## A value, or nothing

A `T?` holds either a `T` or `null`:

```kestrel
int32? maybe = null
str?   name  = "Kestrel"
```

You cannot use a `T?` as a `T` by accident. This is a compile error, not a
crash later:

```kestrel
int32? x = 5
int32  y = x        // error: cannot use int32? as int32 — check for null first
```

## The compiler narrows for you

Once you rule out `null`, the compiler *knows* the value is there, and lets you
use it as the plain type — no ceremony:

```kestrel
func classify(int32? n) -> str {
    if (n == null) {
        return "none"
    }
    // from here on, `n` is a plain int32
    if (n > 0) {
        return "positive: {n}"
    }
    return "non-positive: {n}"
}
```

That works inside an `if (n != null) { … }` block too, and after an early
`return` guard like the one above.

## Two operators for the common cases

When you just want a fallback, use `??`:

```kestrel
int32? missing = null
println("{missing ?? 0}")     // 0
```

When you are certain it is not null and want to say so, use `!!` (it panics at
runtime if you are wrong):

```kestrel
int32? forced = 7
println("{forced!!}")          // 7
```

## And it stays memory-safe

A nullable string owns its text like any other string — you never free
anything, and there are no leaks. We check every example under Valgrind, and the
nullable ones are clean across assignment, reassignment, loops, narrowing, `??`,
and `!!`.

## What's next

One piece of the null story is still on the way: the safe-access operator `?.`
(`node?.value`, `name?.len()`). It reports a clear "not yet" for now, and it's
the very next thing we're building. Everything else about null safety is here in
beta.4 — and, as always, the release is [GPG-signed](2026-07-18-beta-3-release.md)
and the compiler still compiles itself.
