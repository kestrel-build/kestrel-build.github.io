# Getting Started

## Installation

### Linux and macOS

```bash
curl -fsSL https://kestrel-build.github.io/install.sh | sh
```

This installs the `kestrel` binary to `~/.local/bin/`. Add that to your PATH if it isn't already:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

### From source

You need Rust, LLVM, and a C compiler.

```bash
git clone https://github.com/kestrel-build/kestrel
cd kestrel
cargo build --release
```

## Verify the install

```bash
kestrel --version
```

## Hello World

Create a file called `hello.kst`:

```kestrel
func main() -> int32 {
    printf("Hello, world!\n")
    return 0
}
```

Build and run:

```bash
kestrel run hello.kst
```

Or build to a binary:

```bash
kestrel build hello.kst
./hello
```

## Next steps

- [Language guide](language-guide/overview.md) — variables, types, control flow
- [Memory safety](language-guide/memory-safety.md) — how NOVA deallocation works
- [Security model](security/overview.md) — taint tracking and effect types
- [Examples](https://github.com/kestrel-build/examples) — working programs to learn from
