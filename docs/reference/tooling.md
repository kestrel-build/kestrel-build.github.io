# Tooling

Kestrel ships as a single binary that is also your formatter, linter, doc
generator, and test runner — no extra tools to install.

## `kestrel fmt`

Format source to the canonical style: 4-space indentation by brace depth,
trimmed trailing whitespace, collapsed blank runs, and a single trailing
newline. Comments and string contents are preserved exactly.

```bash
kestrel fmt src/main.kst          # reformat in place
kestrel fmt --check src/*.kst     # report unformatted files, exit 1 (for CI)
```

Formatting is idempotent — running it on already-formatted code changes nothing.

## `kestrel lint`

Report style and correctness lints over the parsed source.

```bash
kestrel lint src/*.kst
```

The current checks cover naming conventions (snake_case for functions and
parameters, PascalCase for types and enum variants) and flag empty function
bodies. Exit code is 1 if any lints are found.

## `kestrel doc`

Generate a Markdown API reference for a module's **public** surface — its `pub`
functions, structs, enums, traits, and type aliases, with full signatures and
the prose from their `///` doc comments.

```bash
kestrel doc src/vector.kst              # print to stdout
kestrel doc src/vector.kst -o vector.md # write a file
```

```kestrel
/// Computes the area of a circle given its radius.
pub func circle_area(float64 radius) -> float64 {
    return 3.14159 * radius * radius
}
```

renders the signature followed by *"Computes the area of a circle given its
radius."*

## `kestrel test`

Discover the `@test` functions in a file, run them, and report pass/fail.

```kestrel
// math_test.kst
func add(int32 a, int32 b) -> int32 { return a + b }

@test
func test_add() -> void {
    if (add(2, 3) != 5) {
        fail("add(2, 3) should be 5")
    }
}
```

```bash
kestrel test math_test.kst
# running 1 test(s) from math_test.kst
# test test_add ...
# ok — 1 test(s) passed
```

A test signals failure by aborting (for example via `fail`); the runner reports
a non-zero exit when any test fails.
