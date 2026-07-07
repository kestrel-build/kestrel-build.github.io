---
date: 2026-07-03 09:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-alpha.3 Released

Alpha.3 is about *shapes*: the ways you group data together. Structs, methods,
and arrays all land in this release -- plus a subtle but important fix to how
`&&` and `||` behave.

<!-- more -->

## Structs

Group related values into a named type, read and write their fields, and nest
them freely:

```kestrel
struct Point {
    float64 x
    float64 y
}

struct Line {
    Point start
    Point end
}

func main() -> void {
    Line l = Line {
        start: Point { x: 0.0, y: 0.0 },
        end:   Point { x: 3.0, y: 4.0 }
    }
    println("end.x = {l.end.x}")
}
```

Ask for a field that doesn't exist, or give one the wrong type, and it's a
compile error.

## Methods

Structs can carry behavior. A method takes an implicit `this` receiver, passed
by reference -- so when a method changes a field, the change sticks:

```kestrel
struct Counter {
    int32 count

    pub func bump(this) -> void {
        this.count = this.count + 1
    }
}
```

## Fixed-size arrays -- with real bounds checks

Arrays get literals, indexing, and iteration. The important part: **indexing is
bounds-checked at runtime**. Reach past the end and the program aborts with a
clear message instead of quietly reading someone else's memory:

```kestrel
func main() -> void {
    int32[3] nums = [10, 20, 30]
    for (n in nums) {
        println("{n}")
    }
    println("first = {nums[0]}")
}
```

When the index is a constant the compiler can prove is in range, it skips the
check -- safety where you need it, no cost where you don't.

## Short-circuit `&&` and `||`

Previously `&&` and `||` evaluated *both* sides. Now they short-circuit like
they should: the right side runs only when the left side doesn't already decide
the answer. This matters when the right side has an effect or guards against the
left:

```kestrel
if (p != null && p.is_ready()) { ... }   // p.is_ready() only runs when p != null
```

## Try it out

- [GitHub Releases](https://github.com/kestrel-build/kestrel/releases)
- [Examples repository](https://github.com/kestrel-build/examples)
- [Documentation](https://kestrel-build.github.io)
