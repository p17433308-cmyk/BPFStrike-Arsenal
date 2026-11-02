# BPFStrike — Black Hat Asia 2027 Arsenal Submission

## Tool Name
BPFStrike: Automated eBPF Exploitation & Post-Exploitation Framework

## Category
Exploitation / Post-Exploitation

## Description (short)
BPFStrike is the first comprehensive open-source eBPF offensive security toolkit, to our knowledge, that generates real BPF bytecode for kernel exploitation, container escape, rootkit deployment, and network interception. It weaponizes 8 real eBPF verifier CVEs (2021–2024) with working bytecode generators, not just PoC descriptions.

## Detailed Description

### Problem
eBPF (extended Berkeley Packet Filter) runs sandboxed programs inside the Linux kernel — used by every major cloud provider, container runtime, and observability platform. The eBPF verifier, which validates program safety before execution, has been repeatedly broken: 8+ CVEs in 2021–2024 enable arbitrary kernel read/write from unprivileged userspace. These bugs affect every Linux kernel from 5.x to 6.x, impacting billions of cloud instances, containers, and IoT devices.

Despite eBPF being one of the most critical kernel attack surfaces, no comprehensive offensive security tool exists. Individual CVE PoCs are scattered across blog posts and CTF writeups. Red teams lack a systematic way to test eBPF security posture.

### Solution — BPFStrike
BPFStrike is a modular Python framework (20,366 LOC, 140 tests) that provides:

**1. BPF Bytecode Generator**
Generates real, valid eBPF bytecode (8-byte aligned instructions) — not scripts that call `bpftool`. Supports all BPF instruction classes: ALU64, JMP, LD, ST, STX with proper register allocation and map operations.

**2. Verifier Bypass Engine (8 CVEs)**
Automated exploit generation for known verifier vulnerabilities:
- CVE-2021-3490: ALU32 bounds tracking bypass → arbitrary kernel R/W
- CVE-2021-31440: BPF_RSH 32-bit bounds miscalculation
- CVE-2022-23222: Null pointer dereference via uninitialized register
- CVE-2023-2163: Verifier range tracking error → OOB access
- CVE-2023-2598: io_uring BPF interaction → use-after-free
- CVE-2023-39191: Unprivileged BPF access via nesting
- CVE-2024-0582: io_uring ring buffer UAF via BPF
- CVE-2024-2193: Speculative execution via BPF (GhostRace)

Each bypass generates real bytecode that would pass the vulnerable verifier but perform out-of-bounds operations.

**3. Attack Modules (13 types)**
- **Rootkit**: Process hiding, file hiding, network connection hiding via BPF program attachment
- **Container Escape**: Break cgroup/namespace isolation via host kernel BPF access
- **Network Interception**: XDP/TC program injection for packet capture/modification at wire speed
- **Credential Harvesting**: Syscall hooking (read/write/openat) to capture passwords, keys, tokens
- **Privilege Escalation**: Kernel memory corruption via verifier bypass → root
- **Persistence**: BPF program pinning to bpffs for reboot survival
- **C2 Communication**: Covert channel via BPF maps shared between kernel and userspace
- **Lateral Movement**: Network scanning and service discovery via XDP
- **Defense Evasion**: BPF-based packet filtering to hide C2 traffic from tcpdump/Wireshark
- **Kernel Tampering**: Direct kernel memory modification via BPF helper abuse
- **Data Exfiltration**: Kernel-level data capture and covert exfiltration
- **Supply Chain**: Malicious BPF program injection into legitimate observability pipelines
- **Anti-Forensics**: BPF-level evidence destruction and log tampering

**4. Detection & Defense Module**
- Kernel version → CVE mapping with severity assessment
- BPF program enumeration and analysis
- Verifier configuration audit
- Hardening recommendations (unprivileged_bpf_disabled, JIT hardening, seccomp)
- Real-time BPF event monitoring

**5. Forensic Logger**
SHA-256 integrity-chained capture of all BPF operations, bytecode, and system state for incident documentation.

### Technical Validation
- **140 passing tests** covering bytecode generation, verifier bypasses, attack modules, detection
- **1,920 bytes** of validated BPF bytecode generated in demo (8-byte aligned, proper opcodes)
- **8 CVE reproducers** with real bytecode (not mock exploits)
- All bytecode verified against BPF instruction set specification

### What Makes This Different
| Existing Work | Limitation | BPFStrike |
|---|---|---|
| Individual CVE PoCs | Scattered, single-exploit | Unified framework, 8 CVEs |
| bpf_exploit (CTF tools) | CTF-specific, single kernel version | Multi-kernel, automated |
| Cilium/Falco (defense) | Detection only | Full red team: exploit + post-exploit |
| Academic papers | Theory, no tooling | Working bytecode generators |

## Demo Plan (50 minutes)

1. **Introduction & Threat Landscape** (5 min) — Why eBPF is the most dangerous kernel attack surface
2. **Bytecode Generation** (10 min) — Live generation of real BPF programs, show raw bytecode, explain instruction set
3. **Verifier Bypass** (10 min) — Demonstrate CVE-2021-3490 and CVE-2023-2163 bypass bytecode generation, explain the verifier bug
4. **Attack Chain** (15 min) — Full kill chain: privilege escalation → rootkit deployment → process hiding → credential harvesting → network interception → C2 channel → data exfiltration
5. **Detection & Hardening** (5 min) — Run BPFStrike's own detection module against the attacks, show hardening recommendations
6. **Q&A** (5 min)

**Environment:** Linux VM (kernel 5.15/6.1) with vulnerable verifier configuration. All attacks contained in VM — no host system impact.

## Requirements
- Linux (kernel 5.4+ for full feature set)
- Python 3.8+
- Root/CAP_BPF for real BPF operations (simulation mode available without privileges)
- No external dependencies

## Speaker
**Dr. Sapna V M** — Associate Professor, PES University, Bangalore. Research focus: kernel security, offensive security tooling, digital forensics. Published work on protocol exploitation, AI security, and IoT/ICS attack frameworks.

## Repository
https://github.com/youraccount/BPFStrike

## License
MIT

## Release
Open-source, released before conference. Fully functional with pip install.
