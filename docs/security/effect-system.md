# Effect System

The effect system lets functions declare what they do — I/O, network access, mutation, randomness. The compiler enforces those declarations.

## Effect annotations

```kestrel
@pure func add(int32 a, int32 b) -> int32 {
    return a + b
}

@io func read_file(str path) -> str {
    // ...
}

@network @io func fetch_url(str url) -> str {
    // ...
}
```

## Why this matters

A function marked `@pure` cannot do I/O. You can trust it completely. This matters for:

- **Testing** — pure functions are trivially testable, no mocking needed
- **Security** — a math library cannot sneak in a network call
- **Parallelism** — pure functions can safely run in parallel
- **Caching** — pure functions can be memoized

## Inference

Effects are inferred where possible. A function that only calls `@pure` functions is automatically `@pure`. You only need to annotate when you want to be explicit or when the inference would be wrong.

## Effect propagation

Calling a function with an effect requires the caller to also declare that effect:

```kestrel
// This would be a compile error — read_file is @io but process is not declared @io
func process(str path) -> str {
    return read_file(path)   // error: @io effect not declared
}

// Fix: declare the effect
@io func process(str path) -> str {
    return read_file(path)   // ok
}
```
