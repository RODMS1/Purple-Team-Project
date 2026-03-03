# Network Forensic Evidence — PCAP Analysis

**Exercise:** ICC-003 · CyberCore Purple Team
**Log Source:** Network PCAP capture
**Key Findings:** Apache CVE-2021-41773 RCE payload, FTP credential exfiltration, SMB lateral movement

---

## Finding 1 — Apache CVE-2021-41773 RCE Payload

| Field | Value |
|-------|-------|
| **Protocol** | HTTP |
| **Target** | `10.0.0.219` |
| **CVE** | CVE-2021-41773 |
| **CVSS** | 9.8 (Critical) |
| **Technique** | T1190 — Exploit Public-Facing Application |
| **Visibility** | Detected in PCAP |

**Description:**
The PCAP captured an HTTP request exploiting the Apache 2.4.49 path traversal vulnerability. The payload used URL-encoded path traversal sequences to escape the document root and access `/bin/sh` via `mod_cgi`, achieving unauthenticated Remote Code Execution.

**Evidence:**
```
POST /cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh HTTP/1.1
Host: 10.0.0.219
Content-Type: application/x-www-form-urlencoded

echo Content-Type: text/plain; echo; id; whoami
```

**Detection Value:**
This attack is highly detectable at the network level. The URL-encoded path traversal sequence (`%2e`) in the request path and the `/cgi-bin/` prefix with shell payloads in the body are distinctive signatures.

**IDS Signature:** Snort/Suricata rules exist for CVE-2021-41773. Any WAF or NIDS in the path would have flagged this request.

---

## Finding 2 — FTP Credential Exfiltration (Cleartext)

| Field | Value |
|-------|-------|
| **Protocol** | FTP |
| **Source** | `10.0.0.219` |
| **Destination** | `10.0.0.40` |
| **Technique** | T1005 — Data from Local System |
| **Visibility** | Fully reconstructed from PCAP |

**Description:**
FTP operates in cleartext. The full session (USER, PASS, commands, file content) was reconstructed from the PCAP. The credentials used (`anonymous`/`anonymous`), the directory listing, and the downloaded file contents (`rdp-settings.txt`) are all visible in plaintext.

**Detection Value:**
Any network monitoring tool with FTP dissection (Wireshark, Zeek, Suricata) would have captured the full session including the exfiltrated file content. The use of anonymous FTP is an anomaly in most corporate networks.

---

## Finding 3 — SMB Lateral Movement

| Field | Value |
|-------|-------|
| **Protocol** | SMB |
| **Source** | `10.0.0.219` |
| **Destination** | `10.0.0.40` (via `\\10.0.0.40\dump`) |
| **Technique** | T1021.002 — Remote Services: SMB/Windows Admin Shares |
| **Visibility** | Corroborated by Event 4624 |

**Description:**
SMB traffic observed from `10.0.0.219` to `10.0.0.40` following the RDP session. This corresponds to the LSASS dump file being exfiltrated to the `\\10.0.0.40\dump` share. The SMB session is authenticated using credentials obtained from the LSASS dump, representing a second-hop lateral movement.

---

## PCAP Gaps

| Activity | Visible in PCAP? | Reason |
|----------|-----------------|--------|
| Apache RCE payload | Yes | HTTP cleartext |
| FTP anonymous login + file download | Yes | FTP cleartext |
| SMB lateral movement | Partial | Encrypted; metadata visible but content not |
| LSASS dump operation | No | Local process — no network traffic |
| Hash cracking | No | Offline activity |
| RDP interactive session | Partial | RDP is encrypted; auth events only |

---

## Summary

Network-layer monitoring captured **2 of 6 attack phases with high fidelity** (FTP exfiltration, Apache RCE). The remaining phases either operated locally or used encrypted protocols. This highlights the need for endpoint detection (EDR/Sysmon) alongside network monitoring — network visibility alone leaves significant blind spots in a modern attack chain.
