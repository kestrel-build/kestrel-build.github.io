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

## Debugging the compiler front end

When you need to see *why* a program parses the way it does, two subcommands
expose the front end directly:

```bash
kestrel dump-tokens src/main.kst   # the lexer's token stream: line:col KIND "text"
kestrel dump-ast    src/main.kst   # the parsed AST as an indented tree
```

`dump-ast` prints each statement and its children (assignments show their target
name), so you can see exactly which statements landed in which block — the fastest
way to tell a lexer problem from a parser one.

## Verifying generated code

```bash
kestrel build --verify-ir src/main.kst   # structural IR checks before codegen
```

`--verify-ir` runs a well-formedness pass over the compiler's internal IR — every
block has a terminator, every branch targets a real block, no block is
unreachable — and fails the build if anything is malformed, before it can become a
runaway loop or a link error. It is cheap and safe to leave on.

## `kestrel explain`

Print the long-form explanation for an error code:

```bash
kestrel explain E0300
```

