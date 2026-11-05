# Advanced Software Security

Coursework and lab materials for the **Advanced Software Security** master's subject. The
repository collects the practical seminars, each exploring a different area of offensive and
defensive security through hands-on exercises, with written write-ups of the results.

## Repository structure

```
AdvancedSecurity/
├── seminar1/      Buffer overflow analysis and remediation (C / Visual Studio)
├── seminar2/      Network reconnaissance and traffic analysis (nmap, Wireshark, nc, ...)
└── Advanced-Security/   Supplementary material (git submodule)
```

## Seminars

### Seminar 1 — Memory safety: buffer overflows

A small client/server application (`SecuritateSoftware-v4`) written in C and built with Visual
Studio is studied to understand how an unbounded string copy in a login handler leads to a
stack-based buffer overflow.

The exercise walks through the full defensive lifecycle:

- **Root-cause analysis** of the vulnerability via code review and debugging.
- **Confirmation** that the flaw is server-side and reachable from the network.
- **Remediation** — replacing the unsafe copy with a bounded, length-checked alternative and
  re-enabling compiler and OS hardening (stack cookies, etc.) as defense in depth.
- **Validation** that over-long input is now rejected while a normal login still succeeds.

Key files:

- [`seminar1/requirement.txt`](seminar1/requirement.txt) — the assignment brief.
- [`seminar1/answer.md`](seminar1/answer.md) — the write-up of the analysis and fix.
- [`seminar1/SecuritateSoftware-v4/`](seminar1/SecuritateSoftware-v4/) — the sample application
  (client, server, shared library, and Visual Studio solution).

### Seminar 2 — Network reconnaissance and traffic analysis

A set of ten practical exercises covering host discovery, service enumeration, packet capture,
and protocol inspection in an isolated lab network. Topics include:

- Host discovery and live-host listing with `nmap`.
- Crafting and serving raw HTTP responses with `nc`, and fetching them with `wget`.
- File transfer between hosts over `nc`.
- Capturing DHCP exchanges and TCP handshakes in Wireshark.
- Inspecting HTTP cookies and reasoning about HTTPS confidentiality.
- Understanding a low-level `tcpdump`/BPF HTTP `POST` filter.
- Managing directories over FTP.
- Service/version detection and open-port extraction.
- Comparing TCP (SYN) and UDP scans and their response semantics.

Key files:

- [`seminar2/seminar2.md`](seminar2/seminar2.md) — solutions and explanations for all ten
  exercises.
- [`seminar2/seminar_02.pdf`](seminar2/seminar_02.pdf) — the original exercise sheet.

Captured evidence / command output:

- [`seminar2/ex4.png`](seminar2/ex4.png) — Wireshark DHCP capture (exercise 4).
- [`seminar2/ex4-handshake.png`](seminar2/ex4-handshake.png) — Wireshark TCP three-way
  handshake, via Follow → TCP Stream (exercise 4).
- [`seminar2/live-hosts.lst`](seminar2/live-hosts.lst) — live-host sweep output (exercise 1).
- [`seminar2/scan.txt`](seminar2/scan.txt) — full `nmap -sV -p-` report (exercise 8).
- [`seminar2/ex8-open-ports.txt`](seminar2/ex8-open-ports.txt) — extracted open ports and
  service versions (exercise 8).
- [`seminar2/ex9-capture.txt`](seminar2/ex9-capture.txt) — `tcpdump` capture of the TCP/UDP
  scans (exercise 9).
- [`seminar2/ex5-cookies.txt`](seminar2/ex5-cookies.txt) — `tshark` capture of HTTP cookies in
  GET requests (exercise 5).

## Scope and intent

All material here is produced for an academic course and is meant for **authorized, educational
use in an isolated lab environment**. The exercises emphasize understanding vulnerabilities in
order to detect, fix, and defend against them responsibly.
