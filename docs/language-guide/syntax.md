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

Because a newline ends a statement, a statement that should continue onto the
next line needs one of the two mechanisms below.

### 1. Inside a bracketed, comma-separated list (automatic)

Newlines are ignored between the brackets of a **comma-separated list**, so you
can spread these across lines — and a trailing comma is allowed:

```kestrel
// Function call arguments
int32 total = sum(
    first_value,
    second_value,
    third_value,
)

// List / array literals
int32[3] triple = [
    10,
    20,
    30,
]

// Import selector braces
import std.collections.{
    HashMap,
    HashSet,
}
```

Nested calls may span lines the same way:

```kestrel
int32 s = add(
    add(1, 2),
    add(3, 4),
)
```

### 2. Backslash continuation (explicit, works anywhere)

To break a line *anywhere else*, end it with a backslash `\`. The backslash and
the following newline are removed, joining the two lines into one logical line:

```kestrel
int32 x = 1 + \
          2 + \
          3

str r = "hello".to_upper(). \
        to_lower()
```

## Where automatic continuation does **not** apply

The automatic rule only covers commas inside brackets. In the places below a
newline still ends the statement, so you must keep the code on one line or use a
`\`.

!!! warning "These break across lines only with a trailing `\`"

    **After an operator** — a line ending in a binary or logical operator is an
    error, *even inside parentheses*:

    ```kestrel
    // error: the newline after `&&` ends the expression
    if (a > 0 &&
        a < 10) { }

    // ok — trailing backslash, or keep it on one line
    if (a > 0 && \
        a < 10) { }
    ```

    **Method-call chains** — a `.method()` cannot start (or the previous line
    end with a `.`) on its own line without a `\`:

    ```kestrel
    // error
    str r = name.trim()
                .to_upper()

    // ok
    str r = name.trim(). \
                to_upper()
    ```

    **Function declaration parameter lists** — keep the parameters of a `func`
    declaration on one line:

    ```kestrel
    // error: declaration parameters do not span lines
    func add(
        int32 a,
        int32 b) -> int32 { return a + b }

    // ok
    func add(int32 a, int32 b) -> int32 { return a + b }
    ```

## String interpolation

Inside an interpolated string, `"{ ... }"` holds an expression that is evaluated
and formatted in place. The whole string literal — braces included — stays on a
single source line:

```kestrel
str name = "Kestrel"
println("Hello, {name}! 1 + 1 = {1 + 1}")
```

## Summary

| You want to… | Do this |
|---|---|
| End a statement | Start a new line (no `;`) |
| Break a call's arguments / a list literal / import braces | Break after any comma; a trailing comma is fine |
| Break after an operator, split a method chain, or continue anything else | End the line with `\` |
| Group code visually | Use blank lines (ignored) |
