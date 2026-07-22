# Variables and Types

## Declarations

Variables are declared with the type before the name — C style, not Rust style.

```kestrel
int32 x = 42
int64 big = 1_000_000
float64 pi = 3.14159
str greeting = "hello"
bool flag = true
```

## Mutable by default

All variables are mutable. Use `const` to make something immutable:

```kestrel
const int32 MAX = 100
int32 counter = 0
counter = counter + 1   // ok
// MAX = 200            // compile error
```

## Numeric types

| Type | Size | Range |
|------|------|-------|
| `int32` | 32-bit signed | -2^31 to 2^31-1 |
| `int64` | 64-bit signed | -2^63 to 2^63-1 |
| `uint32` | 32-bit unsigned | 0 to 2^32-1 |
| `uint64` | 64-bit unsigned | 0 to 2^64-1 |
| `float32` | 32-bit float | IEEE 754 |
| `float64` | 64-bit float | IEEE 754 |

No implicit numeric conversions. Use `cast[T]()` to convert explicitly:

```kestrel
int32 x = 42
int64 y = cast[int64](x)
```

## Integer overflow is checked

Arithmetic that overflows its type is a bug — and a frequent source of security
holes, where a wrapped size calculation leads to an undersized buffer. Kestrel
does not let it happen silently. Every `+`, `-`, and `*` on an integer (signed or
unsigned) is checked; if the result does not fit the type, the program stops with
a diagnostic instead of wrapping to a wrong value:

```kestrel
int32 x = 2147483647    // int32 max
int32 y = x + 1         // aborts: "integer overflow: add of two int32 values"
```

This applies to compound assignments (`+=`, `-=`, `*=`) too. In-range arithmetic
is unaffected, and the check is cheap — it compiles to an add plus a
jump-on-overflow, not a software division.

### Opting out: `wrapping_*`

Some algorithms *want* modular arithmetic — hashes, checksums, PRNGs, ring-buffer
indices. For those, ask for wrapping explicitly with `wrapping_add`,
`wrapping_sub`, and `wrapping_mul`:

```kestrel
// FNV-1a hash — defined in terms of wrapping arithmetic, so it says so.
func fnv1a(str s) -> uint32 {
    uint32 hash = 2166136261
    int64 i = 0
    while (i < s.len()) {
        hash = wrapping_mul(hash, 16777619)
        hash = wrapping_add(hash, cast[uint32](s.byte_at(i)))
        i += 1
    }
    return hash
}
```

The default is safe; wrapping is opt-in and visible at the call site.

## Arrays

```kestrel
int64 numbers[5] = [1, 2, 3, 4, 5]
int64 first = numbers[0]
```

Array access is bounds-checked at compile time where possible.

## Type aliases

```kestrel
type Meters = float64
type Seconds = float64

Meters distance = 100.0
Seconds time = 9.58
```
