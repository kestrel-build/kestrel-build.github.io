---
date: 2026-09-29 18:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-beta.12: Safer, and Faster

Two things a systems language owes you: it shouldn't let you write a memory bug,
and it shouldn't make you pay for that safety in speed. beta.12 pushes on both —
the compiler catches two more kinds of mistake before your program ever runs, and
the code it produces got noticeably faster.

<!-- more -->

## Using something before it exists

You can declare a variable and fill it in later. But what if one path forgets to?

```kestrel
func pick() -> int32 {
    int32 x
    if (lucky()) {
        x = 10
    }
    return x        // error: `x` may be used before it is assigned
}
```

Before, a program like this could read whatever happened to be sitting in memory —
the classic source of "works on my machine, crashes on yours," and worse. Now the
compiler traces every path through your function, and if even one of them reaches a
read without a write first, it stops you and points at the spot. If you meant for
it to start at zero, say so (`int32 x = 0`) — but now that's your choice, not an
accident.

(Anything that lives on the heap and is declared without a value is zeroed, so an
early read can never hand back a dangling pointer. Safe by construction.)

## Dividing by zero the compiler can already see

If the compiler can tell the divisor is zero, there's no reason to wait until the
program runs to find out:

```kestrel
int32 bad  = 10 / 0        // error: division by zero
int32 also = x % (3 - 3)   // error: division by zero
```

Divisors it can't know until runtime still trap safely — this just moves the
obvious cases to compile time, where a mistake is cheap to fix.

## Faster, without giving up the checks

Kestrel checks things C doesn't: every array index is in bounds, every `+`, `-`,
and `*` is watched for overflow. Those checks are why a whole category of exploits
simply can't happen here. The catch has always been that checks cost time.

beta.12 gets most of that time back. For a lot of everyday code, the compiler now
*proves* that a check can never fire — and when it can prove it, it removes the
check. A loop that sums an array, a counter that can't overflow, an index that's
plainly in range: the safety is still guaranteed, but the machine code comes out
the same as the unchecked C you'd have written by hand. Several of our benchmarks
that used to run meaningfully slower than C now match it exactly.

You don't switch any of this on. Build your program, and it's just faster.

## Get it

```bash
curl -fsSL https://raw.githubusercontent.com/kestrel-build/kestrel/main/install.sh | sh
```

As always, correct programs keep working unchanged. beta.12 only turns a few
*incorrect* ones from "compiles, then misbehaves" into "here's the line" — and
makes the correct ones run faster on the way out.
