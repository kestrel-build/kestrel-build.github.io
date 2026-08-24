# Getting Started

## Installation

### One-line installer (recommended)

This downloads and installs the **latest** release for your platform:

```bash
curl -fsSL https://raw.githubusercontent.com/kestrel-build/kestrel/main/install.sh | sh
```

To install a specific version instead of the latest, set `KESTREL_VERSION`:

```bash
KESTREL_VERSION=v1.0.0-beta.9 curl -fsSL https://raw.githubusercontent.com/kestrel-build/kestrel/main/install.sh | sh
```

### Manual download

Pre-built Linux binaries are on the [GitHub Releases](https://github.com/kestrel-build/kestrel/releases/latest) page for six architectures: **x86_64, aarch64, ppc64le, riscv64, armv7, and s390x**. Download the one for your platform, extract it, and put `kestrel` on your PATH:

```bash
tar xzf kestrel-<version>-linux-x86_64.tar.gz   # e.g. kestrel-v1.0.0-beta.9-linux-x86_64
mv kestrel ~/.local/bin/
export PATH="$HOME/.local/bin:$PATH"            # if not already on PATH
```

Already have Kestrel for one CPU? You can also **cross-compile** for any of the
others in one command — see [Cross-compilation](reference/cross-compilation.md).
(A macOS build exists in source but is not yet a released target.)

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
    println("Hello, world!")
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
