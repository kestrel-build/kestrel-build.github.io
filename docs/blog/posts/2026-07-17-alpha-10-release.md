---
date: 2026-07-17 12:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-alpha.10: Generic Lists, and Exceptions That Clean Up After Themselves

Alpha.10 is a "sand off the rough edges" release. Two edges in particular: you
can now write generic functions over `List[T]`, and `try`/`catch`/`finally`
finally behaves the way you expect on every path — including the awkward ones,
like returning straight out of a `try`.

<!-- more -->

## Generics that understand lists

Kestrel has had generic functions for a while — `func identity[T](T x) -> T`
and friends. But the moment you tried to be generic over a *list* of something,
the compiler stopped you:

```kestrel
func first[T](List[T] xs) -> T {   // "List[<generic>] is not supported yet"
    return xs.get(0)
}
```

That gap is closed. The type parameter is now matched *structurally*: when you
call `first(nums)` with a `List[int32]`, the compiler looks inside the list
type, binds `T` to `int32`, and stamps out a specialized copy of the function
just for that case. Call it again with a `List[str]` and you get a second copy.
Each copy is a concrete function, so strings are still cloned and freed exactly
right — the whole thing is leak-free under valgrind.

```kestrel
func doubled[T](List[T] xs) -> List[T] {
    List[T] out = list_new()
    for (x in xs) { out.push(x) }
    for (x in xs) { out.push(x) }
    return out
}
```

The same matching works for `ptr[T]`, `T?`, and other wrappers — anywhere the
type parameter is nested one layer deep instead of sitting bare.

### A new standard-library module: `std.list`

With that in place, the standard library grows a genuinely generic list module.
Where `std.collections` has to spell out the element type (`sum_i32`,
`sum_f64` — you can't add up strings), `std.list` doesn't care what's inside:

```kestrel
import std.list.{first, last, contains, index_of, reverse, take}

bool has = contains(names, "kestrel")   // works for List[str]
int32 i  = index_of(nums, 42)           // ...and List[int32]
List[str] r = reverse(names)
```

It ships with `first`/`last`, `contains`/`index_of`/`count`, and
`copy`/`concat`/`fill`/`reverse`/`take`/`drop` — the small, everyday operations
you'd otherwise rewrite in every project.

## finally, done properly

`try`/`catch`/`finally` looks simple, but the corner cases are where languages
quietly cheat. This release fixes three of them.

**Returning out of a `try` now works.** Before, you had to stash your answer in
a variable and return *after* the block. Now you can just return — and the
`finally` still runs, and the exception machinery is cleaned up so a later
`throw` still lands where it should:

```kestrel
func read_config(bool ok) -> int32 {
    try {
        if (ok) { return 200 }
        throw("missing config")
    } catch (Error e) {
        return 500
    } finally {
        println("closed config handle")   // runs on BOTH returns
    }
}
```

**A `try`/`finally` with no `catch` no longer eats the exception.** Previously,
a block that only had a `finally` would run its cleanup and then carry on as if
nothing had gone wrong — swallowing the error. Now it runs the `finally` and
*re-raises*, so the exception travels on to the next handler out. That's the
whole point of a bare `try`/`finally`: guarantee the cleanup, but don't pretend
the failure didn't happen.

**And the leaks are gone.** A string (or list, or map) built inside a `try`
body used to leak if a `throw` jumped past it, because the jump skips the normal
end-of-scope cleanup. Those locals are now freed at the throw site, before the
jump. The droppability analysis was also simply not looking inside `try` blocks
at all, so any heap local declared in one leaked on *every* path — that's fixed
too, and the catch variable's own copy is now freed when the catch block ends.
The common paths are clean under valgrind.

## Under the hood

None of this disturbs the self-hosting story: the Rust seed still compiles the
Kestrel compiler, which still compiles itself to a byte-identical fixpoint, and
the full bootstrap — 24 programs through the self-built compiler — stays green.

As always: `try/catch/finally` is Kestrel's only error model (no `?` operator),
and memory is managed for you by NOVA. Alpha.10 just makes both of those a
little more honest.
