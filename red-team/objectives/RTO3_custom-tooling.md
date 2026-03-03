# RTO3 — Custom Tooling

**Objective:** Develop at least one custom Python tool to support offensive operations. The tool must be documented and included in the repository.

---

## Tool: pentest_scanner.py

**Location:** [`tools/pentest_scanner.py`](../tools/pentest_scanner.py)
**Language:** Python 3
**Usage Guide:** [`tools/README.md`](../tools/README.md)

---

## Description

`pentest_scanner.py` is a unified offensive reconnaissance and scanning tool built for this exercise. It integrates multiple industry-standard tools (Nmap, WhatWeb, SQLMap) into a single interactive interface and generates structured `.txt` reports.

**Key features:**
- Validates target input (IPv4 or hostname) before running any scans — prevents accidental targeting of out-of-scope systems
- Wraps Nmap, WhatWeb, and SQLMap in a menu-driven interface
- Strips ANSI escape codes from tool output for clean report formatting
- Auto-generates timestamped reports with a consistent header (target, date, team attribution)
- Checks for required tool dependencies at runtime and exits gracefully if missing

---

## How It Was Used

| Phase | Usage |
|-------|-------|
| RTO1 — Enumeration | Full Scan run against all discovered in-scope hosts to generate baseline reports |
| RTO2 — Vulnerability Discovery | SQLMap module used to test web applications for injection vulnerabilities |
| RTO2 — Vulnerability Discovery | WhatWeb module used to fingerprint web server technologies and identify version-specific CVEs |

---

## Design Decisions

- **Input validation** (`is_valid_host`) prevents typos from scanning out-of-scope IPs
- **`require_tool()`** checks that each dependency is installed before running, providing clear installation instructions
- **`save_report()`** writes ANSI-stripped output to `.txt` for portability and inclusion in evidence packages
- The tool was kept intentionally minimal — purpose-built for this engagement rather than a general-purpose framework

---

## Sample Report Header

```
============================================================
  NETWORK PENTEST TOOL — Full Scan Report
  Target  : 10.0.0.40
  Date    : 2026-02-13 09:14:22
  By The Bugs
============================================================
```
