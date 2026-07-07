---
date: 2026-07-06 09:00:00
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-alpha.5 Released

Alpha.5 is the biggest step yet toward a compiler that can compile itself.
Programs can now span multiple files, arrays gained slices, structs travel
through functions safely, and kernel modules get the familiar `module_init` /
`module_exit` markers. Every heap-touching feature here was checked leak-free
under Valgrind.

<!-- more -->

## Projects, not just files

Until now, a Kestrel program was a single file. Now `kestrel build` understands
whole projects:

- point it at a **directory** and it compiles every `.kst` beneath it,
- point it at a **`kestrel.toml`** and it compiles that project's `src/`,
- point it at a **single file** and it pulls in the local modules that file
  `import`s, following the chain.

```kestrel
// src/greeting.kst
module greeter.greeting

func greeting(str name) -> str {
    return "Hello, " + name
}
```

```kestrel
// src/main.kst
module greeter.main
import greeter.greeting.{greeting}

func main() -> void {
    println(greeting("Kestrel"))
}
```

```sh
kestrel build .        # or: kestrel build kestrel.toml
```

Both modules link into one binary. (Per-module name privacy and an incremental
build cache come next; today the build merges everything into one program.)

## Slices

A slice is a lightweight, borrowed view into an array -- a length and a pointer,
no copying:

```kestrel
func sum(int32[] xs) -> int32 {
    int32 total = 0
    for (x in xs) { total += x }
    return total
}

func main() -> void {
    int32[5] nums = [10, 20, 30, 40, 50]
    int32[] middle = nums[1..<4]     // a view of [20, 30, 40]
    println("len = {middle.len()}")
    println("sum = {sum(middle)}")
}
```

Slices support `.len()`, indexing (bounds-checked against the slice's own
length), iteration, and passing to functions. Because a slice only *borrows*,
slicing an array of strings copies nothing and leaks nothing.

## Structs travel safely

Passing a struct to a function, or returning one, now does the right thing even
when the struct owns a string. A parameter is a copy the callee can change
without touching the caller's value, and strings are cloned or moved so nothing
is freed twice:

```kestrel
struct Person { str name, int32 age }

func older(Person p) -> Person {
    p.age = p.age + 1        // changes the copy only
    return p
}
```

## `for (i, item in arr)`

Iterate with an index and a value at once:

```kestrel
for (i, name in names) {
    println("{i}: {name}")
}
```

## Kernel modules, the familiar way

Kestrel can build real Linux kernel modules, and now the entry points read the
way kernel developers expect. `@module_init` and `@module_exit` are shorthands
that place your functions in the `.init.text` and `.exit.text` sections:

```kestrel
@module_init
pub func hello_init() -> int32 {
    printk("hello: loaded\n")
    return 0
}

@module_exit
pub func hello_exit() -> void {
    printk("hello: unloaded\n")
}
```

The general `@section(".name")` annotation now works too, and the section really
lands in the compiled binary.

## Proven leak-free

Owned strings, strings inside structs and arrays, `break`/`continue` cleanup,
struct-by-value, and slices were all run under Valgrind on the LLVM 14 image
that mirrors the release environment: **zero leaks, zero errors**.

## Try it out

- [GitHub Releases](https://github.com/kestrel-build/kestrel/releases)
- [Examples repository](https://github.com/kestrel-build/examples)
- [Documentation](https://kestrel-build.github.io)
