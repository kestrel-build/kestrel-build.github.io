# Control Flow

## If / else

Parentheses around the condition are required:

```kestrel
if (x > 0) {
    println("positive")
} else if (x < 0) {
    println("negative")
} else {
    println("zero")
}
```

## Ternary

```kestrel
int32 abs_x = if (x >= 0) x else -x
```

## For loops

```kestrel
// inclusive range (0 through 9)
for (i in 0..9) {
    println("{i}")
}

// exclusive range (0 through 9, not 10)
for (i in 0..<10) {
    println("{i}")
}
```

## While

```kestrel
while (counter < 100) {
    counter = counter + 1
}
```

## Do-while

```kestrel
do {
    counter = counter + 1
} while (counter < 100)
```

## Repeat

```kestrel
repeat (5) {
    println("hello")
}
```

## Match

A `case` uses the colon form: the statements after the `:` run until the next
`case` or the closing `}`. Patterns can be literals, inclusive (`..`) or
exclusive (`..<`) ranges, or `_` for the catch-all.

```kestrel
match (status) {
    case 200: println("OK")
    case 404: println("Not found")
    case 500..599: println("Server error")
    case _: println("Unknown: {status}")
}
```

A case can run several statements — they all belong to the case until the next
`case`:

```kestrel
match (status) {
    case 200:
        log("serving")
        println("OK")
    case _:
        println("Unknown: {status}")
}
```

Enum variants match by their qualified name, and payload fields bind as names:

```kestrel
match (shape) {
    case Shape.Circle(r): return 3.14159 * r * r
    case Shape.Rect(w, h): return w * h
    case Shape.Empty: return 0.0
}
```

## Break and continue

```kestrel
for (i in 0..<100) {
    if (i == 50) {
        break
    }
    if (i % 2 == 0) {
        continue
    }
    println("{i}")
}
```
