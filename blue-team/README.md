# Phase 2 — SOC Visibility & Detection Engineering

**Team:** The Bugs
**Exercise:** ICC-003 · CyberCore Purple Team
**Focus:** Analyze Phase 1 telemetry, map to MITRE ATT&CK, build detections, identify gaps.

---

## Objectives

| ID | Objective | Status |
|----|-----------|--------|
| BTO1 | Analyze logs and telemetry generated during Phase 1 | Complete |
| BTO2 | Identify evidence of adversarial actions in log sources | Complete |
| BTO3 | Create detections and alerts based on observed behavior | Complete |
| BTO4 | Validate which attacks were visible and which were not | Complete |
| BTO5 | Explain visibility gaps explicitly | Complete |

---

## Overall Verdict

> **FULL DOMAIN COMPROMISE — CORP.ORG**
>
> Red team achieved Domain Administrator via **FTP credential theft → RDP login → LSASS dump → hash crack**.
> Apache CVE-2021-41773 provides an independent RCE path.
> **Risk Rating: CRITICAL**

---

## Detection Summary

| Event ID | Event Name | Detected | Severity |
|----------|------------|----------|----------|
| 1102 | Security Log Cleared | Yes | Critical |
| 4625 | Failed Logon Attempt | Yes | High |
| 4624 | Successful Network Logon (Type 3) | Yes | High |
| 1149 | RDP Authentication Success | Yes | Medium |

**4 of 6 attack phases produced detectable telemetry.**

---

## Log Sources Analyzed

| Source | Data Type | Key Findings |
|--------|-----------|-------------|
| Windows Security Event Log | Authentication, account activity | Events 4624, 4625, 1102 |
| TerminalServices-RemoteConnectionManager | RDP authentication | Event 1149 |
| FileZilla FTP Server Logs | FTP sessions | Session 1110 — anonymous login + file download |
| Network PCAP | Raw traffic | FTP command stream, Apache RCE payload |

---

## Visibility Gaps

| Attack Phase | Visible? | Gap |
|-------------|----------|-----|
| Anonymous FTP enumeration | Partial | FTP session logged, but no alert configured |
| RDP credential use from FTP file | Yes | Event 1149 captured |
| LSASS dump | No | No Sysmon/EDR installed on DESKTOP-RCUCBMP |
| Hash cracking (offline) | No | Inherently undetectable from logs alone |
| Domain Admin authentication | Not confirmed | Log cleared before analysis (Event 1102) |
| Apache CVE-2021-41773 RCE | Yes | Payload captured in PCAP |

---

## Navigation

| Section | Description |
|---------|-------------|
| [dashboard/index.html](dashboard/index.html) | Interactive IR dashboard (open in browser) |
| [detections/windows-event-analysis.md](detections/windows-event-analysis.md) | Windows Event Log deep-dive (1102, 4625, 4624, 1149) |
| [detections/ftp-forensics.md](detections/ftp-forensics.md) | FTP Session 1110 packet-by-packet analysis |
| [detections/network-forensics.md](detections/network-forensics.md) | PCAP analysis — Apache RCE + network events |
| [mitre-attack-mapping.md](mitre-attack-mapping.md) | Full MITRE ATT&CK technique mapping |
