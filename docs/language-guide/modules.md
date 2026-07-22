# Modules

## Declaring a module

```kestrel
module math.vector
```

## Visibility: `pub` vs. private

Declarations are **private to their module by default**. A function, struct,
enum, trait, or type alias is only visible to code in the same module unless you
mark it `pub`. Reaching for a private name from another module is a compile
error:

```kestrel
module math.internal

pub func cube(int32 n) -> int32 {   // public API — callable anywhere
    return n * square(n)
}

func square(int32 n) -> int32 {     // private — only this module may use it
    return n * n
}
```

```kestrel
module app.main

import math.internal.{cube}

func main() -> void {
    println("{cube(3)}")            // ok — cube is pub
    // println("{square(3)}")       // compile error: square is private to math.internal
}
```

This keeps a module's surface area deliberate: callers depend only on what you
chose to expose, and you can refactor everything else freely.

## Importing

```kestrel
import math.vector.Vec3
import math.vector.{Vec3, Vec4}
import math.vector.*
```

## Relative imports

```kestrel
import self.utils      // same module
import super.shared    // parent module
```

## Public re-exports

```kestrel
pub import math.vector.Vec3
```

## Example layout

```
src/
  main.kst
  math/
    mod.kst
    vector.kst
    matrix.kst
  io/
    mod.kst
    file.kst
```

`math/mod.kst`:
```kestrel
module math

pub import self.vector
pub import self.matrix
```
