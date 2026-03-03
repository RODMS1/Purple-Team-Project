# MITRE ATT&CK Mapping — ICC-003

**Exercise:** ICC-003 · CyberCore Purple Team
**Framework:** MITRE ATT&CK v14 (Enterprise)

---

## Technique Matrix

| Tactic | Technique ID | Technique Name | Used By Red | Detected | Detection Source |
|--------|-------------|----------------|-------------|----------|-----------------|
| Reconnaissance | T1046 | Network Service Discovery | Yes | Partial | Network traffic anomalies |
| Reconnaissance | T1018 | Remote System Discovery | Yes | No | No host-based logging |
| Initial Access | T1190 | Exploit Public-Facing Application | Yes | Yes | PCAP — Apache CVE-2021-41773 |
| Initial Access | T1078 | Valid Accounts (via stolen creds) | Yes | Yes | Event 4625 / 4624 |
| Execution | T1059 | Command and Scripting Interpreter | Yes | No | No Sysmon on target |
| Persistence | — | — | Not observed | — | — |
| Privilege Escalation | T1003.001 | OS Credential Dumping: LSASS Memory | Yes | No | No EDR on target |
| Defense Evasion | T1070.001 | Indicator Removal: Clear Windows Event Logs | Yes | Yes | Event 1102 |
| Credential Access | T1110.002 | Brute Force: Password Cracking | Yes | No | Offline — no logs |
| Discovery | T1083 | File and Directory Discovery (FTP) | Yes | Yes | FTP Session 1110 (PCAP + FTP logs) |
| Discovery | T1518 | Software Discovery | Yes | No | No logging on web server |
| Lateral Movement | T1021.001 | Remote Services: RDP | Yes | Yes | Event 1149 |
| Lateral Movement | T1021.002 | Remote Services: SMB/Windows Admin Shares | Yes | Partial | Event 4624 + PCAP |
| Collection | T1005 | Data from Local System | Yes | Yes | FTP RETR in PCAP |
| Exfiltration | T1048 | Exfiltration Over Alternative Protocol (FTP) | Yes | Yes | PCAP Session 1110 |

---

## Detection Coverage

```
Detected:     ████████░░░░░░  6 / 14 techniques  (43%)
Not Detected: ░░░░░░░░████░░  8 / 14 techniques  (57%)
```

---

## Visibility by Tactic

| Tactic | Techniques Used | Detected | Coverage |
|--------|----------------|----------|----------|
| Reconnaissance | 2 | 0 | 0% |
| Initial Access | 2 | 2 | 100% |
| Execution | 1 | 0 | 0% |
| Privilege Escalation | 1 | 0 | 0% |
| Defense Evasion | 1 | 1 | 100% |
| Credential Access | 1 | 0 | 0% |
| Discovery | 2 | 1 | 50% |
| Lateral Movement | 2 | 2 | 100% |
| Collection | 1 | 1 | 100% |
| Exfiltration | 1 | 1 | 100% |

---

## Key Detection Gaps

### 1. No Endpoint Detection (EDR / Sysmon)
- LSASS dump (T1003.001) — the highest-impact action — generated **zero detectable telemetry**
- Process execution on compromised hosts (T1059) was completely invisible
- **Fix:** Deploy Sysmon with EventID 10 (ProcessAccess) monitoring on all Windows hosts; deploy EDR agent

### 2. No Alerting on Recon Traffic
- Nmap scans and WhatWeb fingerprinting were not flagged
- **Fix:** Deploy NIDS (Suricata/Snort) with rules for port scan detection

### 3. Offline Credential Cracking Undetectable
- Hash cracking (T1110.002) occurs on the attacker's machine with no network footprint
- **Fix:** Enforce Kerberoasting-resistant account configurations; monitor for unusual authentication from newly-seen IPs

### 4. FTP Session Had No Alert
- Anonymous FTP access was logged but no alert was triggered until post-incident PCAP review
- **Fix:** SIEM rule: alert on any anonymous FTP authentication event; alert on `RETR` commands in anonymous sessions

---

## Detected Techniques — Evidence Summary

| Technique | Evidence |
|-----------|---------|
| T1190 — Apache RCE | HTTP payload in PCAP — URL-encoded path traversal |
| T1078 — Valid Accounts | Events 4625 + 4624 — failed then successful logon from same IP |
| T1070.001 — Log Clear | Event 1102 — 24,362 events deleted from DC |
| T1083 — File Discovery | FTP Session 1110 — LIST + NLST commands captured |
| T1021.001 — RDP | Event 1149 — TerminalServices-RemoteConnectionManager |
| T1021.002 — SMB | Event 4624 (Logon Type 3) + SMB traffic in PCAP |
| T1005 — Local Data | FTP `RETR rdp-settings.txt` captured in PCAP |
| T1048 — Alt Protocol Exfil | FTP session — cleartext exfil in PCAP |
