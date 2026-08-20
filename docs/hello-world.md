# Hello World

The simplest Kestrel program:

```kestrel
func main() -> int32 {
    println("Hello, world!")
    return 0
}
```

Save this as `hello.kst` and run it:

```bash
kestrel run hello.kst
```

## What each line means

- `func main() -> int32 {` — the entry point, returns an integer exit code
- `println(...)` — prints a line to standard output (a built-in; no import needed).
  It appends the newline for you, and `{...}` interpolates values into the string
- `return 0` — exit code 0 means success
- Curly braces define blocks

## String interpolation

```kestrel
func main() -> int32 {
    str name = "Kestrel"
    println("Hello, {name}!")
    return 0
}
```

## Next steps

See the [Language Guide](language-guide/overview.md) for variables, types, and control flow.
