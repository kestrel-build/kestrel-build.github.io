# Security Overview

Kestrel treats security as a type system property, not a set of runtime checks. Eight security dimensions are built into the language.

## The 8 dimensions

### 1. NOVA memory safety

No dangling pointers. No use-after-free. No buffer overflows. Automatic deallocation at scope end with no GC overhead.

### 2. Taint tracking

User input is `@tainted` at the source. The compiler refuses to let tainted data reach a `@sink` (database query, shell command, HTML output) without going through a `@sanitizer`.

```kestrel
@tainted str user_input = request.get("name")
// db.query(user_input)         // compile error
str safe = html_escape(user_input)
response.write(safe)            // ok
```

### 3. Effect types

Functions declare their side effects. A function marked `@pure` cannot do I/O. A function marked `@io` cannot make network calls unless also marked `@network`. This prevents supply-chain attacks where a math library sneaks in a network call.

### 4. Constant-time types

`secret[T]` values cannot leak through timing side-channels. The compiler rejects branches on secret values and ensures constant-time comparison.

### 5. Contracts and refinements

`@requires` and `@ensures` let you attach preconditions and postconditions to functions, checked at compile time where possible.

### 6. Capability permissions

`cap[resource]` tokens control access to system resources. Code must be explicitly granted a capability to use it — it cannot be acquired out of thin air.

### 7. Crypto safety

`@sensitive` memory is wiped before deallocation and protected from compiler optimization that might eliminate the wipe.

### 8. Debug toolkit

`debug_assert`, `debug:` blocks, and `assert_main_thread` for catching threading bugs in development builds.

## Status

- **NOVA memory safety** — implemented
- **Taint tracking, effect types, constant-time, contracts, capabilities, crypto safety, debug toolkit** — designed, not yet implemented

See [Taint Tracking](taint-tracking.md) and [Effect System](effect-system.md) for the design details.
