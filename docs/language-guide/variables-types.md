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
