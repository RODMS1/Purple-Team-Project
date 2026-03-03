# Purple Team Synthesis — ICC-003

**Team:** The Bugs
**Exercise:** ICC-003 · CyberCore Purple Team
**Environment:** corp.org · 10.0.0.0/24 · AWS-hosted
**Overall Risk Rating:** CRITICAL — Full Domain Compromise

---

## Exercise Summary

The Bugs conducted a two-phase purple team exercise against the corp.org Active Directory environment. Phase 1 (Red Team) executed a full black-box engagement, achieving Domain Administrator access. Phase 2 (Blue Team) analyzed all available telemetry to assess detection coverage and identify gaps.

**Final Outcome:** Red Team achieved full domain compromise in ~10 days of operation. Blue Team detected 43% of attack techniques used.

---

## Detection Coverage Summary

| Phase | Attack Techniques | Detected | Coverage |
|-------|------------------|----------|----------|
| Red Team (all tactics) | 14 | 6 | 43% |
| Initial Access | 2 | 2 | 100% |
| Privilege Escalation | 1 | 0 | 0% |
| Lateral Movement | 2 | 2 | 100% |

---

## Visible vs. Blind Activity

### Visible (Detected)
- Anonymous FTP access + file download (FTP logs + PCAP)
- RDP authentication using stolen credentials (Event 1149)
- Failed + successful SMB logon during lateral movement (Events 4625, 4624)
- Apache CVE-2021-41773 RCE payload (PCAP)
- Security log clearance on Domain Controller (Event 1102)

### Blind (Not Detected)
- Network reconnaissance (Nmap, WhatWeb) — no NIDS in place
- LSASS process memory dump — no Sysmon/EDR on target hosts
- Command execution on compromised hosts — no process-level telemetry
- Offline password hash cracking — inherently undetectable
- Full scope of activity post-DC compromise (log cleared)

---

## Critical Findings

1. **Anonymous FTP is the root cause.** Removing `rdp-settings.txt` from the FTP share — or disabling anonymous FTP entirely — would have broken the attack chain at the first step.

2. **No EDR/Sysmon on Windows hosts.** The most impactful Red Team action (LSASS dump → Domain Admin) generated **zero detectable events**. This is the single most important gap to close.

3. **Plaintext credentials at rest.** `rdp-settings.txt` storing plaintext RDP credentials represents a fundamental credential hygiene failure.

4. **Log clearance went undetected for days.** Event 1102 was the last detectable event — by then the damage was done. Real-time SIEM alerting on Event 1102 is required.

5. **Independent RCE path via Apache CVE-2021-41773.** Even if the FTP chain had been blocked, the unpatched Apache server provides a second full-compromise path. Patching Apache 2.4.49 is critical.

---

## Priority Remediation Recommendations

| Priority | Recommendation |
|----------|---------------|
| P1 | Disable anonymous FTP / remove credentials from FTP-accessible directories |
| P1 | Deploy Sysmon on all Windows hosts (EventID 10 for LSASS access) |
| P1 | Patch Apache to latest version (CVE-2021-41773 is 4+ years old) |
| P1 | Real-time SIEM alert on Event ID 1102 (log cleared) |
| P2 | Deploy EDR solution across all endpoints |
| P2 | Deploy NIDS (Suricata) for network-layer recon detection |
| P2 | Enforce strong password policy + MFA for RDP |
| P3 | Disable HTTP TRACE method (XST mitigation) |
| P3 | Restrict MySQL port to application servers only |

---

## Navigation

| Section | Description |
|---------|-------------|
| [attack-path-summary.md](attack-path-summary.md) | Full kill chain with MITRE IDs and evidence |
| [../red-team/](../red-team/) | Red Team Phase 1 — offensive operations |
| [../blue-team/](../blue-team/) | Blue Team Phase 2 — detection and analysis |
| [../docs/](../docs/) | Assignment brief (ICC-003 PDF) |
