# Cross-compilation

Kestrel can build a binary for a different CPU than the one you are compiling on,
in one command:

```sh
kestrel build --target aarch64 -o app-arm64 main.kst
```

That produces an ARM64 Linux executable from, say, an x86-64 laptop. No separate
steps, no hand-run `llc`.

## Why it works

The compiler emits **target-neutral LLVM IR** — it does not bake in a CPU or a
memory layout. So the same compiled program can be lowered for any supported CPU;
`--target` just tells the backend which one, and picks the matching cross
C compiler to link with.

## Supported targets

| `--target` | Architecture | Notes |
|------------|--------------|-------|
| `x86_64` | 64-bit Intel/AMD | the host default |
| `aarch64` (alias `arm64`) | 64-bit ARM | phones, Raspberry Pi 4/5, Apple-class |
| `ppc64le` | 64-bit PowerPC (little-endian) | |
| `riscv64` | 64-bit RISC-V | hard-float |
| `armv7` (alias `arm`) | 32-bit ARM (hard-float) | Raspberry Pi 2/3, many embedded boards |
| `s390x` | 64-bit IBM Z (big-endian) | |
| `i686` (alias `x86`) | 32-bit Intel/AMD | older / smaller x86 devices |
| `mipsel` | 32-bit MIPS (little-endian) | routers, embedded |
| `mips64el` | 64-bit MIPS (little-endian) | |
| `powerpc` (alias `ppc`) | 32-bit PowerPC (big-endian) | routers, older Macs, embedded |
| `mips` | 32-bit MIPS (big-endian) | routers, embedded |

You can pass either the short name (`aarch64`) or the full triple
(`aarch64-linux-gnu`).

All eleven run correct programs — every byte order and word size (32- and 64-bit,
little- and big-endian) — checked on every change, so a target on this list will
not silently break.

## What you need

`--target` shells out to the matching cross toolchain, so it must be installed:

| Target | Package (Debian/Ubuntu) |
|--------|-------------------------|
| `aarch64` | `gcc-aarch64-linux-gnu` |
| `ppc64le` | `gcc-powerpc64le-linux-gnu` |
| `riscv64` | `gcc-riscv64-linux-gnu` |
| `armv7` | `gcc-arm-linux-gnueabihf` |
| `s390x` | `gcc-s390x-linux-gnu` |
| `i686` | `gcc-i686-linux-gnu` |
| `mipsel` | `gcc-mipsel-linux-gnu` |
| `mips64el` | `gcc-mips64el-linux-gnuabi64` |
| `powerpc` | `gcc-powerpc-linux-gnu` |
| `mips` | `gcc-mips-linux-gnu` |

If the toolchain is missing, the compiler says exactly which package to install.

## Running the result

A cross-built binary runs on the real target device. To try it on your build
machine, use an emulator:

```sh
kestrel build --target aarch64 -o app-arm64 main.kst
qemu-aarch64-static -L /usr/aarch64-linux-gnu ./app-arm64
```

## Notes

- `--target` is a **build-only** option — `run`, `check`, and `emit-*` ignore it
  (you cannot execute a foreign-CPU binary directly on the host).
- Every cross build is Linux/ELF and keeps the same security hardening as a native
  build (PIE, full RELRO, non-executable stack, stack canaries).
- The output is a normal ELF executable, indistinguishable in shape from a
  C program built with GCC for that CPU.
