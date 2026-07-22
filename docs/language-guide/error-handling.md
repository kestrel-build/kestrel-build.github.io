# Error Handling

## Try / catch / finally

```kestrel
try {
    str content = read_file("data.txt")
    process(content)
} catch (IOError e) {
    printf("File error: {e.message}\n")
} catch (Error e) {
    printf("Unexpected error: {e.message}\n")
} finally {
    cleanup()
}
```

The `finally` block always runs, whether or not an error occurred.

## With (resource cleanup)

The `with` block closes resources automatically, even if an error occurs:

```kestrel
with (File f = open("log.txt")) {
    f.write("entry\n")
}
// f is closed here
```

## Defer

`defer` schedules an expression to run when its enclosing block exits — by any
path: falling off the end, an early `return`, a `break`/`continue`, or a `throw`.
Deferred expressions run **last-in-first-out**, and **before** the block's locals
are freed, so cleanup can still use them.

```kestrel
func process() -> void {
    File f = open("data.txt")
    defer f.close()          // runs no matter how process() exits

    str line = f.read_line()
    // ... f.close() runs here automatically ...
}
```

`defer` is **block-scoped**: a `defer` inside a loop body fires at the end of
each iteration, not all at once when the function returns.

```kestrel
for (i in 0..2) {
    defer println("close {i}")
    println("open {i}")
}
// open 0 / close 0 / open 1 / close 1 / open 2 / close 2
```

## Contracts

`@requires` and `@ensures` attach a precondition and a postcondition to a
function. Both are boolean expressions checked at run time; `@ensures` also sees
the returned value as `result`. A violated contract reports which one failed and
stops the program — a broken assumption halts at the boundary that stated it,
instead of corrupting whatever comes next.

```kestrel
@requires(n >= 0)
@ensures(result >= 1)
func factorial(int32 n) -> int32 {
    if (n <= 1) {
        return 1
    }
    return n * factorial(n - 1)
}
```

Several `@requires` may be stacked; each is checked on entry.

## Nested error handling

```kestrel
func load_and_process(str path) -> void {
    try {
        str data = read_file(path)
        try {
            parse(data)
        } catch (ParseError e) {
            printf("Bad format: {e.message}\n")
        }
    } catch (IOError e) {
        printf("Cannot read {path}: {e.message}\n")
    }
}
```
