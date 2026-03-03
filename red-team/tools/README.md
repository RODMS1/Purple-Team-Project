# pentest_scanner.py — Custom Network & Pen Test Tool

Custom Python tool developed by **The Bugs** for ICC-003 purple team operations.
Wraps multiple scanning tools into a single workflow and generates a structured `.txt` report including discovered CVEs.

---

## Requirements

The following tools must be installed and available in `$PATH`:

```
nmap
whatweb
sqlmap
```

Install on Debian/Ubuntu:
```bash
sudo apt install nmap sqlmap && gem install whatweb
```

---

## Usage

```bash
python pentest_scanner.py
```

The tool will prompt for a target IP or hostname, then present a scan menu.

**Validated input formats:**
- IPv4 address — e.g. `10.0.0.40`
- Hostname — e.g. `target.corp.org`

---

## Scan Options

| Option | Description |
|--------|-------------|
| Full Scan | Runs all modules sequentially and saves a `.txt` report |
| Port Scan | Nmap SYN/service scan |
| Web Scan | WhatWeb technology fingerprinting |
| SQL Injection | SQLMap automated injection test |

---

## Report Output

Full Scan generates a timestamped file:

```
scan_<host>_<YYYYMMDD_HHMMSS>.txt
```

**Report structure:**
```
============================================================
  NETWORK PENTEST TOOL — Full Scan Report
  Target  : <host>
  Date    : <timestamp>
  By The Bugs
============================================================

[Nmap output]
[WhatWeb output]
[SQLMap output]
[CVE findings]
```

---

## How It Was Used in ICC-003

The tool was run against all in-scope hosts in `10.0.0.0/24` during the initial enumeration phase (RTO1/RTO2). Full scan reports were generated for each discovered host and reviewed for CVE matches.

See [RTO3_custom-tooling.md](../objectives/RTO3_custom-tooling.md) for detailed usage notes from the exercise.
