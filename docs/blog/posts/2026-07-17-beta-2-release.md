---
date: 2026-07-17 21:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-beta.2: The Trait System, Finished — and Signed Releases

Beta.1 landed traits. Beta.2 *finishes* them — impls on primitives, generic
impls, dynamic dispatch, and associated types — and turns on **GPG-signed
releases** so you can verify what you download.

<!-- more -->

## Signed releases

The build pipeline now GPG-signs the checksum file for every tagged release.
Each release ships a `SHA256SUMS.asc` signature and the public key
(`kestrel-signing-key.asc`) right alongside the binaries, so you can verify a
download end to end without hunting anything down:

```bash
# Download the binary, SHA256SUMS, SHA256SUMS.asc, and kestrel-signing-key.asc
# from the release, then:
gpg --import kestrel-signing-key.asc
gpg --verify SHA256SUMS.asc SHA256SUMS   # authentic checksums
sha256sum -c SHA256SUMS                  # binaries match the checksums
```

A `Good signature from "KestrelBuild"` means the bytes you have are the bytes we
built.

## The trait system is complete

Beta.1 shipped the core: trait declarations, impls, default methods, and static
(monomorphized) dispatch with trait bounds on generics. Beta.2 fills in the four
remaining pieces.

**Impls on primitives.** You can implement a trait for `int32`, `bool`, `str`,
and the rest — the receiver `this` is the value itself:

```kestrel
impl Describable for int32 {
    func describe() -> str { return "the int {this}" }
}
```

**Generic impls.** Implement a trait for a generic struct across every element
type at once; the methods are monomorphized alongside the struct:

```kestrel
impl[T] Show for Box[T] {
    func show() -> str { return "Box holding {this.value}" }
}
```

**Dynamic dispatch — `dyn Trait`.** When you need one value that could be any
implementer, chosen at runtime, use a trait object:

```kestrel
dyn Speak a = Dog { sound: "grr" }
dyn Speak b = Robot { id: 7 }
println(a.say())   // dispatched through a vtable
println(b.say())
```

A `dyn Trait` is a fat pointer `{ data, vtable }` — the C++ virtual-call model,
but non-intrusive: a plain `Dog` stays plain data and only carries a vtable when
used as `dyn`. The trait object owns its boxed value and frees it (and any owned
fields) automatically.

**Associated types.** A trait can declare a type each impl fills in — resolved
at compile time, zero runtime cost, just like a C++ member typedef:

```kestrel
trait Container {
    type Item
    func first() -> Item
}
impl Container for IntBox {
    type Item = int32
    func first() -> int32 { return this.value }
}
```

And where the two features meet, Kestrel enforces **object safety** the same way
C++ and Rust do: a trait with associated types can't be used as `dyn` (the type
would be erased) — a clean compile error, never a silent miscompile.

## Design choices, on the record

Two calls worth stating plainly, both aimed at staying close to C/C++:

- **Static by default, dynamic on request.** Ordinary trait calls cost nothing
  extra — they compile to a direct call. You only pay for a vtable when you
  write `dyn`.
- **`dyn` is a vtable, not a tagged union.** It's the mechanism C++ programmers
  already know (and the Linux kernel's `struct file_operations` is a hand-rolled
  version of it), and unlike a closed tagged union it doesn't need to know every
  implementer up front.

The full write-up is in [`docs/TRAITS.md`](https://github.com/kestrel-build/kestrel).

As always: memory is managed for you (every trait object and boxed value is
freed automatically, valgrind-clean), the compiler still compiles itself, and
the whole thing is now signed.
