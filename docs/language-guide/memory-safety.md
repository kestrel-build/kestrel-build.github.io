# Memory Safety

Kestrel uses **NOVA** (No Overhead Value Allocation) to manage heap memory automatically — without a garbage collector and without manual `free` calls.

## How it works

When a value goes out of scope, the compiler inserts a `free` call automatically. There is no runtime overhead beyond the allocation itself.

```kestrel
func process() -> void {
    str data = read_file("input.txt")   // heap allocation
    printf("{data}\n")
    // data is freed here automatically — end of scope
}
```

## Ownership

Each value has one owner. When you assign a value to a new variable, ownership transfers (a move):

```kestrel
str a = "hello"
str b = a       // 'a' is moved into 'b'
// printf(a)    // compile error — 'a' was moved
printf(b)       // ok
```

## Borrow checker

The borrow checker prevents use-after-move at compile time, with no runtime checks needed.

```kestrel
str message = "hello"
str other = message      // move
printf(message)          // error: value was moved
```

## Raw pointers

For C interoperability, raw pointers are available — but only in `unsafe` blocks:

```kestrel
extern "C" func malloc(uint64 size) -> ptr[void]

unsafe {
    ptr[int32] p = cast[ptr[int32]](malloc(4))
    *p = 42
    int32 val = *p
}
```

## No null pointer dereferences

Nullable pointers use `ptr[T]?` syntax and must be checked before use:

```kestrel
ptr[int32]? maybe = find_value()
if (maybe != null) {
    unsafe {
        int32 val = *maybe
    }
}
```
