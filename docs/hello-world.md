# Hello World

The simplest Kestrel program:

```kestrel
func main() -> int32 {
    printf("Hello, world!\n")
    return 0
}
```

Save this as `hello.kst` and run it:

```bash
kestrel run hello.kst
```

## What each line means

- `func main() -> int32 {` — the entry point, returns an integer exit code
- `printf(...)` — calls the C standard library printf; available without an import
- `return 0` — exit code 0 means success
- Curly braces define blocks

## String interpolation

```kestrel
func main() -> int32 {
    str name = "Kestrel"
    printf("Hello, {name}!\n")
    return 0
}
```

## Next steps

See the [Language Guide](language-guide/overview.md) for variables, types, and control flow.
