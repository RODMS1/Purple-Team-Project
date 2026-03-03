# Phase 1 — Red Team (Black Box)

**Team:** The Bugs
**Exercise:** ICC-003 · CyberCore Purple Team
**Scope:** `10.0.0.0/24`
**Initial Access:** `10.0.0.219` (credentials provided separately)

---

## Objectives

| ID | Objective | Status |
|----|-----------|--------|
| RTO1 | Network enumeration + topology diagram | Complete |
| RTO2 | Vulnerability discovery (≥1 web app) with CWE + CVSS | Complete |
| RTO3 | Custom Python tool (documented + in repo) | Complete |
| RTO4 | Exploitation, lateral movement, privilege escalation | Complete |

---

## Out-of-Scope Hosts

The following hosts must **not** be targeted under any circumstances:

- `10.0.0.23`
- `10.0.0.100`
- `10.0.0.200`
- `10.0.0.176`

---

## Rules of Engagement — Prohibited Actions

- Denial-of-Service (DoS)
- RDP brute-force
- Attacking hypervisor or cloud control plane
- Attacking OpenVPN or security tooling
- Modifying firewall or port configurations
- Performing OS or software updates
- Deleting or corrupting data
- Disrupting availability of any service

> Reboots must be reported to White Team immediately.
> All activities must comply with the AWS Penetration Testing Policy.

---

## Tools Used

| Tool | Category | Purpose |
|------|----------|---------|
| Nmap | Recon | Host discovery, port scanning, service fingerprinting |
| WhatWeb | Recon | Web application technology identification |
| SQLMap | Exploitation | SQL injection detection and exploitation |
| LinPEAS | Post-exploitation | Linux privilege escalation enumeration |
| pentest_scanner.py | Custom | Integrated scanning + CVE report generation |

---

## Navigation

| Section | Description |
|---------|-------------|
| [tools/](tools/) | Custom Python scanner tool + usage guide |
| [objectives/RTO1_network-enumeration.md](objectives/RTO1_network-enumeration.md) | Host discovery + topology |
| [objectives/RTO2_vulnerability-discovery.md](objectives/RTO2_vulnerability-discovery.md) | CVE findings with CVSS scores |
| [objectives/RTO3_custom-tooling.md](objectives/RTO3_custom-tooling.md) | Custom tool documentation |
| [objectives/RTO4_exploitation.md](objectives/RTO4_exploitation.md) | Exploitation + lateral movement chain |
| [evidence/](evidence/) | Screenshots and raw scan outputs |
