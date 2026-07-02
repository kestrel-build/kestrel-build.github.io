# Modules

## Declaring a module

```kestrel
module math.vector
```

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
