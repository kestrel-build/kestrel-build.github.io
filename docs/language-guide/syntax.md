# Statements & Line Breaks

Kestrel uses **newline as the statement terminator** — like Go, and unlike C.
There are no semicolons. Write one statement per line:

```kestrel
int32 x = 42
int32 y = x + 1
println("{y}")
```

Blank lines are ignored, so use them freely to group code. Indentation is not
significant (blocks are delimited by braces), but 4-space indentation is the
convention.

## Breaking a long line

A newline ends a statement *only* when the line so far could be a complete
statement. In the two situations below the newline is treated as a space and the
statement continues — the same rule Go uses. There is no line-continuation
character (a `\` is not special here).

### 1. After an operator that needs a right-hand side

A line that ends on an operator which cannot finish a statement — a binary,
comparison, logical, bitwise, or assignment operator, `??`, a trailing `.`, or
`->` — continues onto the next line:

```kestrel
int32 total = first_value +
              second_value +
              third_value

if (ready &&
    count > 0) {
    // ...
}
```

Method-call chains break the same way, on a **trailing** `.`:

```kestrel
str result = name.trim().
    to_upper()
```

(The `.` must end the upper line. A line that ends on a complete value — e.g.
`name.trim()` — terminates, so a `.method()` cannot *start* the next line.)

### 2. Inside brackets `(` … `)` and `[` … `]`

Newlines between a matching `(`/`)` or `[`/`]` are ignored, so call arguments,
function parameters, array/list literals, and parenthesized expressions may span
lines:

```kestrel
int32 s = add(
    add(1, 2),
    add(3, 4))

func distance(float64 x,
              float64 y) -> float64 {
    return x * x + y * y
}

int32[3] triple = [10,
                   20,
                   30]
```

!!! note "No trailing comma"

    Close the list right after the last element (`3)` above, not `3,)`). A
    trailing comma before the closing bracket is a syntax error.

## String interpolation

Inside an interpolated string, `"{ ... }"` holds an expression that is evaluated
and formatted in place. The whole string literal stays on a single source line:

```kestrel
str name = "Kestrel"
println("Hello, {name}! 1 + 1 = {1 + 1}")
```

## Summary

| You want to… | Do this |
|---|---|
| End a statement | Start a new line (no `;`) |
| Continue a long expression | End the line on an operator (`+`, `&&`, `=`, a trailing `.`, …) — it carries to the next line |
| Spread call args / parameters / a list literal | Break after a comma inside `(…)` or `[…]` — no trailing comma |
| Group code visually | Use blank lines (ignored) |
