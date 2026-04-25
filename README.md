# MCSS / D3LTA2033

Systems programmer. Low-level implementation focus. Execution over presentation.

---

## Languages I Built

### Aether *(v1.0 — production ready)*
> Systems programming language — memory-safe, ABI-stable, C++-competitive performance

- **Memory model**: ARC — no GC pauses, no borrow checker overhead
- **ABI stability**: guaranteed binary compatibility across versions
- **Toolchain**: `build`, `test`, `bench`, `profile`, LSP, formatter, linter
- **Targets**: Linux (x86_64, AArch64) · macOS (x86_64, AArch64) · Windows (x86_64)
- **Stdlib**: `std.core` · `std.io` · `std.net` · `std.json` · `std.crypto` · `std.concurrent`
- **FFI**: direct C interop · task-based concurrency · `defer` for deterministic cleanup

```aether
fn divide(a: int, b: int) -> Result[int, string] {
    if b == 0 { return err("division by zero"); }
    return ok(a / b);
}
```

**Install**: `curl -sSf https://install.aether-lang.org | sh`

---

### Super Compiled Security Assembler (SCSA) *(v1.0.4)*
> Deterministic, security-first systems language with formal verification guarantees

Not for beginners. Not for prototypes. Built for mission-critical environments where correctness is non-negotiable.

- **Architecture**: hybrid register/stack VM · 50+ microcoded opcodes · constant-time execution
- **Memory**: linear type system · region-based allocation · hostile-by-default zeroization on every boundary
- **Cryptography**: AES-256-GCM · ChaCha20-Poly1305 · Kyber KEM · Dilithium · SPHINCS+ (post-quantum throughout)
- **Verification**: 89.3% of critical security properties verified via Coq + TLA+ model checking
- **Security**: BORE firewall · capability-based access control · taint propagation · proof-carrying bytecode
- **Toolchain**: monolithic single-binary — compiler, optimizer, verifier, VM — zero external dependencies
- **Scale**: 71 cryptographically verified modules across kernel, memory, network, security, distributed systems

```sh
scsa --compile file.scsa   # compile to hardened bytecode
scsa --run file.scsa       # compile + execute with runtime integrity verification
scsa --audit file.scsa     # formal security analysis
scsa --verify artifact.sc  # cryptographic signature check
```

**Repo**: [github.com/D3LTA2033/Super-Compiling-Security-Assemblor](https://github.com/D3LTA2033/Super-Compiling-Security-Assemblor)

---

## Modular Libraries

Production-ready modules. Clean APIs, tested, drop-in.

### [ModularX](https://github.com/D3LTA2033/ModularX)
> Cross-language modular library collection — C++, Rust, JS/TS, Python, Ruby, Assembly

Comprehensive set of production modules organized by category. Each module is self-contained with its own setup, examples, and tests.

**Categories**: caching · logging · async workers · rate limiters · event buses · state managers · memory managers · security modules

**Performance baselines**: >1M cache ops/sec · >10M log writes/sec · >5M events/sec · <1% memory overhead

```
ModularX/
├── cpp/        ├── rust/       ├── js/
├── python/     ├── ruby/       ├── assembly/
└── modularx/   └── docs/
```

**Built with**: [@mcs.s](https://discord.com/users/mcs.s)

---

### [SmartCache++](https://github.com/D3LTA2033/SmartCachePP)
> High-performance C++ caching library

```cpp
#include "SmartCache.h"
SmartCache<std::string, std::string> cache(1000);
cache.set("key", "value");
auto value = cache.get("key");
```

---

### [SmartRateLimiter](https://github.com/D3LTA2033/SmartRateLimiter)
> API rate limiting and throttling module

Pluggable rate limiting with configurable windows and burst handling. Part of the ModularX ecosystem.

---

### [universal-config-loader](https://github.com/D3LTA2033/universal-config-loader)
> Modular configuration loader for Node.js

Multiple sources and formats, seamless fallback to defaults, flexible validation. Drop-in for any Node.js project.

---

## Operating Systems

### [AstraOS](https://github.com/D3LTA2033/AstraOS)
> Modular hybrid kernel OS built from scratch — x86 (i686) and x86_64

Security-first, production-oriented design with clean separation between architecture-dependent and architecture-independent subsystems.

```
arch/x86/   kernel/   include/   user/   config/grub/   scripts/
```

---

### [StrixOS](https://github.com/D3LTA2033/StrixOS)
> Custom OS *(in development)*

---

### Contributions

| Project | Description |
|---|---|
| [emexOS](https://github.com/D3LTA2033/emexOS) | Contributed to emexOS — 160+ commits |
| [HexiumOS](https://github.com/D3LTA2033/HexiumOS) | Contributed to HexiumOS — custom OS built from scratch |

---

## Portfolio

| | |
|---|---|
| **Website** | [d3lta2033.nl](https://d3lta2033.nl) · [backup](https://portfolio-54ym.vercel.app) |
| **Kernel work** | [kernel_tut.md](https://github.com/D3LTA2033/D3LTA2033/blob/main/portfolio/kernel_tut.md) |
| **Main portfolio** | [portfolio.md](https://github.com/D3LTA2033/D3LTA2033/blob/main/portfolio/portfolio.md) |
| **Extended portfolio** | [portfolio_2.md](https://github.com/D3LTA2033/D3LTA2033/blob/main/portfolio/portfolio_2.md) |
| **AI evaluation** | [ai_opinion.md](https://github.com/D3LTA2033/D3LTA2033/blob/main/portfolio/ai_opinion.md) |

---

*I work on an older laptop. Hardware is not the bottleneck.*
*Yes i have a Desktop too.*
