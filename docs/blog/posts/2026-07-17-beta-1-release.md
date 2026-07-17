---
date: 2026-07-17 18:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-beta.1: Out of Alpha

Kestrel is leaving alpha. `v1.0.0-beta.1` is the first beta — the point where the
language has all three of the things we said had to be true before we'd call it
one: a real module system, real generics, and a real error model, on top of a
compiler that already compiles itself.

<!-- more -->

## What "beta" means here

We set the bar for leaving alpha a long time ago, and it was deliberately
concrete — no moving goalposts:

- **Modules** — multi-file builds, per-module namespaces. ✓
- **Generics** — generic functions, generic structs, generic `List[T]`, and now
  **traits**. ✓
- **Error model** — `try` / `catch` / `finally`, with the corner cases (return
  out of a `try`, re-raising from a bare `try`/`finally`) actually correct. ✓
- **Self-hosting** — the trigger to leave alpha. The Kestrel compiler compiles
  itself to a byte-identical fixpoint, and has since alpha.7. ✓

The one piece still missing on that list was traits. This release lands them, so
here we are.

## Traits

A trait is a shared interface: a set of methods a type promises to provide. You
declare one, implement it for your types, and then write code against the trait
instead of against any one type.

```kestrel
trait Describable {
    func name() -> str
    func describe() -> str {     // a default — impls may inherit this
        return "a thing"
    }
}

impl Describable for Dog {
    func name() -> str { return "Dog" }
    func describe() -> str { return "a dog that says {this.sound}" }
}

impl Describable for Cat {
    func name() -> str { return "Cat" }   // inherits the default describe()
}
```

Trait **bounds** on a generic parameter let one function work over every type
that implements the trait:

```kestrel
func announce[T: Describable](T x) -> void {
    println("here is a {x.name()}")
}
```

Two things worth saying about how this works under the hood, because they're
choices, not accidents:

- **Dispatch is static.** Each call to `announce` is monomorphized — the
  compiler stamps out a specialized copy for each concrete type it's called
  with, and the method call goes straight to that type's implementation. No
  vtable, no pointer chase, no runtime cost. It's the zero-overhead path.
- **Dispatch is by name, not by shape.** If two different structs happen to have
  the same field layout, generic dispatch still calls the right one. (That
  sounds obvious; it's the kind of thing a structural backend gets wrong if you
  don't take care, so we took care — and there's a regression test for exactly
  that case.)

Default methods are inherited when an impl doesn't override them, an `impl` that
forgets a required method is a compile error, and inherent `impl` blocks (no
trait) let you hang plain methods on a type.

## Also in this release

- **A miscompile fix**: calling a method directly on a call that returns a
  `List` or `HashMap` — `make_list().len()` — used to silently read an empty
  container and return 0. It now works (and no longer leaks the temporary).

## What beta doesn't mean

Beta means the core language is here and coherent, not that it's finished. The
road to `1.0` still runs through concurrency (green threads), the full security
type system, cross-compilation, and a package manager. But the shape of the
language you'd write real programs in is now settled — and it compiles itself to
prove it.
