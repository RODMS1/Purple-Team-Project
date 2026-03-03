# ICC-003 — Purple Team Security Exercise

**Team:** The Bugs
**Environment:** CyberCore · corp.org · AWS · `10.0.0.0/24`
**Result:** Full Domain Compromise — CRITICAL

Purple team simulation: Red Team executes a black-box engagement, then Blue Team analyzes telemetry to assess detection coverage and identify blind spots.

---

## Quick Navigation

| Section | Description |
|---------|-------------|
| [red-team/](red-team/) | Phase 1 — offensive operations, tools, findings |
| [blue-team/](blue-team/) | Phase 2 — detection engineering, event analysis, MITRE mapping |
| [purple-team/](purple-team/) | Combined synthesis, kill chain, remediation priorities |
| [docs/](docs/) | Assignment brief (ICC-003 PDF) |

---

## Exercise Overview

| Field | Value |
|-------|-------|
| **Type** | Purple Team (Red executes → Blue analyzes) |
| **Scope** | `10.0.0.0/24` |
| **Initial Access** | `10.0.0.219` (credentials provided separately) |
| **Result** | Full Active Directory domain compromise |
| **Overall Risk** | CRITICAL |

**Out-of-scope hosts (do not target):** `10.0.0.23` · `10.0.0.100` · `10.0.0.200` · `10.0.0.176`

---

## Phase 1 — Red Team

Objectives: Network enumeration, vulnerability discovery, custom tooling, exploitation + lateral movement.

**Key outcomes:**
- 6 of 9 in-scope hosts affected
- 2 critical, 2 high, 3 medium vulnerabilities discovered
- Full Domain Administrator access achieved via: **FTP credential theft → RDP → LSASS dump → hash crack**
- Independent RCE via **Apache CVE-2021-41773** (CVSS 9.8)
- Custom scanner tool built and deployed ([pentest_scanner.py](red-team/tools/pentest_scanner.py))

See [red-team/](red-team/) for full details.

---

## Phase 2 — Blue Team

Objectives: Analyze Phase 1 telemetry, map to MITRE ATT&CK, identify detection gaps.

**Key outcomes:**
- 4 Windows Events detected: `1102` · `4625` · `4624` · `1149`
- FTP credential exfiltration and Apache RCE captured in PCAP
- 43% of attack techniques detected (6 of 14)
- Critical gap: LSASS dump (highest-impact action) generated **zero detectable events**

See [blue-team/](blue-team/) for full details and [blue-team/dashboard/](blue-team/dashboard/) for the interactive IR dashboard.

---

## Rules of Engagement — Prohibited

- DoS  
- RDP brute-force  
- Attacking hypervisor/cloud control plane  
- Attacking OpenVPN/security tooling  
- Changing firewall/port configs  
- OS/software updates  
- Data deletion/corruption  
- Disrupting availability  

Reboots → contact White Team.  
Follow AWS Penetration Testing Policy.

## Phase 1 – Red Team Objectives

1. Network enumeration + topology diagram  
2. Vulnerability discovery (≥1 web app) → CWE + CVSS  
3. ≥1 custom Python tool (documented in repo)  
4. Exploitation, lateral movement, privesc (document failures)

## Phase 2 – Detection Objectives

- Analyze Phase 1 telemetry  
- Build detections/alerts  
- Map to MITRE ATT&CK  
- Document visible vs. blind activity + log sources + queries  
- Explain gaps explicitly


