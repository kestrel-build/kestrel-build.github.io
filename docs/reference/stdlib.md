# Standard Library Reference

The Kestrel standard library is in active development. See the [std repository](https://github.com/kestrel-build/std) for current status.

## Modules

### `io`

File and console I/O.

```kestrel
import io.file

str content = io.file.read("data.txt")
io.file.write("output.txt", content)
```

### `collections`

Common data structures.

```kestrel
import collections.list
import collections.map
import collections.set
```

### `math`

Mathematical functions and constants.

```kestrel
import math

float64 result = math.sqrt(2.0)
float64 pi = math.PI
```

### `string`

String manipulation beyond the built-in methods.

```kestrel
import string

str trimmed = string.trim("  hello  ")
str upper = string.to_upper("hello")
str[] parts = string.split("a,b,c", ",")
```

## Built-in string methods

These are available on any `str` value without an import:

```kestrel
str s = "Hello, World!"
int32 n = s.len()
bool b = s.contains("World")
bool starts = s.starts_with("Hello")
bool ends = s.ends_with("!")
str t = s.trim()
str u = s.to_upper()
str l = s.to_lower()
str r = s.replace("World", "Kestrel")
```
