---
date: 2026-07-06
authors:
  - kestrel
categories:
  - Release
---

# Kestrel v1.0.0-alpha.6 Released

Alpha.6 is the "toolbox" release. It adds the data structures and text-handling
a real program -- and eventually the Kestrel compiler itself -- needs: hash
maps, lists of structs, string processing, character literals, and exceptions.
Every heap-touching feature was checked leak-free under Valgrind.

<!-- more -->

## Hash maps

A string-keyed hash map, the thing every compiler's symbol table is built on:

```kestrel
func main() -> void {
    HashMap[str, int32] ages = map_new()
    ages.insert("alice", 30)
    ages.insert("bob", 25)
    if (ages.contains("alice")) {
        int32 a = ages.get("alice")
        println("alice is {a}")
    }
    println("entries: {ages.len()}")
}
```

The map owns its keys and (for string values) its values -- it clones them in
and frees them when it goes out of scope. No manual cleanup, no leaks.

## Lists of structs

Lists already held numbers and strings. Now they hold **structs** too --
including structs that own strings, like a list of tokens:

```kestrel
struct Token {
    str text
    int32 kind
}

func main() -> void {
    List[Token] toks = list_new()
    toks.push(Token { text: "if", kind: 1 })
    println("first: {toks.get(0).text}")
}
```

Pushing a variable deep-copies it (your original stays valid), and every element
is freed when the list is dropped.

## Reading text

Strings gained the operations a lexer lives on:

```kestrel
str s = "abc123"
println("length: {s.len()}")
int32 first = s.byte_at(0)          // 97
str head = s.substring(0, 3)        // "abc"
```

## Character literals

And characters, which are just their `int32` code points -- so they compare
directly with `byte_at`:

```kestrel
func is_digit(str s, int64 i) -> bool {
    int32 c = s.byte_at(i)
    return c >= '0' && c <= '9'
}
```

`'a'`, `'\n'`, `'\t'`, `'\0'` all work.

## Exceptions

And a real error-handling path -- `try` / `catch` / `finally` with `throw`:

```kestrel
func checked_div(int32 a, int32 b) -> int32 {
    if (b == 0) {
        throw("division by zero")
    }
    return a / b
}

func main() -> void {
    try {
        println("{checked_div(10, 2)}")
        println("{checked_div(1, 0)}")
    } catch (Error e) {
        println("caught: {e}")
    } finally {
        println("done")
    }
}
```

`throw` carries a message; the nearest `try` catches it and binds it to the
catch variable; `finally` always runs.

## Why this release matters

Each of these is a piece the Kestrel compiler will need when it's rewritten in
Kestrel: hash maps for interners, lists of tokens and AST nodes, byte-level
string scanning for the lexer. Alpha.6 is the language growing up enough to
describe itself. Generics and traits are next.

## Try it out

- [GitHub Releases](https://github.com/kestrel-build/kestrel/releases)
- [Examples repository](https://github.com/kestrel-build/examples)
- [Documentation](https://kestrel-build.github.io)
