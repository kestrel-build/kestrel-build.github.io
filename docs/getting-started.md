# Getting Started

## Installation

### Download from GitHub Releases

Pre-built binaries for Linux and macOS are available on the [GitHub Releases](https://github.com/kestrel-build/kestrel/releases) page.

Download the archive for your platform, extract it, and place the `kestrel` binary somewhere on your PATH:

```bash
tar xzf kestrel-v1.0.0-alpha.1-linux-x86_64.tar.gz
mv kestrel ~/.local/bin/
```

Make sure `~/.local/bin` is in your PATH:

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
