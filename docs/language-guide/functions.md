# Functions

## Basic syntax

```kestrel
func add(int32 a, int32 b) -> int32 {
    return a + b
}
```

## Default parameters

```kestrel
func connect(str host = "localhost", int32 port = 8080) -> bool {
    // ...
    return true
}
```

## Named arguments

```kestrel
connect(port: 443, host: "example.com")
```

## Multiple return values via tuples

```kestrel
func divmod(int32 a, int32 b) -> (int32, int32) {
    return (a / b, a % b)
}

(int32 q, int32 r) = divmod(17, 5)
```

## Lambdas

```kestrel
func(int32, int32) -> int32 add = |a, b| a + b

// Multi-line lambda
func(int32) -> int32 process = |x| {
    int32 y = x * 2
    return y + 1
}
```

## Methods (UFCS)

```kestrel
struct Point {
    float64 x
    float64 y
}

func (Point p).distance() -> float64 {
    return sqrt(p.x * p.x + p.y * p.y)
}

Point origin = Point { x: 0.0, y: 0.0 }
float64 d = origin.distance()
```

## Const functions

Evaluated at compile time when all arguments are constants:

```kestrel
const func factorial(int32 n) -> int32 {
    if (n <= 1) {
        return 1
    }
    return n * factorial(n - 1)
}

const int32 F10 = factorial(10)
```
