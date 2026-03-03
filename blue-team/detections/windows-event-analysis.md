# Windows Event Log Analysis

**Exercise:** ICC-003 · CyberCore Purple Team
**Log Source:** Windows Security Event Log
**Events Detected:** 4 (1102, 4625, 4624, 1149)

---

## Event 1102 — Security Log Cleared

| Field | Value |
|-------|-------|
| **Severity** | Critical |
| **Action Type** | Destructive / Anti-Forensics |
| **Timestamp** | 2026-02-23 · 16:52:47 UTC |
| **Computer** | `EC2AMAZ-R2T6KBN.corp.org` (Domain Controller) |
| **Subject** | `CORP\Administrator` |
| **Logon ID** | `0x739FA` |
| **Log Size Before Clear** | 24,362 events |
| **MITRE Technique** | T1070.001 — Indicator Removal: Clear Windows Event Logs |

**Analysis:**
The security audit log on the Domain Controller was manually cleared by `CORP\Administrator` — a classic post-compromise anti-forensics action. This event occurred **1 day after the confirmed domain compromise** (2026-02-22), indicating the attacker returned to destroy evidence of their lateral movement and privilege escalation activity. The deletion of 24,362 events represents a significant loss of forensic telemetry.

**Detection Rule Recommendation:**
Alert on any occurrence of Event ID 1102 on Domain Controllers. This event has no legitimate routine use case and should always be treated as a high-confidence indicator of compromise.

---

## Event 4625 — Failed Logon Attempt

| Field | Value |
|-------|-------|
| **Severity** | High |
| **Action Type** | Credential Attack |
| **Timestamp** | 2026-02-15 · 04:23:31 AM |
| **Computer** | `DESKTOP-RCUCBMP` |
| **Account** | `Administrator` (WORKGROUP) |
| **Source IP** | `10.0.0.219` |
| **Status Code** | `0xC000006D` — bad username or authentication |
| **Sub-Status Code** | `0xC000006A` — correct username, wrong password |
| **Process** | Caller PID `0x0` — system-initiated |
| **MITRE Technique** | T1078 — Valid Accounts |

**Analysis:**
Initial failed authentication attempt from the attacker's host (`10.0.0.219`) at 04:23:31. The sub-status code `0xC000006A` is significant: it confirms the **username was valid** but the password was incorrect. This indicates the attacker had already enumerated valid account names before attempting authentication. A successful logon followed 23 seconds later (Event 4624).

**Detection Rule Recommendation:**
Correlate Event 4625 sub-status `0xC000006A` with subsequent successful logons (4624) from the same source IP within a short window — high confidence indicator of credential spraying or targeted access.

---

## Event 4624 — Successful Network Logon (Type 3)

| Field | Value |
|-------|-------|
| **Severity** | High |
| **Action Type** | Unauthorized Access |
| **Timestamp** | 2026-02-15 · 04:23:54 AM |
| **Computer** | `DESKTOP-RCUCBMP` |
| **Account** | `DESKTOP-RCUCBMP\Admin` |
| **Logon ID** | `0x41ED0C5` |
| **Source IP** | `10.0.0.219:44322` |
| **Workstation** | `IP-10-0-0-219` |
| **Auth Package** | `NTLMv2` · 128-bit session key |
| **Logon Type** | `3` — Network (SMB share access) |
| **MITRE Technique** | T1021.002 — Remote Services: SMB/Windows Admin Shares |

**Analysis:**
Successful SMB/network logon 23 seconds after the failed attempt. The attacker from `10.0.0.219` authenticated as local `Admin` via NTLMv2. Logon Type 3 indicates network authentication — this access enabled the subsequent LSASS dump file exfiltration via the `\\10.0.0.40\dump` SMB share. The 23-second gap between failure and success is consistent with a manual credential attempt (not automated brute-force).

**Detection Rule Recommendation:**
Alert on Type 3 logons originating from unexpected source IPs, particularly when preceded by a 4625 failure from the same host. Cross-reference with known admin workstations.

---

## Event 1149 — RDP Authentication Success

| Field | Value |
|-------|-------|
| **Severity** | Medium |
| **Action Type** | Initial Foothold |
| **Timestamp** | 2026-02-13 · 08:59:23 AM |
| **Target Host** | `10.0.0.119` (DESKTOP-RCUCBMP) |
| **Account** | `Admin` (local administrator) |
| **Source IP** | `10.0.0.219` |
| **Log Source** | `TerminalServices-RemoteConnectionManager` |
| **Preceding Event** | Anonymous FTP download of `rdp-settings.txt` |
| **MITRE Technique** | T1021.001 — Remote Services: Remote Desktop Protocol |

**Analysis:**
RDP authentication success logged by TerminalServices-RemoteConnectionManager. The attacker (`10.0.0.219`) used credentials obtained from the anonymous FTP server (`rdp-settings.txt`). This is the **initial foothold** that enabled all subsequent compromise — LSASS dumping, SMB lateral movement, and domain admin escalation. Occurred 2 days before Events 4625/4624, confirming the FTP→RDP kill chain sequence.

**Detection Rule Recommendation:**
Alert on RDP authentication (Event 1149) from non-standard source IPs. Cross-correlate with FTP access logs to identify the credential theft chain.

---

## Timeline Correlation

```
2026-02-13 08:59:23  Event 1149  — RDP login (Admin)        ← INITIAL FOOTHOLD via FTP creds
2026-02-15 04:23:31  Event 4625  — Failed SMB logon         ← Lateral movement attempt
2026-02-15 04:23:54  Event 4624  — Successful SMB logon     ← LSASS dump exfil path established
2026-02-22 (approx)              — Domain Admin compromise
2026-02-23 16:52:47  Event 1102  — Security log cleared     ← Anti-forensics (24,362 events deleted)
```
