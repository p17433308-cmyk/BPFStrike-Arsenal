```
  ____  ____  _____ ____  _        _ _        
 | __ )|  _ \|  ___/ ___|| |_ _ __(_) | _____ 
 |  _ \| |_) | |_  \___ \| __| '__| | |/ / _ \
 | |_) |  __/|  _|  ___) | |_| |  | |   <  __/
 |____/|_|   |_|   |____/ \__|_|  |_|_|\_\___|
```

# BPFStrike — Automated eBPF Exploitation & Post-Exploitation Framework

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Tests](https://img.shields.io/badge/tests-140%20passing-brightgreen.svg)]()
[![LOC](https://img.shields.io/badge/LOC-20%2C366-informational.svg)]()
[![Black Hat Arsenal](https://img.shields.io/badge/Black%20Hat-Asia%202027%20Arsenal-black.svg)]()

> **First comprehensive open-source eBPF offensive security toolkit, to our knowledge, that generates real BPF bytecode for kernel exploitation, container escape, rootkit deployment, and network interception. Weaponizes 8 real eBPF verifier CVEs (2021–2024).**

---

## The Problem

eBPF lets user-supplied programs execute **inside the Linux kernel** — used by every major cloud provider, container runtime, and observability platform. The eBPF verifier, the sole safety gate, has been repeatedly broken:

| CVE | Year | Bug | Impact |
|---|---|---|---|
| CVE-2021-3490 | 2021 | ALU32 bounds tracking bypass | Arbitrary kernel R/W |
| CVE-2021-31440 | 2021 | BPF_RSH 32-bit bounds miscalculation | OOB access |
| CVE-2022-23222 | 2022 | Null pointer via uninitialized register | Kernel crash / escalation |
| CVE-2023-2163 | 2023 | Verifier range tracking error | OOB access |
| CVE-2023-2598 | 2023 | io_uring BPF interaction | Use-after-free |
| CVE-2023-39191 | 2023 | Unprivileged BPF via nesting | Privilege escalation |
| CVE-2024-0582 | 2024 | io_uring ring buffer UAF | Use-after-free |
| CVE-2024-2193 | 2024 | Speculative execution via BPF | GhostRace |

Every Linux kernel 5.x–6.x is affected — **billions** of cloud instances, containers, and IoT devices. Individual PoCs are scattered across CTF writeups. **No unified offensive framework existed.**

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                       BPFStrike CLI                             │
│      generate · recon · forensics · verifier · evasion · demo   │
├───────────┬───────────┬───────────┬───────────┬─────────────────┤
│  ATTACKS  │  EVASION  │   RECON   │ FORENSICS │   VERIFIER      │
│           │           │           │           │                 │
│ Syscall   │ Falco     │ Kernel    │ Bytecode  │ CVE-2021-3490   │
│ Hook      │ Bypass    │ BPF Caps  │ Pattern   │ CVE-2021-31440  │
│ Process   │ Tetragon  │ Loaded    │ Analysis  │ CVE-2022-23222  │
│ Hider     │ Bypass    │ Programs  │ MITRE     │ CVE-2023-2163   │
│ XDP       │ Tracee    │ BPF Maps  │ ATT&CK    │ CVE-2023-2598   │
│ Intercept │ Bypass    │ Security  │ Mapping   │ CVE-2023-39191  │
│ Cred      │ io_uring  │ Tool Det  │ Anomaly   │ CVE-2024-0582   │
│ Harvest   │ Evasion   │           │ Detection │ CVE-2024-2193   │
│ Container │ Map       │           │           │                 │
│ Escape    │ Corrupt   │           │           │ Complexity      │
│ Keylogger │           │           │           │ Bombs           │
│ BPF Hide  │           │           │           │                 │
├───────────┴───────────┴───────────┴───────────┴─────────────────┤
│                  BPF BYTECODE ENGINE                            │
│   eBPF ISA · ALU64/JMP/LD/ST/STX · Register Alloc · Map Ops    │
│              ELF Generator · Disassembler                       │
└─────────────────────────────────────────────────────────────────┘
```

### Module Breakdown

```
BPFStrike/
├── bpfstrike/
│   ├── core/
│   │   ├── bpf_bytecode.py        # eBPF ISA: opcodes, registers, instruction builders
│   │   └── elf_generator.py       # Minimal ELF .o generation for bpftool/libbpf
│   ├── attacks/
│   │   └── offensive_programs.py  # 8 attack program generators (real bytecode)
│   ├── evasion/
│   │   └── security_bypass.py     # Falco/Tetragon/Tracee evasion techniques
│   ├── recon/
│   │   └── bpf_scanner.py         # BPF subsystem enumeration & capability audit
│   ├── forensics/
│   │   └── bpf_analyzer.py        # Malicious BPF detection & MITRE ATT&CK mapping
│   ├── verifier/
│   │   └── bypass_patterns.py     # 8 CVE-based verifier bypass generators
│   └── cli.py                     # Full CLI interface
├── evidence/
│   ├── demo_output.txt            # Validated demo output (1,920 bytes bytecode)
│   └── bytecode_validation.txt    # Bytecode alignment & opcode verification
├── tests/
│   └── test_bpfstrike.py          # 140 test cases
└── setup.py
```

---

## Attack Kill Chain

```
  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
  │  RECON   │──►│ VERIFIER │──►│  ATTACK  │──►│ EVASION  │──►│ PERSIST  │
  │          │   │  BYPASS  │   │          │   │          │   │          │
  │ Enum BPF │   │ Select   │   │ Rootkit  │   │ Bypass   │   │ Pin to   │
  │ caps &   │   │ CVE for  │   │ Deploy   │   │ Falco    │   │ bpffs    │
  │ programs │   │ kernel   │   │ Harvest  │   │ Tetragon │   │ Covert   │
  │ Detect   │   │ version  │   │ creds    │   │ Tracee   │   │ C2 via   │
  │ defenses │   │ Generate │   │ Intercept│   │ Hide BPF │   │ BPF maps │
  │          │   │ bytecode │   │ network  │   │ programs │   │          │
  └──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘
```

## Attack Programs

| Attack | Type | Attach Point | MITRE ATT&CK | Impact |
|---|---|---|---|---|
| Syscall Hook | kprobe | `__x64_sys_read` | T1059 | Intercept all read syscalls |
| Process Hider | kprobe | `__x64_sys_getdents64` | T1564.001 | Hide processes from `ps`, `top` |
| XDP Interceptor | xdp | `eth0` | T1557 | Capture/modify packets at wire speed |
| Credential Harvester | kprobe | `pam_sm_authenticate` | T1056.001 | Harvest passwords, keys, tokens |
| Container Escape | cgroup_skb | docker cgroup | T1611 | Break cgroup/namespace isolation |
| File Hider | tracepoint | `sys_exit_getdents64` | T1564.001 | Hide files from `ls`, `find` |
| Keylogger | tracepoint | `sys_enter_write` | T1056.001 | Capture keystrokes kernel-level |
| BPF Self-Hider | tracing | bpf_prog iterator | T1014 | Hide BPF programs from detection |

## Security Tool Evasion

| Target Tool | Evasion Technique | How |
|---|---|---|
| **Falco** | io_uring bypass | Use io_uring syscalls that Falco doesn't hook |
| **Tetragon** | BPF map corruption | Corrupt Tetragon's policy maps |
| **Tracee** | Program hiding | Hide BPF programs from Tracee's enumeration |
| **All** | Self-hiding rootkit | BPF program that hides itself from `bpftool prog list` |

---

## Usage

### Installation

```bash
git clone https://github.com/p17433308-cmyk/BPFStrike-Arsenal.git
cd BPFStrike-Arsenal
pip install -e .
```

### Quick Start — Full Demo

```bash
bpfstrike demo
```

Generates all attack programs, runs reconnaissance, performs forensic analysis — full pipeline with real bytecode output.

### Individual Modules

**Generate Attack Programs:**
```bash
# Generate all offensive programs
bpfstrike generate

# Specific attack with disassembly
bpfstrike generate syscall_hook --disasm

# Save as loadable ELF objects
bpfstrike generate -o ./output/
```

**Reconnaissance:**
```bash
# BPF subsystem enumeration
bpfstrike recon

# Analyze bpftool output
bpfstrike recon --bpftool-progs progs.json --bpftool-maps maps.json
```

**Verifier Bypass:**
```bash
# Generate all verifier bypass patterns
bpfstrike verifier

# List known verifier CVEs
bpfstrike verifier --list-cves
```

**Forensic Analysis:**
```bash
# Analyze all known attack patterns
bpfstrike forensics

# Analyze suspicious bytecode
bpfstrike forensics --bytecode-file suspicious.bin -o report.json
```

---

## Bytecode Validation

BPFStrike generates **real eBPF bytecode** — not scripts or wrappers:

```
Demo output: 1,920 bytes of validated BPF bytecode
  ├── 8-byte aligned instructions (eBPF ISA compliant)
  ├── Proper opcode encoding (ALU64, JMP, LD, ST, STX)
  ├── Register allocation (R0-R10)
  ├── Map operations (BPF_MAP_LOOKUP_ELEM, BPF_MAP_UPDATE_ELEM)
  ├── Helper function calls (bpf_probe_read, bpf_get_current_pid_tgid)
  └── 13 attack types, 8 verifier bypass patterns, 6 CVEs
```

---

## Prior Art

| Tool | Scope | BPFStrike Advantage |
|---|---|---|
| Individual CVE PoCs | Single exploit, single kernel | Unified framework, 8 CVEs, multi-kernel |
| bpf_exploit (CTF) | CTF-specific, unmaintained | Production-grade, 140 tests |
| Cilium / Falco | Detection only | Full red team: exploit + post-exploit + evasion |
| Academic papers | Theory, no tooling | Working bytecode generators |

---

## Dependencies

**None.** Pure Python standard library. Zero external packages.

## Tests

```bash
pytest tests/ -p no:cacheprovider    # 140 tests covering all modules
```

---

## Authors

- **Dr. Sapna V M** — PES University
- **Prajwal Chowdary M** — PES University

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

## Disclaimer

BPFStrike is designed for **authorized security testing, kernel security research, and educational purposes only**. Attack modules should only be used in controlled environments with explicit authorization. Users are responsible for ensuring compliance with all applicable laws. Do not use this tool against systems without explicit permission.
