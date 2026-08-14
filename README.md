# mcss / D3LTA2033

Systems programmer. Low-level implementation focus. Execution over presentation.

**C · C++ · Rust · JS/TS · Python · x86 asm**
[d3lta2033.nl](https://d3lta2033.nl) · Discord [`@mcs.s`](https://discord.com/users/mcs.s)

---

## Languages I Built

### [Aether](https://github.com/D3LTA2033/aether) *(v1.0)*
> Systems programming language — ARC memory management, ABI stability by design, performance targets in C++ territory

- **~19k lines of C++** — full pipeline in one repo: lexer → parser → HIR → semantic analysis → optimizer → codegen / VM
- **Toolchain in-tree**: `build` · `test` · `bench` · `profile` · LSP · formatter · linter · package registry
- **Memory model**: ARC — no GC pauses, no borrow checker overhead
- **Stdlib**: `std.core` · `std.io` · `std.net` · `std.json` · `std.crypto` · `std.concurrent`
- **FFI**: direct C interop · task-based concurrency · `defer` for deterministic cleanup
- **Targets**: Linux · macOS · Windows

```aether
fn divide(a: int, b: int) -> Result[int, string] {
    if b == 0 { return err("division by zero"); }
    return ok(a / b);
}
```

**Install**: `curl -sSf https://install.aether-lang.org | sh`

---

### [SCSA — Super Compiled Security Assembler](https://github.com/D3LTA2033/Super-Compiling-Security-Assemblor) *(v1.0.4)*
> Security-oriented assembler + VM in a single C file. One binary, no dependencies beyond OpenSSL.

- **~1.2k-line C core**, ported per platform: Linux · macOS · Windows
- **56-instruction hybrid register/stack ISA** — arithmetic, control flow, structs, enums, `try`/`catch`, threads, GC
- **Built-in primitives**: file I/O · networking (`NET_BIND` / `LISTEN` / `CONNECT` / `SEND` / `RECV`) · AES-256 `ENCRYPT` / `DECRYPT` via OpenSSL
- **70+ example modules** in-repo
- **Ecosystem**: [`scsa_compiler`](https://github.com/D3LTA2033/scsa_compiler) · [`scsa-vscode`](https://github.com/D3LTA2033/scsa-vscode)

```sh
scsa --compile file.scsa   # compile to bytecode
scsa --run file.scsa       # compile + execute
scsa --modular file.scsa   # run with module system
```

---

## Modular Libraries

### [ModularX](https://github.com/D3LTA2033/ModularX)
> Cross-language module collection — C++, Rust, JS/TS, Python, Ruby, Assembly

Self-contained modules, each with its own setup, examples, and tests.

**C++**: SmartCachePP · SmartLoggerPP · AsyncWorkerPP
**JS/TS**: eventbusx · smart-ratelimiterx · statemanagerx · ucl
**Plus** Python, Rust, Ruby, and Assembly modules

### Standalone

| Repo | What it is |
|---|---|
| [SmartCachePP](https://github.com/D3LTA2033/SmartCachePP) | C++ caching library — drop-in header + impl |
| [SmartRateLimiter](https://github.com/D3LTA2033/SmartRateLimiter) | Node.js rate limiting — configurable windows, burst handling |
| [universal-config-loader](https://github.com/D3LTA2033/universal-config-loader) | Node.js config loader — multi-source, fallback defaults, validation |

```cpp
#include "SmartCache.h"
SmartCache<std::string, std::string> cache(1000);
cache.set("key", "value");
auto value = cache.get("key");
```

---

## Operating Systems

### [AstraOS](https://github.com/D3LTA2033/AstraOS)
> Modular hybrid-kernel OS built from scratch — x86

**~13k lines of C and assembly.** Clean separation between architecture-dependent (`arch/x86/` — boot, linker script) and architecture-independent subsystems (`kernel/`, `user/`). GRUB config and build scripts in-tree.

### Forks & Experiments

| Repo | |
|---|---|
| [emexOS](https://github.com/D3LTA2033/emexOS) | Fork of [Voxi0/emexOS1](https://github.com/Voxi0/emexOS1) — my working branch |
| [HexiumOS](https://github.com/D3LTA2033/HexiumOS) | Fork of [Abdallah-Alwarawreh/HexiumOS](https://github.com/Abdallah-Alwarawreh/HexiumOS) — Rust OS from scratch |

---

## Also Building

[**Venoly**](https://venoly.nl) — communication platform: chat, communities, voice/video. ([venoly-desktops](https://github.com/D3LTA2033/venoly-desktops))

More on the profile: [Neoarc-engine](https://github.com/D3LTA2033/Neoarc-engine) · [Security-Recovery-Core](https://github.com/D3LTA2033/Security-Recovery-Core-SRC) · [secureforge](https://github.com/D3LTA2033/secureforge) · [+20 more](https://github.com/D3LTA2033?tab=repositories)

---

## Portfolio

| | |
|---|---|
| **Website** | [d3lta2033.nl](https://d3lta2033.nl) · [backup](https://portfolio-54ym.vercel.app) |
| **Kernel notes** | [kernel_tut.md](https://github.com/D3LTA2033/D3LTA2033/blob/main/portfolio/kernel_tut.md) |
| **Portfolio** | [portfolio.md](https://github.com/D3LTA2033/D3LTA2033/blob/main/portfolio/portfolio.md) · [portfolio_2.md](https://github.com/D3LTA2033/D3LTA2033/blob/main/portfolio/portfolio_2.md) |

---

*Hardware is not the bottleneck.*
