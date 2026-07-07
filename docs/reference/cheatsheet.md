# Kestrel Cheatsheet

A single-page reference for the Kestrel language and CLI. For deeper explanations
see the [Language Guide](../language-guide/overview.md).

---

## CLI commands

| Command | What it does |
|---------|--------------|
| `kestrel build <file>` | Compile a `.kst` file to a native binary |
| `kestrel run <file>` | Compile and run in one step |
| `kestrel check <file>` | Type-check only, no binary |
| `kestrel new <name>` | Scaffold a new project |
| `kestrel emit-ir <file>` | Print Kestrel IR (debugging) |
| `kestrel emit-llvm <file>` | Emit LLVM IR (`.ll`) |
| `kestrel clean` | Remove build artifacts |
| `kestrel version` | Print version |
| `kestrel help` | Print help |

### Options

| Option | Effect |
|--------|--------|
| `-o <output>` | Output binary path |
| `--release` | Build with optimizations |
| `--verbose` | Print timing information |
| `--version` | Print version |
| `--help` | Print help |

```sh
kestrel run hello.kst
kestrel build -o myapp --release main.kst
```

---

## Types

| Keyword | Meaning |
|---------|---------|
| `int32` `int64` | Signed integers (fixed width) |
| `uint32` `uint64` | Unsigned integers |
| `float32` `float64` | IEEE 754 floats |
| `bool` | `true` / `false` |
| `str` | UTF-8 string (owned, length-prefixed) |
| `T[N]` | Fixed-size array |
| `T[]` | Slice (borrowed view) |
| `List[T]` | Growable list |
| `HashMap[str, V]` | String-keyed hash map |
| `T?` | Nullable |
| `ptr[T]` | Raw pointer (`unsafe`) |

```kestrel
int32 x = 42
float64 pi = 3.14159
str name = "Kestrel"
bool active = true
int32 flags = zeroed()      // zero-initialize any type
auto n = 42                 // inferred type
```

**Mutable by default** — use `const` for immutability. No `let` / `mut`.

```kestrel
const int32 MAX = 100
```

---

## Functions

```kestrel
func add(int32 a, int32 b) -> int32 {
    return a + b
}

func greet(str name) -> void {       // -> void is required
    println("Hello, {name}!")
}

func identity[T](T val) -> T {       // generics use [T], not <T>
    return val
}
```

String interpolation: `"total = {a + b}"`. Booleans interpolate as `true`/`false`.

---

## Operators

| Group | Operators |
|-------|-----------|
| Arithmetic | `+` `-` `*` `/` `%` |
| Comparison | `==` `!=` `<` `<=` `>` `>=` |
| Logical | `&&` `\|\|` `!` (no `and`/`or`/`not`) |
| Bitwise | `&` `\|` `^` `~` `<<` `>>` |
| Assignment | `=` `+=` `-=` `*=` `/=` `%=` `&=` `\|=` `^=` `<<=` `>>=` |
| Nullable | `?.` (safe call) · `??` (default) |

---

## Control flow

```kestrel
if (x > 0) {
    // ...
} else if (x == 0) {
    // ...
} else {
    // ...
}

for (i in 0..10) { }        // inclusive range
for (i in 0..<10) { }       // exclusive range
for (item in list) { }
for (i, item in list) { }

while (cond) { }

match (color) {
    case Color.Red: handle_red()
    case Color.Blue: handle_blue()
    case _: handle_other()
}
```

One `for` loop covers all iteration — no `foreach`. Parentheses and braces are
required. Newline terminates a statement (no semicolons).

---

## Collections

```kestrel
// List[T]
List[int32] xs = list_new()
xs.push(10)
int32 v = xs.get(0)         // bounds-checked
int32 first = xs.first()
int32 last = xs.last()
int32 top = xs.pop()        // remove + return last
xs.sort()                   // ascending; numeric + str elements
xs.reverse()
bool has = xs.contains(10)
int64 at = xs.index_of(10)  // first match, or -1
bool empty = xs.is_empty()
int64 n = xs.len()
xs.clear()                  // empty it, keep capacity

// HashMap[str, V]
HashMap[str, int32] ages = map_new()
ages.insert("ada", 36)
int32 a = ages.get("ada")
bool known = ages.contains("ada")
```

Indexing is bounds-checked and aborts on out-of-range — there is no negative
indexing; use `first()` / `last()` for the ends.

---

## Structs, classes, enums

```kestrel
struct Point {              // private by default; pub / protected to expose
    float64 x
    float64 y
    pub func norm() -> float64 { return this.x * this.x + this.y * this.y }
}

class Dog : Animal {        // inheritance; `parent`, not `base`
    override func speak() -> void {
        parent.speak()
        println("Woof!")
    }
}

enum Color : int32 {        // backing type required
    Red = 0
    Green = 1
    Blue = 2
}
Point p = Point { x: 1.0, y: 2.0 }
```

`clone()` is a deep copy — modifying a clone never touches the original.

---

## Error handling

```kestrel
try {
    File f = open("data.txt")
} catch (IOError e) {
    log(e.message)
} finally {
    cleanup()
}

defer close(resource)       // scope cleanup (not error propagation)
throw("something failed")
```

`try/catch/finally` only — there is no `?` operator.

---

## Casting & conversions

```kestrel
float64 f = cast[float64](x)        // safe cast
int32 n = f.to_int32()              // conversion method
const_cast[T](v)                    // drop const
unsafe_cast[T](v)                   // reinterpret (unsafe)
int64 sz = sizeof(Point)
```

---

## Nullable types

```kestrel
str? maybe = null
if (maybe != null) {
    println(maybe.len())            // compiler narrows the type
}
int32 len = maybe?.len() ?? 0
```

---

## Concurrency

Seven keywords: `spawn` `chan` `atomic` `mutex` `rwlock` `spinlock` `semaphore`.

```kestrel
spawn worker()                      // green thread (Flight)
chan[int32] ch = chan()
atomic[int64] counter = atomic(0)
counter.fetch_add(1)

mutex[int32] m = mutex(0)
with (m.guard()) {                  // scoped lock, auto-unlocks
    // ...
}
```

---

## Memory (NOVA)

Automatic, leak-free memory with no garbage collector. Every value has an owner;
heap buffers are freed at scope exit. States used in diagnostics: **Declared,
Owned, Moved, Borrowed, Locked, Freed**.

```kestrel
unsafe {
    ptr[int32] p = &x
    int32 val = *p
}
```

---

## Annotations

| Annotation | Purpose |
|------------|---------|
| `@no_discard` | Caller must use the return value |
| `@no_return` | Function never returns |
| `@packed` / `@aligned(N)` | Struct layout control |
| `@inline` / `@cold` / `@hot` | Optimization hints |
| `@section("name")` | Place symbol in a linker section |
| `@deprecated("msg")` | Emit a deprecation warning |
| `@sensitive` | Encrypt value in the binary |
| `@tainted` / `@sanitizer` / `@sink` | Taint tracking |
| `@test` | Mark a test function |
| `@repr("C")` | C-compatible layout |

---

## Imports

```kestrel
import std.io.File
import std.collections.{HashMap, HashSet}
```

Use `import`, not `use`.
