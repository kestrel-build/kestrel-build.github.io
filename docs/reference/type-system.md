# Type System Reference

## Primitive types

| Type | Description |
|------|-------------|
| `int32` | 32-bit signed integer |
| `int64` | 64-bit signed integer |
| `uint32` | 32-bit unsigned integer |
| `uint64` | 64-bit unsigned integer |
| `float32` | 32-bit IEEE 754 float |
| `float64` | 64-bit IEEE 754 float |
| `bool` | Boolean (`true` / `false`) |
| `str` | UTF-8 string |

## Compound types

| Syntax | Description |
|--------|-------------|
| `T[N]` | Fixed-size array of N elements of type T |
| `(T, U)` | Tuple |
| `func(T, U) -> V` | Function type |
| `ptr[T]` | Raw pointer (unsafe) |
| `ptr[T]?` | Nullable raw pointer |

## Generic types

```kestrel
struct Pair[T, U] {
    T first
    U second
}

enum Option[T] {
    Some(T)
    None
}
```

## Casting

```kestrel
cast[T](val)              // safe numeric cast
unsafe_cast[T](val)       // unsafe reinterpret / downcast (unsafe)
const_cast[T](val)        // remove const
```

## Type aliases

```kestrel
type Meters = float64
type NodeId = uint64
```
