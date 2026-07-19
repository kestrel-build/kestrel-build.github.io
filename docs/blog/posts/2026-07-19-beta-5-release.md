---
date: 2026-07-19 12:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-beta.5: Safe Access, and Null Safety Complete

Beta.4 made a value that might be `null` a *different type*. Beta.5 adds the last
operator — `?.` — and closes the loop: null safety is now complete.

<!-- more -->

## Reach in safely

The safe-access operator `?.` lets you read a field or call a method on a
nullable value without a manual check. If the value is there, you get the
result; if it is `null`, the whole expression is `null`:

```kestrel
str? name = "Kestrel"
int64 length = name?.len() ?? 0     // 7
```

It works on struct fields and struct methods too, and — like everything about a
nullable — the result is itself nullable, so you pair it with `??` to land on a
plain value:

```kestrel
Point? here = Point { x: 3, y: 4 }
Point? nowhere = null
println("{(here?.manhattan()) ?? 0}")     // 7
println("{(nowhere?.x) ?? 0}")            // 0
```

## Nullable structs, all the way through

Your own types can be nullable now — `Point? p` parses and works end to end:
declaration, `== null`, and flow narrowing that reaches *into* the value. After
a null check, you can read fields and call methods on it directly:

```kestrel
func describe(Point? p) -> str {
    if (p == null) {
        return "nowhere"
    }
    // `p` is a plain Point here — field access and methods just work
    return "at {p.x}, {p.y} (dist {p.manhattan()})"
}
```

And if that struct owns a string, it is freed for you when the nullable goes out
of scope — every nullable path is Valgrind-clean.

## The whole null story, in one place

That completes the set: `T?`, `null`, `==`/`!= null` with narrowing, `??`, `!!`,
and now `?.`. `examples/nullable.kst` shows all of it in one program. As always,
the release is GPG-signed and the compiler still compiles itself.
