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

Run cleanup code at scope exit:

```kestrel
func process() -> void {
    ptr[void] buf = malloc(1024)
    defer free(buf)

    // ... use buf ...
    // free(buf) runs here automatically
}
```

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
