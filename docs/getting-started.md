# Getting Started

## Installation

### One-line installer (recommended)

This downloads and installs the **latest** release for your platform:

```bash
curl -fsSL https://raw.githubusercontent.com/kestrel-build/kestrel-public/main/install.sh | sh
```

To install a specific version instead of the latest, set `KESTREL_VERSION`:

```bash
KESTREL_VERSION=v1.0.0-alpha.6 curl -fsSL https://raw.githubusercontent.com/kestrel-build/kestrel-public/main/install.sh | sh
```

### Manual download

Pre-built binaries for Linux and macOS are on the [GitHub Releases](https://github.com/kestrel-build/kestrel/releases/latest) page. Download the archive for your platform from the latest release, extract it, and put `kestrel` on your PATH:

```bash
tar xzf kestrel-<version>-linux-x86_64.tar.gz   # e.g. kestrel-v1.0.0-alpha.6-linux-x86_64
mv kestrel ~/.local/bin/
export PATH="$HOME/.local/bin:$PATH"            # if not already on PATH
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
