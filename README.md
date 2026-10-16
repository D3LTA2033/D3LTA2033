<div align="center">

<img width="100%" alt="header" src="https://capsule-render.vercel.app/api?type=waving&height=220&color=0:1a1b27,50:3d59a1,100:bb9af7&text=mcss&fontColor=ffffff&fontSize=74&fontAlignY=36&animation=fadeIn&desc=kernels%20%C2%B7%20compilers%20%C2%B7%20execution%20control&descAlignY=60&descSize=18">

<a href="https://github.com/D3LTA2033">
  <img alt="typing" src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=600&size=19&duration=2800&pause=1100&color=7AA2F7&center=true&vCenter=true&repeat=true&width=720&height=44&lines=Kernels%2C+compilers%2C+and+the+layer+that+decides+what+runs;61%2C144+hand-written+lines+across+11+repos;i686+kernel+with+no+libc%2C+ring+0+to+ring+3;exec+firewall+in+Rust+on+top+of+fanotify;hardening+wizard+that+writes+its+own+uninstaller;Shipping+Venoly%2C+a+security-first+chat+platform;Open+to+low-level+and+security+work">
</a>

<br>

<a href="https://d3lta2033.nl"><img alt="site" src="https://img.shields.io/badge/site-d3lta2033.nl-313131?style=flat&labelColor=545454"></a>
<a href="https://venoly.nl"><img alt="venoly" src="https://img.shields.io/badge/product-venoly.nl-313131?style=flat&labelColor=545454"></a>
<img alt="discord" src="https://img.shields.io/badge/discord-%40mcs.s-313131?style=flat&labelColor=545454">
<img alt="lines" src="https://img.shields.io/badge/measured-61%2C144_lines-313131?style=flat&labelColor=545454">
<img alt="c-cpp" src="https://img.shields.io/badge/C%20%2B%20C%2B%2B-60.9%25_of_bytes-313131?style=flat&labelColor=545454">
<a href="https://github.com/D3LTA2033?tab=followers"><img alt="followers" src="https://img.shields.io/github/followers/D3LTA2033?style=flat&labelColor=545454&color=313131"></a>
<img alt="api status" src="https://img.shields.io/website?url=https%3A%2F%2Fapi.venoly.nl%2Fhealth&label=api.venoly.nl&up_message=online&down_message=offline&style=flat&labelColor=545454&up_color=2ea043">
<img alt="last commit" src="https://img.shields.io/github/last-commit/D3LTA2033/D3LTA2033?style=flat&labelColor=545454&color=313131">

<br><br>

<img alt="stack" src="https://skillicons.dev/icons?i=c,cpp,rust,go,py,js,bash,linux,arch,git,nodejs,react&perline=12">

<br><br>

<img src="assets/terminal.svg" width="720" alt="terminal demo">

</div>

<br>

I write low-level software: an i686 kernel with no libc, a systems language and its
compiler, an execution firewall that gates `execve` from userspace, and a hardening
wizard that generates bash you can take back out again. Mostly C, C++ and Rust.

Line counts come from `wc -l` on a clean clone; language bytes are GitHub's Linguist
figures; the Venoly sizes were measured over the wire.

<div align="center">

| [Numbers](#by-the-numbers) | [Measured](#measured) | [Languages](#languages-by-measured-bytes) | [Map](#how-it-fits-together) | [Detail](#detail) | [Repos](#also-in-the-tree) | [Pulse](#repo-pulse) | [Flags](#build-flags-and-scorecard) | [Limits](#read-before-you-run) | [Shipping](#shipping) | [Verify](#verify-it-yourself) |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|

</div>

<img src="assets/divider.svg" width="100%" alt="">

## At a glance

<div align="center">

| | |
|:--|:--|
| **Kernel** | AstraOS — i686, no libc, ring 0 → ring 3, own GUI, own language |
| **Compilers** | Aether (C++23, emits C), hvc (C → IR + register VM), SCSA (one file, tokenizer → VM) |
| **Defence** | RESS — fanotify execution firewall, hand-rolled sandbox · secureforge — reversible hardening, 9 distros |
| **Firmware** | Security-Recovery — 8-state recovery machine, A/B rotation, 16 MB SPI layout |
| **Product** | Venoly — chat, voice/video, communities · desktop builds with checksums |
| **Also** | a 302.7M-parameter char-level GPT from scratch, a local AI launcher in Go, a 22-pass Arch maintenance script |

</div>

<img src="assets/divider.svg" width="100%" alt="">

## By the numbers

Small figures that are easy to check against the source.

<div align="center">

| | | | |
|:-:|:-:|:-:|:-:|
| **16**<br>AstraOS syscalls | **48**<br>AstraOS ISR stubs | **18**<br>SCSA opcodes | **12**<br>RESS policy predicates |
| **10**<br>RESS crates | **5**<br>RESS sandbox namespaces | **9**<br>secureforge distros | **23**<br>secureforge modules |
| **27**<br>Astracor builtins | **17**<br>Aether keywords | **18**<br>Aether CLI subcommands | **10**<br>AstraOS capability bits |
| **16**<br>SCSA registers | **64 MiB**<br>SCSA mmap RAM | **302.7M**<br>kittybrain parameters | **22**<br>archcraft passes |

</div>

<img src="assets/divider.svg" width="100%" alt="">

## Measured

Counted with `find | xargs wc -l` on a clean clone. Nothing vendored, generated or
bundled is included in any figure.

```
aether             16,419   C++23     compiler front to back, 101 files in src/
AstraOS            12,740   C + NASM  i686 kernel, no libc, ring 0 -> ring 3
secureforge         9,666   JS (ESM)  99 files, 23 hardening modules, 9 distros
ModularX            7,579   6 langs   19 small libraries + a JS orchestrator
RESS                4,430   Rust      10-crate workspace, fanotify exec firewall
hvc                 4,039   C23       C -> IR compiler + register VM
Security-Recovery   3,474   C         firmware recovery state machine
SCSA                1,151   C         one translation unit: tokenizer -> VM
ai-rofi-launcher    1,098   Go        local AI server with an embedded web UI
kittybrain            325   Python    302M-param char-level GPT, from scratch
archcraft-full-upg    223   Bash      22-pass Arch maintenance script
                  ───────
                   61,144   lines
```

A few of these carry a second language that is not in the column: 984 lines of Python
CLI in Security-Recovery, 712 lines of Bash under ai-rofi-launcher.

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#3d59a1','primaryTextColor':'#ffffff','lineColor':'#7aa2f7'}}}%%
xychart-beta
    title "Hand-written lines per repo"
    x-axis ["aether", "AstraOS", "secureforge", "ModularX", "RESS", "hvc", "Sec-Rec", "SCSA", "rofi", "kitty", "archcraft"]
    y-axis "lines" 0 --> 17000
    bar [16419, 12740, 9666, 7579, 4430, 4039, 3474, 1151, 1098, 325, 223]
```

<img src="assets/divider.svg" width="100%" alt="">

## Languages, by measured bytes

GitHub's per-language byte counts across my own repositories, forks excluded. These
are Linguist's numbers, so a share of the Makefile and CMake rows is generated build
output rather than hand-written — see the by-commit card below, which does not have
that problem.

```
LANGUAGE            BYTES   SHARE - one block = 1%              PCT
───────────────────────────────────────────────────────────────────
C                 751,494   ██████████████████████████████▉  30.81%
C++               733,748   ██████████████████████████████▏  30.09%
JavaScript        416,810   █████████████████▏               17.09%
Rust              159,061   ██████▌                           6.52%
Python             86,370   ███▌                              3.54%
Makefile           76,925   ███▏                              3.15%
HTML               53,409   ██▎                               2.19%
Shell              52,704   ██▏                               2.16%
Ruby               31,144   █▎                                1.28%
CMake              28,079   █▏                                1.15%
Go                 25,445   █                                 1.04%
Assembly           21,067   ▉                                 0.86%
Linker Script       1,956   ▏                                 0.08%
TypeScript            366                                     0.02%
Batchfile             248                                     0.01%
───────────────────────────────────────────────────────────────────
total           2,438,826                                   100.00%
```

C and C++ together are 60.9% of every byte. By **commit** volume the mix reads
`C · Shell · C++ · Rust · Python` — the JavaScript share above is one hardening
wizard and one library collection, and it compresses away the moment you count
changes instead of characters.

```mermaid
%%{init: {'theme':'base','themeVariables':{'pie1':'#7aa2f7','pie2':'#bb9af7','pie3':'#e0af68','pie4':'#f7768e','pie5':'#9ece6a','pie6':'#7dcfff','pie7':'#ff9e64','pie8':'#73daca','pie9':'#565f89','pieTitleTextColor':'#c0caf5','pieSectionTextColor':'#1a1b27','pieLegendTextColor':'#c0caf5'}}}%%
pie showData title Bytes by language (Linguist, forks excluded)
    "C" : 751494
    "C++" : 733748
    "JavaScript" : 416810
    "Rust" : 159061
    "Python" : 86370
    "Makefile" : 76925
    "HTML" : 53409
    "Shell" : 52704
    "Other (7 langs)" : 108305
```

**Where the bytes come from — every language traced to a project:**

| Language | Concentrated in | Doing what |
| --- | --- | --- |
| **C** | AstraOS, SCSA, hvc, Security-Recovery | Kernel and drivers, a single-file compiler+VM, a firmware state machine |
| **C++** | aether, ModularX | A compiler front to back, plus cache and logger libraries |
| **Rust** | RESS, ModularX | A 10-crate fanotify execution firewall, plus a small async cache crate |
| **JavaScript** | secureforge, ModularX, SmartRateLimiter | A 99-file hardening wizard, a cross-language library collection, a rate limiter |
| **Assembly** | AstraOS, ModularX | 412 lines of NASM for boot, ISRs and ring switches; 564 lines of x86-64 bit/SIMD utilities |
| **Python** | kittybrain, Security-Recovery, ModularX | Transformer training, a PAM-authenticated CLI, async utilities |
| **Shell** | archcraft-full-upg, ai-rofi-launcher, secureforge | System maintenance, multi-distro installers |
| **Go** | ai-rofi-launcher | A localhost AI proxy with the web UI compiled into the binary |
| **HTML** | mcss, ai-rofi-launcher | One hand-written 894-line landing page; one 467-line self-contained web UI compiled into a Go binary |

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://github-profile-summary-cards.vercel.app/api/cards/repos-per-language?username=D3LTA2033&theme=tokyonight">
  <source media="(prefers-color-scheme: light)" srcset="https://github-profile-summary-cards.vercel.app/api/cards/repos-per-language?username=D3LTA2033&theme=github">
  <img alt="Top languages by repository" src="https://github-profile-summary-cards.vercel.app/api/cards/repos-per-language?username=D3LTA2033&theme=tokyonight">
</picture>
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://github-profile-summary-cards.vercel.app/api/cards/most-commit-language?username=D3LTA2033&theme=tokyonight">
  <source media="(prefers-color-scheme: light)" srcset="https://github-profile-summary-cards.vercel.app/api/cards/most-commit-language?username=D3LTA2033&theme=github">
  <img alt="Top languages by commit volume" src="https://github-profile-summary-cards.vercel.app/api/cards/most-commit-language?username=D3LTA2033&theme=tokyonight">
</picture>
</p>

<img src="assets/divider.svg" width="100%" alt="">

## How it fits together

Not a dependency graph — a map of which layer each project lives on.

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#24283b','primaryTextColor':'#c0caf5','primaryBorderColor':'#7aa2f7','lineColor':'#7aa2f7','secondaryColor':'#1a1b27','tertiaryColor':'#1a1b27'}}}%%
flowchart TB
  subgraph PROD["Product"]
    V["Venoly<br/>chat · voice/video · communities"]
    VD["venoly-desktops<br/>NSIS · portable · AppImage · deb"]
  end

  subgraph DEF["Defence"]
    R["RESS<br/>fanotify exec firewall"]
    SF["secureforge<br/>reversible hardening"]
  end

  subgraph LANG["Languages and compilers"]
    AE["Aether<br/>C++23 → C → gcc -O2"]
    HV["hvc<br/>C → IR → register VM"]
    SC["SCSA<br/>tokenizer → bytecode → VM"]
  end

  subgraph LOW["Metal"]
    A["AstraOS<br/>i686 kernel, no libc"]
    SR["Security-Recovery<br/>firmware recovery"]
  end

  subgraph TOOL["Tooling and experiments"]
    MX["ModularX"]
    RF["ai-rofi-launcher"]
    KB["kittybrain"]
    AC["archcraft-full-upg"]
  end

  V --> VD
  SF -. "hardens the hosts" .-> V
  R -. "gates execve on" .-> A
  A --> AS["Astracor<br/>in-kernel language"]
  AE --> G["gcc"]

  classDef prod fill:#3d59a1,stroke:#7aa2f7,color:#fff
  classDef def fill:#5a3e8e,stroke:#bb9af7,color:#fff
  class V,VD prod
  class R,SF def
```

> The two dotted edges are intent, not code: RESS targets Linux generally, not
> AstraOS, and secureforge is a tool for Linux hosts. They show the layer each one
> protects.

<img src="assets/divider.svg" width="100%" alt="">

## Patterns across the repos

| | |
|:--|:--|
| **Counted, not claimed** | Every size in this file is `wc -l`, Linguist, or measured over the wire. |
| **Observe before you enforce** | RESS ships an `--observe` mode that logs verdicts without gating, so a policy can be tuned before it bites. |
| **Reversible by construction** | secureforge records a manifest and code-generates its own uninstaller from it. |
| **Small trusted base** | AstraOS uses only three compiler-provided headers. SCSA is one translation unit. RESS hand-rolls its glob matcher and its sandbox. |
| **Say what is unfinished** | secureforge prints full-screen ALPHA/BETA warnings before touching anything. Limits are listed [below](#read-before-you-run). |
| **Checksums on shipped binaries** | Every Venoly desktop artifact has a line in `SHA256SUMS-1.0.0.txt`. |

### The same work, as a map

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#24283b','primaryTextColor':'#c0caf5','primaryBorderColor':'#7aa2f7','lineColor':'#7aa2f7'}}}%%
mindmap
  root((mcss))
    Kernels
      AstraOS
      Security-Recovery
    Compilers
      Aether
      hvc
      SCSA
    Defence
      RESS
      secureforge
    Product
      Venoly
      venoly-desktops
    Experiments
      kittybrain
      ai-rofi-launcher
      archcraft-full-upg
```

<img src="assets/divider.svg" width="100%" alt="">

## Detail

Click to open. Each section is measured from the repo, not estimated.

<details>
<summary><b>AstraOS</b> — 12,740 hand-written lines, no libc, its own GUI and its own language · <a href="https://github.com/D3LTA2033/AstraOS">repo</a></summary>

<br>

10,289 lines of C across 35 files, 2,039 lines of headers across 36, and 412 lines
of NASM across 7. Nothing vendored, nothing generated. All 32 kernel translation
units and all 3 userland programs compile clean under
`-m32 -std=c11 -ffreestanding -fno-builtin -nostdlib -Wall -Wextra -Werror -O2` —
32 of 32, zero failures.

The only system headers anywhere in `kernel/`, `include/`, `arch/` or `user/` are the
three the compiler itself provides: `<stdint.h>`, `<stddef.h>`, `<stdbool.h>`.
`kstrlen`, `kmemcpy` and friends are hand-rolled in `kernel/lib/kstring.c`.

**Address space.** Real x86 two-level paging — 1024-entry page directory and page
tables, `PAGE_USER` on ring-3 pages, physical frames from a bitmap allocator that
walks the GRUB multiboot memory map. `pmm.c` 220 lines, `vmm.c` 301, `kheap.c` 211,
`page_fault.c` 72.

```
 0xC000_0000  ┌────────────────────────────┐  kernel heap base   kheap.c
              │  kernel heap               │
              ├─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ──┤
              │  ~3 GiB unmapped        ⋮  │  not to scale
              ├─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ──┤  _kernel_end        linker.ld
              │  .text .rodata .data .bss  │
 0x0010_0000  ├────────────────────────────┤  1 MiB, Multiboot 1 load address
              │  low memory / BIOS area    │
 0x0000_0000  └────────────────────────────┘
```

`boot.asm` carries a Multiboot 1 header that asks GRUB for a 1024×768×32 linear VESA
framebuffer. `linker.ld` places the kernel at 1 MiB with 4 KiB-aligned sections and
exports `_kernel_end` for the allocator.

**Ring transition.** `usermode.asm` hand-builds an `IRET` frame with `SS = 0x23` and
`CS = 0x1B` — RPL 3, interrupts re-enabled. `context_switch.asm` is a genuine
callee-saved switch over `EFLAGS / EDI / ESI / EBX / EBP` plus an `ESP` handoff.
`isr.asm` generates 48 stubs: 32 CPU exception entries (23 without an error code,
9 with) and 16 IRQ entries.

**Scheduler.** Preemptive round robin, 50 ms quantum (5 PIT ticks at 10 ms), an idle
task that `hlt`s, lazy reaping of dead tasks. 306 lines.

```mermaid
stateDiagram-v2
    [*] --> Ready
    Ready --> Running: schedule()
    Running --> Ready: PIT tick — quantum expired
    Running --> Blocked: sleep / read / waitpid
    Blocked --> Ready: IRQ or signal
    Ready --> Idle: run queue empty
    Idle --> Ready: hlt until IRQ
    Running --> Zombie: exit()
    Zombie --> [*]: lazy reap
```

**Syscalls.** Sixteen, every one implemented rather than stubbed, dispatched through
a 555-line `kernel/core/syscall.c`:

| group | calls |
|---|---|
| process | `exit` · `getpid` · `getppid` · `yield` · `sleep` · `spawn` · `waitpid` |
| vfs | `open` · `read` · `write` · `close` · `stat` · `readdir` |
| ipc & signals | `pipe` · `kill` |
| system | `reboot` |

**Capabilities.** A 32-bit per-task mask, strictly subtractive on inheritance, with
`cap_grant` / `cap_revoke` and a `security_audit()` trail over serial.

```c
/* ten defined bits; a child task can only ever hold a subset of its parent's */
CAP_VFS_READ       CAP_VFS_WRITE      CAP_DEVICE_ACCESS
CAP_PROC_CREATE    CAP_SYS_ADMIN      CAP_RAW_IO
CAP_NET            CAP_SIGNAL         CAP_MOUNT
CAP_AUDIT_READ
```

**Above the kernel.** A VFS with ramfs, devfs and initramfs as separate sources, plus
procfs, pipes and signals. A 557-line VESA framebuffer driver doing real per-pixel
alpha blending, `(src·a + dst·(255−a)) / 255`, over a double buffer, with an 8×16
bitmap font; PS/2 keyboard and mouse, serial, RTC, PIT, VGA text. On top of that a
518-line compositing window manager, 1,860 lines of built-in applications — terminal,
multi-tab code editor, file manager, settings — and a 609-line first-boot installer.

```mermaid
flowchart BT
  HW["x86 hardware<br/>PIT · PS/2 · RTC · serial · VESA"] --> BOOT["boot.asm<br/>Multiboot 1, loaded at 1 MiB"]
  BOOT --> MM["memory<br/>pmm · vmm · kheap · page_fault"]
  MM --> SCH["scheduler<br/>50 ms quantum"]
  SCH --> SYS["16 syscalls + capabilities"]
  SYS --> VFS["VFS<br/>ramfs · devfs · initramfs · procfs · pipes"]
  VFS --> WM["compositing window manager"]
  WM --> APP["terminal · editor · file manager · settings"]
```

**Astracor** is a language living inside the kernel — a 1,434-line recursive-descent
tree-walking interpreter with dynamic typing, user functions, `if/elif/else`, `while`,
`for-in`, arrays, parent-linked scope environments, 27 native builtins, and four
importable modules (`math`, `string`, `array`, `io`) whose sources are embedded as C
string literals. Tree-walking only: no bytecode path, and functions capture no
environment, so no closures.

```console
$ make       # kernel ELF - needs i686-elf-gcc + nasm
$ make iso   # grub-mkrescue -> build/astraos.iso
$ make run   # qemu-system-i386 -cdrom build/astraos.iso -m 128M -serial stdio
$ ./scripts/run-qemu.sh --debug   # same, paused, GDB server on :1234
```

Targets i686 — the Makefile builds with `i686-elf-gcc` and boots under
`qemu-system-i386`.

</details>

<details>
<summary><b>RESS</b> — a Linux execution firewall in 4,430 lines of Rust · <a href="https://github.com/D3LTA2033/RESS-Remote-Execution-Security-System">repo</a></summary>

<br>

A ten-crate Cargo workspace that sits in front of `execve`. The kernel stops every
exec and waits for a userspace verdict; the daemon dissects the ELF, scores it, runs
it through a policy engine and answers. `cargo check --workspace --all-targets`
completes with zero errors and zero warnings across all ten crates.

```mermaid
flowchart TD
  E["process calls execve()"] --> K["kernel: fanotify<br/>FAN_OPEN_EXEC_PERM on /"]
  K -->|"exec held, waiting"| D["ress-agent"]
  D --> A["static analysis<br/>goblin ELF, Shannon entropy<br/>40+ risky imports, 13 heuristics"]
  A --> S["risk score 0-100<br/>Safe 0-24, Suspicious 25-59, Malicious 60-100"]
  S --> P["policy engine<br/>12 predicates, all/any/not<br/>priority, first match"]
  P --> V1["Allow — FAN_ALLOW"]
  P --> V2["Sandbox — unshare + overlayfs + seccomp + cgroup v2"]
  P --> V3["Block — FAN_DENY"]
  P --> V4["Quarantine"]
  P --> V5["EscalateApproval"]
  V1 --> R["fanotify_response written to the fd"]
  V3 --> R
```

**Interception is `fanotify`** — not eBPF, not an LSM, not `ptrace`. `fanotify_init`
with `FAN_CLASS_CONTENT | FAN_UNLIMITED_QUEUE | FAN_UNLIMITED_MARKS`, the whole `/`
mount marked `FAN_MARK_MOUNT | FAN_OPEN_EXEC_PERM`, raw `fanotify_event_metadata`
read off the fd, a `fanotify_response` of `FAN_ALLOW` (0x01) or `FAN_DENY` (0x02)
written back. The kernel genuinely holds the exec until the answer arrives.

```mermaid
sequenceDiagram
    participant P as process
    participant K as kernel (fanotify)
    participant A as ress-agent
    participant E as policy engine
    P->>K: execve()
    K-->>A: FAN_OPEN_EXEC_PERM event, exec held
    A->>A: parse ELF, entropy, imports, heuristics
    A->>E: risk score 0-100 plus features
    E-->>A: verdict from the first matching rule
    A->>K: fanotify_response (FAN_ALLOW or FAN_DENY)
    K-->>P: exec proceeds or is refused
```

**Scoring.** A `goblin` ELF parse extracting RELRO, BIND_NOW, NX, PIE, static,
stripped, RPATH, RUNPATH, SONAME, interpreter, `DT_NEEDED`, imports and exports.
Per-section Shannon entropy. A 40-plus entry weighted risky-import table bucketed by
category — exec, process, antidebug, privilege, network, memory, loader — each
category capped at 14.0 so no single axis can dominate. 13 regex string heuristics
(`reverse_shell`, `curl_pipe`, `ld_preload`, `ransom`, `c2_proto`, `antidebug`,
`persistence`, `self_delete`, `kernel_obj`, `dns_exfil`, …) and a packing heuristic.

**Policy** is a YAML DSL: twelve predicate atoms — `risk_score_at_least`,
`risk_category_in`, `path_prefix`, `path_glob`, `uid_equals`, `uid_at_least`,
`euid_equals`, `is_setuid`, `digest_in`, `has_feature`, `always`, `never` — combined
with `all` / `any` / `not`, sorted by integer priority, first match wins, and
hot-reloadable over `SIGHUP`. The glob matcher is hand-rolled backtracking, no `glob`
crate. The shipped bundle is seven rules in 81 lines: allow package-managed system
binaries under risk 50, block malicious, quarantine packed binaries carrying
persistence markers, escalate non-system setuid, sandbox anything from `/tmp`,
`/var/tmp`, `/dev/shm` or a user's `Downloads`, sandbox risk ≥ 35 for non-root, allow
low-risk user binaries.

**The sandbox is hand-rolled** — no bubblewrap, no Docker. `fork()`, then
`unshare(CLONE_NEWNS|CLONE_NEWPID|CLONE_NEWUTS|CLONE_NEWIPC|CLONE_NEWNET)`, an
ephemeral overlayfs root, `sethostname`, `chroot`, `setresuid`/`setresgid` to 65534,
all five capability sets cleared (effective, permitted, inheritable, ambient,
bounding), `PR_SET_NO_NEW_PRIVS`, `PR_SET_DUMPABLE = 0`, a seccomp BPF program built
with `seccompiler` against a hand-written syscall table covering x86_64 *and*
aarch64 — `init_module`, `finit_module`, `kexec_load`, `bpf`, `userfaultfd`,
`perf_event_open`, `setns`, `memfd_create`, `capset` and about two dozen more denied
to `EPERM` — then `execve` with a scrubbed `envp`.

**cgroup v2** is written straight against sysfs: create `/sys/fs/cgroup/ress/<id>`,
write `+memory +cpu +pids +io` to `cgroup.subtree_control`, then `memory.max`,
`pids.max`, `cpu.max`, attach via `cgroup.procs`, and `cgroup.kill` to tear the whole
tree down. A runtime supervisor `SIGKILL`s anything over budget; `Drop` removes the
cgroup.

**Telemetry** is hand-written netlink: `socket(AF_NETLINK, SOCK_DGRAM, NETLINK_CONNECTOR)`
bound to `CN_IDX_PROC`, `PROC_CN_MCAST_LISTEN` sent, and `#[repr(C)]` `nlmsghdr` /
`cn_msg` / `proc_event` parsed by hand for `FORK`, `EXEC`, `UID` and `EXIT`, fanned
out over a tokio broadcast alongside an inotify watcher on the world-writable dirs.

```console
$ cargo build --release -p ress-cli
$ ./target/release/ress analyze /usr/bin/ls
elf         class=ELF64  machine=x86_64  pie=true  static=false  stripped=true
hardening   relro=true  bind_now=true  nx=true
imports     112 symbols
entropy     6.023
RISK        score=14  category=Safe
```

*Abridged — the full output also prints the sha256 digest, the interpreter and the
`DT_NEEDED` list. The import count and entropy are specific to this machine's build
of coreutils.*

Apache-2.0. Eight subcommands, two systemd units, an install/uninstall pair, an
append-only JSONL audit log, per-execution forensic JSON, and an `--observe` mode
that logs verdicts without gating anything, so a policy can be tuned before it bites.

Enforcement needs `CAP_SYS_ADMIN`; `--observe` runs the whole analysis and policy
path without it.

</details>

<details>
<summary><b>Aether</b> — a systems language in 16,419 lines of C++23 · <a href="https://github.com/D3LTA2033/aether">repo</a></summary>

<br>

A front-to-back compiler: it lexes, parses, type-checks, lowers to an HIR, optimizes,
emits C, and hands the C to `gcc -O2`. `src/cli/main.cpp`'s `buildFile()` runs
`Lexer → Parser → SemanticAnalyzer → HIRLowering → Optimizer → CodeGen → Linker` with
real dataflow between the stages — these are not stub calls, and all eight core
translation units pass `g++ -std=c++23 -Isrc -fsyntax-only` on their own.

```
  stage line counts, C++

  .ae ──► Lexer ──► Parser ──► AST ──► Semantic ──► HIR
           236       785                 351        134
                                                     │
                                                     ▼
        native binary ◄── Linker ◄── CodeGen ◄── Optimizer
                          gcc -O2    emits C        334
```

**Optimizer.** An abstract `OptimizerPass` base with five concrete passes —
`ConstantFolding`, `DeadCodeElimination`, `UnusedVariableRemoval`, `FunctionInlining`,
`ExpressionSimplification` — four of which are registered in the default pipeline.

**Memory model.** `src/runtime/arc.cpp`, 393 lines: `ARCObject` retain/release,
`WeakRef`, a depth-first cycle detector, a borrow checker tracking mutable and shared
borrow state, a stack promoter, and `ARCPtr` / `WeakARCPtr` smart pointers. ARC, not
a GC.

**The language.** 17 keywords — `let var mut fn return if else match struct enum impl
defer task parallel unsafe module import` — 22 AST node types including `MatchExpr`,
`DeferStmt`, `TaskDecl` and `ParallelBlock`, and generics in square brackets:

```rust
fn identity[T](value: T) -> T { return value; }
```

`examples/hello.ae`, run end to end with the prebuilt binary committed at the repo
root:

```rust
module main

fn main() -> int {
    print("Hello, World!");
    return 0;
}
```

```console
$ ./aether run examples/hello.ae
Built examples/hello.ae -> examples/hello
Hello, World!
```

101 source files in `src/`; 24,454 lines of C++ repo-wide once validation, examples
and benchmarks are counted; 43 Markdown specification and governance documents
totalling 17,969 lines. The CLI dispatches 18 subcommands: `new add remove build run
test fmt lint doc publish search install bench profile clean doctor repl help`.

`examples/hello.ae.c` is a checked-in sample of the C the backend emits.

</details>

<details>
<summary><b>SCSA</b> — a whole toolchain in one 1,151-line C file · <a href="https://github.com/D3LTA2033/Super-Compiling-Security-Assemblor">repo</a></summary>

<br>

`src/scsa.c` is 1,151 lines, 40,443 bytes, and the entire program. The repository
contains zero header files. Inside it: a tokenizer, a parser, a bytecode compiler, a
16-register virtual machine, an interactive REPL and a module store. Ported by hand
to Linux, win32 and macOS — three copies of the file, roughly 95% shared, with 58-
and 77-line diffs.

```
  demo.scsa
      │
      ▼
  scsa_tokenize ──► scsa_parse ──► scsa_compile ──► demo.scsa.sc
                       AST         fixed-width         bytecode
                                      Instr               │
                                                          ▼
                                                      scsa_exec
                                            ┌────────────────────────┐
                                            │ 16 x uint64 registers  │
                                            │ 64 MiB mmap RAM        │
                                            │ 1024-slot stack        │
                                            │ pthread-guarded        │
                                            └────────────────────────┘
```

**Container format.** `--compile` emits a flat file: a four-byte magic followed by
fixed-width instruction records.

```
  offset   size   field
  0x0000      4   magic  0x53430000
  0x0004    280   Instr[0]
  0x011C    280   Instr[1]
   ...      280   Instr[n]
```

The magic `0x53430000` is the bytes `0x53 0x43` — ASCII `S` `C` — followed by two zero bytes.

**The ISA.** The VM's dispatch switch executes eighteen opcodes:

| class | opcodes |
|---|---|
| move / memory | `LOAD` `STORE` `MOV` |
| arithmetic | `ADD` `SUB` `INC` `DEC` |
| bitwise | `AND` `OR` `XOR` `NOT` `SHL` `SHR` |
| control | `JMP` `CMP` `HALT` |
| io | `PRINT` `LOG` |

```console
$ printf 'INC 1;\nSHL 1 3;\nPRINT 1;\nHALT;\n' > demo.scsa
$ ./build/scsa --run demo.scsa
8
$ ./build/scsa --compile demo.scsa      # -> demo.scsa.sc
$ ./build/scsa                          # no args -> REPL
```

Statements are semicolon-terminated: the tokenizer eats whitespace wholesale, so a
newline is not a statement separator and a newline-separated program compiles to
exactly one instruction. Built with `-O2 -Wall -Wextra -fstack-protector-strong
-D_FORTIFY_SOURCE=2 -fPIE -pie -Wl,-z,relro,-z,now`; the output is a PIE ELF.

One thing to know before you run it: `main()` calls `scsa_auto_install()` before it
parses `argv`, so any invocation copies the binary into `~/.local/bin` and creates
`~/.scsa/modules/`.

</details>

<details>
<summary><b>secureforge</b> — 9,666 lines of JS that write bash you can take back out · <a href="https://github.com/D3LTA2033/secureforge">repo</a></summary>

<br>

99 hand-written ESM files, 342,172 bytes, no bundler, no minified vendor blob,
`node_modules` gitignored. Nine distributions are genuinely wired up, each with its
own module directory: arch, debian, ubuntu, fedora, rhel, centos, opensuse, gentoo,
alpine. 23 unique modules in source; 21 to 22 are active per distro (`scap` on
RHEL/CentOS, `portage` on Gentoo).

Per-distro implementations of `ssh network kernel firewall login root audit
filesystem services faketwinlogin`; shared implementations of `usbguard ntp
grubpassword crypto ids acct tarpit canary geofence memforge procguard`. Firewall
branches cover UFW, nftables, firewalld and awall, and init-system awareness is real:
OpenRC scripts for Alpine and Gentoo, `NoNewPrivileges` systemd units elsewhere.

The sysctl fragment:

```
kernel.randomize_va_space   kernel.kptr_restrict     kernel.yama.ptrace_scope
kernel.unprivileged_bpf_disabled                     net.core.bpf_jit_harden
kernel.perf_event_paranoid  kernel.sysrq             fs.suid_dumpable
fs.protected_hardlinks      fs.protected_symlinks    fs.protected_fifos
fs.protected_regular        net.ipv4.tcp_syncookies  net.ipv4.tcp_rfc1337
net.ipv4.conf.*.rp_filter
```

The auditd ruleset is CIS-flavoured for real — `-b 8192`, `-f 1`, write and attribute
watches on `passwd/group/shadow/gshadow/sudoers/pam.d`,
`-a always,exit -F arch=b64 -S execve`, `-S setuid -S setgid`,
`-S init_module -S delete_module`, and delete tracking with
`-F auid>=1000 -F auid!=4294967295`.

**Changes are reversible.** `src/utils/writer.js` records a manifest at
`/etc/secureforge/manifest.json` (mode 600) and then code-generates
`/usr/local/bin/secureforge-uninstall` line by line from it — restoring backed-up
configs, deleting created files, re-enabling disabled services. Around 200 distinct
system paths appear across the generated scripts; the manifest records the subset it
backs up, creates or disables, and the uninstaller is generated from that.

```mermaid
flowchart LR
  W["wizard run"] --> M["manifest.json<br/>mode 600"]
  W --> C["apply changes<br/>backup, create, disable"]
  M --> U["secureforge-uninstall<br/>code-generated"]
  U --> RB["restore configs<br/>delete created files<br/>re-enable services"]
```

Deception modules do what they say: `canary` plants a `passwords.txt`, a
syntactically valid fake `id_rsa`, `.aws-credentials` and `db_backup.sql`, then runs
`inotifywait -m -r -e access,open,read`, resolves the reading PID through `lsof`,
reads `/proc/<pid>/comm`, and logs, webhooks or kills. `tarpit` builds `endlessh`
from source and moves the real sshd. `faketwinlogin` is a PAM duress password.
`maturity.js` prints full-screen ALPHA/BETA warnings per distro and per module before
anything is touched.

```console
$ git clone https://github.com/D3LTA2033/secureforge && cd secureforge
$ npm install
$ node index.js --distro arch --list
```

Pass `--distro` explicitly, and try it in a VM first. Licensed under a hand-written
SecureForge Public License with a no-offensive-use clause.

</details>

<img src="assets/divider.svg" width="100%" alt="">

## Also in the tree

<details>
<summary><b>ai-rofi-launcher</b> — hotkey AI chat for Linux · 1,098 Go · 712 Bash · <a href="https://github.com/D3LTA2033/ai-rofi-launcher">repo</a></summary>

<br>

A Go server with the web UI `go:embed`ded and bound to `127.0.0.1`, five providers,
and API keys that stay server-side and never reach the browser. Bubble Tea TUI for
config, rofi popup as the fallback surface. MIT.

The web UI is a single self-contained 467-line HTML file compiled into the binary.

</details>

<details>
<summary><b>ModularX</b> — 19 small libraries, 6 languages · 7,579 lines · <a href="https://github.com/D3LTA2033/ModularX">repo</a></summary>

<br>

19 small libraries across C++, Rust, JS, Python, Ruby and x86-64 NASM, plus a JS
orchestrator. Includes cache and logger libraries in C++, a small async cache crate in
Rust, async utilities in Python, and 564 lines of x86-64 bit/SIMD utilities in NASM.

</details>

<details>
<summary><b>hvc</b> — a second compiler and register VM · 4,039 lines of C · <a href="https://github.com/D3LTA2033/hvc">repo</a></summary>

<br>

Lexer, precedence-climbing parser, scoped symbol table, IR builder, five optimizer
passes, and an `HVBC` bytecode container, executed by a register VM. MIT.

</details>

<details>
<summary><b>Security-Recovery</b> — firmware recovery · 3,474 C · 984 Python · <a href="https://github.com/D3LTA2033/Security-Recovery-Core-SRC">repo</a></summary>

<br>

An 8-state recovery machine, A/B backup rotation, a documented 16 MB SPI layout, and a
Python CLI with HMAC-SHA256-tagged state and PAM-gated operations.

</details>

<details>
<summary><b>kittybrain</b> — a character-level GPT from scratch · 325-line notebook · <a href="https://github.com/D3LTA2033/kittybrain">repo</a></summary>

<br>

A ~302.7M-parameter character-level GPT in raw PyTorch — 24 pre-LN blocks, `d_model`
1024, 16 heads, causal `-inf` mask, no HuggingFace — streaming TinyStories, time-boxed
to a 70-minute GPU window.

</details>

<details>
<summary><b>archcraft-full-upg</b> — one Arch maintenance script · 223 lines of Bash · <a href="https://github.com/D3LTA2033/archcraft-full-upg">repo</a></summary>

<br>

22 labelled passes over an Arch box — keyring, AUR, bootloader, microcode, initramfs,
SMART, fstab, cache — with an optional `rsync` snapshot first.

</details>

<img src="assets/divider.svg" width="100%" alt="">

## Repo pulse

Live badges, straight from GitHub — they update on their own.

| repo | top language | stars | last commit | license |
|:--|:-:|:-:|:-:|:-:|
| [AstraOS](https://github.com/D3LTA2033/AstraOS) | ![lang](https://img.shields.io/github/languages/top/D3LTA2033/AstraOS?style=flat&labelColor=545454&color=313131&label=) | ![stars](https://img.shields.io/github/stars/D3LTA2033/AstraOS?style=flat&labelColor=545454&color=313131&label=) | ![commit](https://img.shields.io/github/last-commit/D3LTA2033/AstraOS?style=flat&labelColor=545454&color=313131&label=) | ![license](https://img.shields.io/github/license/D3LTA2033/AstraOS?style=flat&labelColor=545454&color=313131&label=) |
| [RESS](https://github.com/D3LTA2033/RESS-Remote-Execution-Security-System) | ![lang](https://img.shields.io/github/languages/top/D3LTA2033/RESS-Remote-Execution-Security-System?style=flat&labelColor=545454&color=313131&label=) | ![stars](https://img.shields.io/github/stars/D3LTA2033/RESS-Remote-Execution-Security-System?style=flat&labelColor=545454&color=313131&label=) | ![commit](https://img.shields.io/github/last-commit/D3LTA2033/RESS-Remote-Execution-Security-System?style=flat&labelColor=545454&color=313131&label=) | ![license](https://img.shields.io/github/license/D3LTA2033/RESS-Remote-Execution-Security-System?style=flat&labelColor=545454&color=313131&label=) |
| [aether](https://github.com/D3LTA2033/aether) | ![lang](https://img.shields.io/github/languages/top/D3LTA2033/aether?style=flat&labelColor=545454&color=313131&label=) | ![stars](https://img.shields.io/github/stars/D3LTA2033/aether?style=flat&labelColor=545454&color=313131&label=) | ![commit](https://img.shields.io/github/last-commit/D3LTA2033/aether?style=flat&labelColor=545454&color=313131&label=) | ![license](https://img.shields.io/github/license/D3LTA2033/aether?style=flat&labelColor=545454&color=313131&label=) |
| [SCSA](https://github.com/D3LTA2033/Super-Compiling-Security-Assemblor) | ![lang](https://img.shields.io/github/languages/top/D3LTA2033/Super-Compiling-Security-Assemblor?style=flat&labelColor=545454&color=313131&label=) | ![stars](https://img.shields.io/github/stars/D3LTA2033/Super-Compiling-Security-Assemblor?style=flat&labelColor=545454&color=313131&label=) | ![commit](https://img.shields.io/github/last-commit/D3LTA2033/Super-Compiling-Security-Assemblor?style=flat&labelColor=545454&color=313131&label=) | ![license](https://img.shields.io/github/license/D3LTA2033/Super-Compiling-Security-Assemblor?style=flat&labelColor=545454&color=313131&label=) |
| [secureforge](https://github.com/D3LTA2033/secureforge) | ![lang](https://img.shields.io/github/languages/top/D3LTA2033/secureforge?style=flat&labelColor=545454&color=313131&label=) | ![stars](https://img.shields.io/github/stars/D3LTA2033/secureforge?style=flat&labelColor=545454&color=313131&label=) | ![commit](https://img.shields.io/github/last-commit/D3LTA2033/secureforge?style=flat&labelColor=545454&color=313131&label=) | ![license](https://img.shields.io/github/license/D3LTA2033/secureforge?style=flat&labelColor=545454&color=313131&label=) |
| [hvc](https://github.com/D3LTA2033/hvc) | ![lang](https://img.shields.io/github/languages/top/D3LTA2033/hvc?style=flat&labelColor=545454&color=313131&label=) | ![stars](https://img.shields.io/github/stars/D3LTA2033/hvc?style=flat&labelColor=545454&color=313131&label=) | ![commit](https://img.shields.io/github/last-commit/D3LTA2033/hvc?style=flat&labelColor=545454&color=313131&label=) | ![license](https://img.shields.io/github/license/D3LTA2033/hvc?style=flat&labelColor=545454&color=313131&label=) |
| [Security-Recovery](https://github.com/D3LTA2033/Security-Recovery-Core-SRC) | ![lang](https://img.shields.io/github/languages/top/D3LTA2033/Security-Recovery-Core-SRC?style=flat&labelColor=545454&color=313131&label=) | ![stars](https://img.shields.io/github/stars/D3LTA2033/Security-Recovery-Core-SRC?style=flat&labelColor=545454&color=313131&label=) | ![commit](https://img.shields.io/github/last-commit/D3LTA2033/Security-Recovery-Core-SRC?style=flat&labelColor=545454&color=313131&label=) | ![license](https://img.shields.io/github/license/D3LTA2033/Security-Recovery-Core-SRC?style=flat&labelColor=545454&color=313131&label=) |
| [ModularX](https://github.com/D3LTA2033/ModularX) | ![lang](https://img.shields.io/github/languages/top/D3LTA2033/ModularX?style=flat&labelColor=545454&color=313131&label=) | ![stars](https://img.shields.io/github/stars/D3LTA2033/ModularX?style=flat&labelColor=545454&color=313131&label=) | ![commit](https://img.shields.io/github/last-commit/D3LTA2033/ModularX?style=flat&labelColor=545454&color=313131&label=) | ![license](https://img.shields.io/github/license/D3LTA2033/ModularX?style=flat&labelColor=545454&color=313131&label=) |
| [ai-rofi-launcher](https://github.com/D3LTA2033/ai-rofi-launcher) | ![lang](https://img.shields.io/github/languages/top/D3LTA2033/ai-rofi-launcher?style=flat&labelColor=545454&color=313131&label=) | ![stars](https://img.shields.io/github/stars/D3LTA2033/ai-rofi-launcher?style=flat&labelColor=545454&color=313131&label=) | ![commit](https://img.shields.io/github/last-commit/D3LTA2033/ai-rofi-launcher?style=flat&labelColor=545454&color=313131&label=) | ![license](https://img.shields.io/github/license/D3LTA2033/ai-rofi-launcher?style=flat&labelColor=545454&color=313131&label=) |
| [kittybrain](https://github.com/D3LTA2033/kittybrain) | ![lang](https://img.shields.io/github/languages/top/D3LTA2033/kittybrain?style=flat&labelColor=545454&color=313131&label=) | ![stars](https://img.shields.io/github/stars/D3LTA2033/kittybrain?style=flat&labelColor=545454&color=313131&label=) | ![commit](https://img.shields.io/github/last-commit/D3LTA2033/kittybrain?style=flat&labelColor=545454&color=313131&label=) | ![license](https://img.shields.io/github/license/D3LTA2033/kittybrain?style=flat&labelColor=545454&color=313131&label=) |
| [archcraft-full-upg](https://github.com/D3LTA2033/archcraft-full-upg) | ![lang](https://img.shields.io/github/languages/top/D3LTA2033/archcraft-full-upg?style=flat&labelColor=545454&color=313131&label=) | ![stars](https://img.shields.io/github/stars/D3LTA2033/archcraft-full-upg?style=flat&labelColor=545454&color=313131&label=) | ![commit](https://img.shields.io/github/last-commit/D3LTA2033/archcraft-full-upg?style=flat&labelColor=545454&color=313131&label=) | ![license](https://img.shields.io/github/license/D3LTA2033/archcraft-full-upg?style=flat&labelColor=545454&color=313131&label=) |
| [venoly-desktops](https://github.com/D3LTA2033/venoly-desktops) | ![lang](https://img.shields.io/github/languages/top/D3LTA2033/venoly-desktops?style=flat&labelColor=545454&color=313131&label=) | ![stars](https://img.shields.io/github/stars/D3LTA2033/venoly-desktops?style=flat&labelColor=545454&color=313131&label=) | ![commit](https://img.shields.io/github/last-commit/D3LTA2033/venoly-desktops?style=flat&labelColor=545454&color=313131&label=) | ![license](https://img.shields.io/github/license/D3LTA2033/venoly-desktops?style=flat&labelColor=545454&color=313131&label=) |

<img src="assets/divider.svg" width="100%" alt="">

## Build flags and scorecard

Strict flags where there is a toolchain to be strict with.

| Project | Toolchain and flags |
|:--|:--|
| **AstraOS** | `i686-elf-gcc` · `-m32 -std=c11 -ffreestanding -fno-builtin -nostdlib -Wall -Wextra -Werror -O2` · `nasm` · `grub-mkrescue` · `qemu-system-i386` |
| **SCSA** | `-O2 -Wall -Wextra -fstack-protector-strong -D_FORTIFY_SOURCE=2 -fPIE -pie -Wl,-z,relro,-z,now` → PIE ELF |
| **Aether** | `g++ -std=c++23 -Isrc -fsyntax-only` on each core unit · generated C goes through `gcc -O2` |
| **RESS** | `cargo check --workspace --all-targets` |

| Check | Result |
|:--|:--|
| AstraOS kernel translation units | 32 of 32 compile clean under `-Werror` |
| AstraOS userland programs | 3 of 3 compile clean |
| RESS workspace | 0 errors, 0 warnings across 10 crates |
| Aether core translation units | 8 of 8 pass `-fsyntax-only` |

<img src="assets/divider.svg" width="100%" alt="">

## Read before you run

Every project has sharp edges. These are the ones that matter, stated up front.

| Project | Know this first |
|:--|:--|
| **AstraOS** | Targets i686 only; needs `i686-elf-gcc` and `nasm`. Astracor is tree-walking with no closures. |
| **RESS** | Enforcement needs `CAP_SYS_ADMIN`. Start with `--observe` — it runs the full analysis and policy path without gating anything. |
| **Aether** | Four of the five optimizer passes are registered in the default pipeline. |
| **SCSA** | Statements end with `;` — a newline-separated program compiles to one instruction. `main()` auto-installs to `~/.local/bin` before parsing `argv`. |
| **secureforge** | Alpha/beta by its own warnings. Pass `--distro` explicitly and run it in a VM first. |
| **kittybrain** | Time-boxed to a 70-minute GPU window; it is a from-scratch exercise, not a product. |
| **Venoly desktop** | Closed source. Windows builds are not code-signed yet, so SmartScreen will warn. macOS is not shipped. |
| **Language stats** | Linguist bytes include some generated build output in Makefile and CMake. Use the by-commit card for the real mix. |

<img src="assets/divider.svg" width="100%" alt="">

## Shipping

<p>
<img alt="web" src="https://img.shields.io/website?url=https%3A%2F%2Fvenoly.nl&label=venoly.nl&up_message=online&down_message=offline&style=flat&labelColor=545454&up_color=2ea043">
<img alt="api" src="https://img.shields.io/website?url=https%3A%2F%2Fapi.venoly.nl%2Fhealth&label=api.venoly.nl&up_message=online&down_message=offline&style=flat&labelColor=545454&up_color=2ea043">
</p>

**[Venoly](https://venoly.nl)** — a security-first chat, voice/video and communities
platform I build and ship. Live web app, live API (`api.venoly.nl/health`), and
desktop builds for Windows and Linux. Shipped bundles measure 1.39 MB of app JS, a
dedicated 635 KB LiveKit chunk for WebRTC voice and video, 427 KB of React and 333 KB
of CSS; passkey auth via SimpleWebAuthn, realtime over socket.io, HSTS with a
two-year max-age and `X-Frame-Options: DENY`.

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#24283b','primaryTextColor':'#c0caf5','primaryBorderColor':'#7aa2f7','lineColor':'#7aa2f7'}}}%%
flowchart LR
  U["web app"] -->|"HTTPS · HSTS 2y"| API["api.venoly.nl"]
  D["desktop<br/>Windows · Linux"] --> API
  API --> RT["socket.io<br/>realtime"]
  API --> PK["passkeys<br/>SimpleWebAuthn"]
  U --> LK["LiveKit<br/>WebRTC voice + video"]
```

```
  shipped bundle sizes, measured over the wire

  app JS      1.39 MB  ████████████████████████████████████████
  LiveKit      635 KB  ██████████████████▎
  React        427 KB  ████████████▎
  CSS          333 KB  █████████▌
```

[venoly-desktops](https://github.com/D3LTA2033/venoly-desktops) ships four artifacts —
NSIS per-user installer, a portable `.exe` for locked-down machines, an AppImage and a
`.deb` — each with a line in `SHA256SUMS-1.0.0.txt`. Verify before you run:

```console
$ sha256sum -c SHA256SUMS-1.0.0.txt
$ chmod +x Venoly-1.0.0-x64.AppImage && ./Venoly-1.0.0-x64.AppImage
```

Closed source, checksummed. The Windows builds are not code-signed yet, so
SmartScreen will warn; macOS is not shipped.

<img src="assets/divider.svg" width="100%" alt="">

## Verify it yourself

<details>
<summary><b>Reproduce the numbers</b></summary>

<br>

The method behind every line count above. Adjust the extensions per repo — the point
is to count only hand-written source, with nothing vendored or generated:

```console
$ git clone https://github.com/D3LTA2033/<repo> && cd <repo>
$ git ls-files | grep -E '\.(c|h|cpp|hpp|rs|go|py|sh|js|asm)$' | xargs wc -l | tail -1
```

Using `git ls-files` keeps ignored and untracked output (build directories,
`node_modules`) out of the count.

</details>

<details>
<summary><b>Run each project's own check</b></summary>

<br>

```console
$ cargo check --workspace --all-targets            # RESS: zero errors, zero warnings
$ ./target/release/ress analyze /usr/bin/ls        # RESS: score a binary
$ node index.js --distro arch --list               # secureforge: list modules, change nothing
$ ./aether run examples/hello.ae                   # Aether: end-to-end build and run
$ ./build/scsa --run demo.scsa                     # SCSA: run a program
$ make iso && make run                             # AstraOS: boot it in QEMU
$ sha256sum -c SHA256SUMS-1.0.0.txt                # Venoly desktop: verify artifacts
```

</details>

<details>
<summary><b>Check the live product</b></summary>

<br>

```console
$ curl -sI https://venoly.nl | grep -iE 'strict-transport|x-frame'
$ curl -s https://api.venoly.nl/health
```

The headers to expect are HSTS with a two-year max-age and `X-Frame-Options: DENY`.

</details>

<img src="assets/divider.svg" width="100%" alt="">

## Activity

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://github-profile-summary-cards.vercel.app/api/cards/stats?username=D3LTA2033&theme=tokyonight">
  <source media="(prefers-color-scheme: light)" srcset="https://github-profile-summary-cards.vercel.app/api/cards/stats?username=D3LTA2033&theme=github">
  <img alt="Contribution statistics" src="https://github-profile-summary-cards.vercel.app/api/cards/stats?username=D3LTA2033&theme=tokyonight">
</picture>
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://github-profile-summary-cards.vercel.app/api/cards/profile-details?username=D3LTA2033&theme=tokyonight">
  <source media="(prefers-color-scheme: light)" srcset="https://github-profile-summary-cards.vercel.app/api/cards/profile-details?username=D3LTA2033&theme=github">
  <img alt="Profile details" src="https://github-profile-summary-cards.vercel.app/api/cards/profile-details?username=D3LTA2033&theme=tokyonight">
</picture>
</p>

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://github-profile-summary-cards.vercel.app/api/cards/productive-time?username=D3LTA2033&theme=tokyonight&utcOffset=1">
  <source media="(prefers-color-scheme: light)" srcset="https://github-profile-summary-cards.vercel.app/api/cards/productive-time?username=D3LTA2033&theme=github&utcOffset=1">
  <img alt="Productive time" src="https://github-profile-summary-cards.vercel.app/api/cards/productive-time?username=D3LTA2033&theme=tokyonight&utcOffset=1">
</picture>
</p>

<p align="center">
  <img alt="Contribution graph" src="https://github-readme-activity-graph.vercel.app/graph?username=D3LTA2033&bg_color=1a1b27&color=7aa2f7&line=7aa2f7&point=c0caf5&area=true&area_color=7aa2f7&hide_border=true&radius=8">
</p>

<p align="center">
  <img alt="Streak" src="https://streak-stats.demolab.com?user=D3LTA2033&theme=tokyonight&hide_border=true&background=00000000">
</p>

<p align="center">
  <img alt="Trophies" src="https://github-profile-trophy.vercel.app/?username=D3LTA2033&theme=tokyonight&no-frame=true&no-bg=true&row=1&column=7&margin-w=8">
</p>

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/D3LTA2033/D3LTA2033/output/github-snake-dark.svg">
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/D3LTA2033/D3LTA2033/output/github-snake.svg">
  <img alt="Contribution graph, eaten by a snake" src="https://raw.githubusercontent.com/D3LTA2033/D3LTA2033/output/github-snake.svg">
</picture>
</p>

<img src="assets/divider.svg" width="100%" alt="">

## Elsewhere

| | |
|:--|:--|
| **Site** | [d3lta2033.nl](https://d3lta2033.nl) |
| **Platform** | [venoly.nl](https://venoly.nl) |
| **Discord** | `@mcs.s` |

**Reading.**

- [portfolio/kernel_tut.md](https://github.com/D3LTA2033/D3LTA2033/blob/main/portfolio/kernel_tut.md)
  — Linux kernel internals study notes: paging, VFS, char and block drivers, the
  network stack, LSM, RCU, syscall entry. 12 chapters, 35 worked code blocks.
- [portfolio/portfolio.md](https://github.com/D3LTA2033/D3LTA2033/blob/main/portfolio/portfolio.md)
  · [portfolio_2.md](https://github.com/D3LTA2033/D3LTA2033/blob/main/portfolio/portfolio_2.md)
  · [ai_opinion.md](https://github.com/D3LTA2033/D3LTA2033/blob/main/portfolio/ai_opinion.md)

<div align="center">

<img alt="open to work" src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=600&size=17&duration=3200&pause=1500&color=BB9AF7&center=true&vCenter=true&width=520&height=36&lines=Open+to+low-level+and+security+work;Say+hi+on+Discord%3A+%40mcs.s;Thanks+for+reading+this+far">

<img width="100%" alt="footer" src="https://capsule-render.vercel.app/api?type=waving&height=120&color=0:bb9af7,50:3d59a1,100:1a1b27&section=footer">

</div>
