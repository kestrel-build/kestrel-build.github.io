# Taint Tracking

Taint tracking is a compile-time data flow analysis that prevents injection attacks. It is part of the type system — not a linter, not a runtime check.

## How it works

Data from external sources is marked `@tainted`. That taint propagates through the program. Any value derived from tainted data is also tainted. A `@sink` — a function that could cause harm if it receives untrusted input — refuses tainted data at compile time. A `@sanitizer` removes the taint by validating or escaping the data.

```kestrel
// Source: tainted input from the web
@tainted str name = request.get("name")

// Propagation: anything derived from 'name' is also tainted
@tainted str greeting = "Hello, " + name

// Sink: the database refuses tainted data
// db.query("SELECT * FROM users WHERE name = '{name}'")  // compile error

// Sanitizer: validate and escape, taint is removed
str safe_name = sql_escape(name)

// Now it can reach the sink
db.query("SELECT * FROM users WHERE name = '{safe_name}'")
```

## Declaring sources, sinks, and sanitizers

```kestrel
// A source — returns tainted data
@source func read_http_param(str key) -> @tainted str {
    // ...
}

// A sanitizer — accepts tainted, returns clean
@sanitizer func sql_escape(@tainted str input) -> str {
    // ...
}

// A sink — refuses tainted data (compiler enforces this)
@sink func db_query(str sql) -> void {
    // ...
}
```

## What injection attacks this prevents

- SQL injection
- Command injection
- HTML/XSS injection
- Path traversal
- LDAP injection
- Any custom sink you define
