# RTO1 — Network Enumeration

**Objective:** Discover all live hosts, open services, and exposed applications within the `10.0.0.0/24` scope. Document tools, techniques, and produce a network topology diagram.

---

## Tools Used

| Tool | Command | Purpose |
|------|---------|---------|
| Nmap | `nmap -sV -sC -O 10.0.0.0/24` | Host discovery + service fingerprinting |
| Nmap | `nmap -p- --min-rate 5000 <host>` | Full port sweep on confirmed hosts |
| WhatWeb | `whatweb http://<host>` | Web technology fingerprinting |
| pentest_scanner.py | Full Scan | Automated enumeration + report generation |

---

## Discovered Hosts (In-Scope)

| IP Address | Hostname | OS | Key Services |
|------------|----------|----|-------------|
| 10.0.0.40 | — | Windows Server | FTP (21), SMB (445), RDP (3389) |
| 10.0.0.119 | DESKTOP-RCUCBMP | Windows 10 | RDP (3389), SMB (445) |
| 10.0.0.219 | IP-10-0-0-219 | Linux | SSH (22), HTTP (80) |
| 10.0.0.x | EC2AMAZ-R2T6KBN | Windows Server (DC) | LDAP (389), SMB (445), RDP (3389) |

> **Note:** A full topology diagram is located in `evidence/screenshots/`. Raw Nmap outputs are in `evidence/scan-outputs/`.

---

## Key Findings

- **Anonymous FTP** enabled on `10.0.0.40` (FileZilla Server 1.8.2) — directory listing available without credentials
- **RDP exposed** on multiple hosts — `10.0.0.119` and the Domain Controller
- **Apache web server** on `10.0.0.219` — version disclosure, later confirmed vulnerable to CVE-2021-41773
- **Domain Controller identified** — Active Directory environment (corp.org)
- **SMB shares** accessible for lateral movement post-authentication

---

## Techniques

| Technique | MITRE ID | Description |
|-----------|----------|-------------|
| Network Service Discovery | T1046 | Nmap SYN scan across /24 |
| Remote System Discovery | T1018 | Host discovery via ICMP + TCP probes |
| Software Discovery | T1518 | WhatWeb web tech fingerprinting |
