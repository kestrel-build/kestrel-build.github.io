---
date: 2026-07-02 15:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-alpha.2 Released

Alpha.1 could compile a program. Alpha.2 makes it a *language*. This release
fills in the core of everyday Kestrel: a real type checker, strings that work,
enums with pattern matching, and the first piece of what makes Kestrel
different -- taint tracking.

<!-- more -->

## A real semantic pass

Alpha.1 checked types loosely. Alpha.2 adds a full second pass that actually
holds the line:

- **Name resolution** -- every identifier must refer to something real.
- **No implicit conversions** -- an `int32` is not a `float64`. If you want to
  cross that line, you say so with `cast[T]()`. The compiler never guesses.
- **`const` means const** -- assigning to a `const` is an error, not a warning.
- **Honest errors** -- anything the backend can't yet compile is reported as a
  clear "not supported yet" error instead of silently producing a wrong program.

That last point is a rule we hold ourselves to: **no miscompiles by
construction**. If the compiler can't do something correctly, it refuses,
loudly.

## Strings that work

Strings now go end to end -- interpolation, concatenation, and equality:

```kestrel
func main() -> void {
    str name = "Kestrel"
    str greeting = "Hello, " + name
    println(greeting)
    if (name == "Kestrel") {
        println("Names match!")
    }
}
```

Under the hood, strings are length-prefixed `{length, data}` values -- not
null-terminated -- backed by a small native runtime.

## Enums and `match`

Enums carry a backing type, and `match` checks that you handled every case:

```kestrel
enum Color : int32 {
    Red = 0
    Green = 1
    Blue = 2
}

func describe(Color c) -> str {
    match (c) {
        case Color.Red:   return "warm"
        case Color.Green: return "calm"
        case Color.Blue:  return "cool"
    }
}
```

Miss a case and the compiler tells you before your program ever runs.

## The first taste of security

Taint tracking -- the feature that makes Kestrel *Kestrel* -- is now wired into
the compiler. Mark where untrusted data enters (`@tainted`), where it gets
cleaned (`@sanitizer`), and where it must never arrive dirty (`@sink`). Let
tainted data reach a sink and the build fails.

## Also new

- `break` and `continue` in loops
- Type-aware `println` (picks the right format for strings, integers, floats)
- A constant-folding optimizer pass
- `kestrel new` to scaffold a project
- An end-to-end example suite that compiles **and runs** every example, checking
  its output -- because green unit tests once coexisted with a compiler that
  couldn't print "Hello".

## Try it out

- [GitHub Releases](https://github.com/kestrel-build/kestrel/releases)
- [Examples repository](https://github.com/kestrel-build/examples)
- [Documentation](https://kestrel-build.github.io)
