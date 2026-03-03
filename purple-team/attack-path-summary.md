# Attack Path Summary — Full Kill Chain

**Exercise:** ICC-003 · CyberCore Purple Team
**Result:** Full Active Directory Domain Compromise (corp.org)

---

## Primary Kill Chain — FTP → RDP → LSASS → Domain Admin

```
┌─────────────────────────────────────────────────────────────────┐
│  STEP 1 — Anonymous FTP Reconnaissance                          │
│  Host: 10.0.0.40 (FileZilla Server 1.8.2)                      │
│  MITRE: T1083 — File and Directory Discovery                    │
│  Action: Anonymous login → directory listing → identify target  │
│  Evidence: FTP Session 1110 · 2026-02-24 05:36:44 UTC          │
└─────────────────────────────┬───────────────────────────────────┘
                              │ rdp-settings.txt downloaded (46 bytes)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 2 — Credential Theft via FTP Exfiltration                 │
│  MITRE: T1048 — Exfiltration Over Alternative Protocol          │
│  Action: RETR rdp-settings.txt → plaintext RDP creds recovered  │
│  Evidence: PCAP · FTP RETR command captured                     │
└─────────────────────────────┬───────────────────────────────────┘
                              │ Admin / [password]
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 3 — RDP Initial Foothold                                  │
│  Host: 10.0.0.119 (DESKTOP-RCUCBMP)                            │
│  MITRE: T1021.001 — Remote Services: RDP                        │
│  Action: RDP login as Admin using stolen credentials            │
│  Evidence: Event 1149 · 2026-02-13 08:59:23 AM                 │
└─────────────────────────────┬───────────────────────────────────┘
                              │ Local Admin shell
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 4 — LSASS Credential Dump                                 │
│  Host: 10.0.0.119 (DESKTOP-RCUCBMP)                            │
│  MITRE: T1003.001 — OS Credential Dumping: LSASS Memory        │
│  Action: LSASS memory dumped → NTLM hashes extracted           │
│  Evidence: NO DETECTION (no Sysmon/EDR on host)                │
└─────────────────────────────┬───────────────────────────────────┘
                              │ NTLM hash exfil via \\10.0.0.40\dump
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 5 — SMB Lateral Movement (Exfil Path)                     │
│  Host: 10.0.0.219 → 10.0.0.40                                  │
│  MITRE: T1021.002 — SMB/Windows Admin Shares                    │
│  Action: SMB auth to dump share to retrieve LSASS dump file     │
│  Evidence: Event 4624 (Type 3) · Event 4625 · PCAP             │
└─────────────────────────────┬───────────────────────────────────┘
                              │ NTLM hash
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 6 — Offline Hash Cracking                                 │
│  MITRE: T1110.002 — Brute Force: Password Cracking             │
│  Action: Hashcat/John offline crack → Domain Admin plaintext   │
│  Evidence: NO DETECTION (offline activity)                      │
└─────────────────────────────┬───────────────────────────────────┘
                              │ Domain Administrator credentials
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 7 — DOMAIN ADMINISTRATOR ACCESS                           │
│  Host: EC2AMAZ-R2T6KBN.corp.org (Domain Controller)            │
│  MITRE: T1078 — Valid Accounts                                  │
│  Result: FULL corp.org Active Directory compromise              │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 8 — Anti-Forensics                                        │
│  Host: EC2AMAZ-R2T6KBN.corp.org (Domain Controller)            │
│  MITRE: T1070.001 — Indicator Removal: Clear Windows Event Logs │
│  Action: Security log cleared (24,362 events deleted)          │
│  Evidence: Event 1102 · 2026-02-23 16:52:47 UTC                │
└─────────────────────────────────────────────────────────────────┘
```

---

## Secondary Attack Path — Apache RCE (Independent)

```
┌─────────────────────────────────────────────────────────────────┐
│  Apache CVE-2021-41773 — Path Traversal / RCE                   │
│  Host: 10.0.0.219 (Apache 2.4.49)                              │
│  MITRE: T1190 — Exploit Public-Facing Application              │
│  Action: HTTP path traversal payload → /bin/sh via mod_cgi     │
│  Result: Unauthenticated RCE as web server user                │
│  Evidence: RCE payload captured in PCAP                        │
│                                                                  │
│  Note: This path operates independently of the FTP→RDP chain.  │
│  Even without the FTP credential theft, this vulnerability       │
│  provides an attacker with a foothold on 10.0.0.219.           │
└─────────────────────────────────────────────────────────────────┘
```

---

## Detection Overlay

| Step | Detected | Source |
|------|----------|--------|
| 1 — FTP recon | Yes (post-incident) | FTP logs + PCAP |
| 2 — File exfiltration | Yes (post-incident) | PCAP |
| 3 — RDP foothold | Yes | Event 1149 |
| 4 — LSASS dump | **No** | No EDR/Sysmon |
| 5 — SMB lateral movement | Yes | Events 4624/4625 + PCAP |
| 6 — Hash cracking | **No** | Offline |
| 7 — Domain Admin access | **No** (log cleared) | Event 1102 only |
| 8 — Log clearance | Yes | Event 1102 |
| Apache RCE | Yes | PCAP |

**Detected: 6/9 steps** — but the most critical step (LSASS dump → DA) was invisible.
