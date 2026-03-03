# FTP Forensics — Session 1110

**Exercise:** ICC-003 · CyberCore Purple Team
**Log Source:** FileZilla FTP Server Logs + PCAP
**Key Finding:** Anonymous login used to download plaintext RDP credentials

---

## Session Metadata

| Field | Value |
|-------|-------|
| **Session ID** | 1110 |
| **Date** | 2026-02-24 · 05:36:44 UTC |
| **Source (Attacker)** | `10.0.0.219` |
| **Destination (FTP Server)** | `10.0.0.40` — FileZilla Server 1.8.2 |
| **Authentication Method** | Anonymous |
| **File Exfiltrated** | `rdp-settings.txt` (46 bytes) |
| **Session Duration** | ~14 seconds |
| **MITRE Technique** | T1083 — File and Directory Discovery |
| **MITRE Technique** | T1005 — Data from Local System |

---

## Packet-by-Packet Analysis

| # | Time (rel.) | Command | Description | Status |
|---|------------|---------|-------------|--------|
| 1 | 14.110s | `USER anonymous` | Anonymous login attempt | 230 Proceed |
| 2 | 18.287s | `PASS anonymous` | Anonymous password — file-based auth | 230 Login successful |
| 3 | 18.288s | `SYST` | Query server OS type (reconnaissance) | Success |
| 4 | 18.288s | `FEAT` | Query server feature list (reconnaissance) | Success |
| 5 | 20.600s | `EPSV` | Extended passive mode — data channel setup | Success |
| 6 | 20.601s | `LIST` | Directory listing — filesystem exploration | Success |
| 7 | 24.084s | `EPSV` | Extended passive mode — second transfer | Success |
| 8 | 24.086s | `NLST` | List filenames only — targeted enumeration | Success |
| 9 | 25.134s | `TYPE I` | Set transfer mode to Binary/Image | Success |
| 10 | 25.134s | `SIZE rdp-settings.txt` | Check target file size before download | 213 · 46 bytes |
| 11 | 25.135s | `EPSV` | Extended passive mode — download channel | Success |
| **12** | **25.136s** | **`RETR rdp-settings.txt`** | **EXFILTRATION — RDP settings downloaded from `C:\ftp-files\`** | **Success (CRITICAL)** |
| 13 | 25.139s | `MDTM rdp-settings.txt` | Query file modification timestamp | Success |
| 14 | 27.994s | `QUIT` | Session terminated gracefully | 221 Goodbye |

---

## Analysis

**Attacker Behavior:**
The FTP session follows a deliberate pattern of reconnaissance before exfiltration:
1. `SYST` + `FEAT` — server fingerprinting before interacting with filesystem
2. `LIST` + `NLST` — two directory listing methods used to identify filenames
3. `SIZE` — file size checked before download (efficient, low-noise)
4. `RETR` — targeted download of the single high-value file (`rdp-settings.txt`)
5. `MDTM` — timestamp checked after download (likely to preserve metadata for operational security)

**Why This Matters:**
The `rdp-settings.txt` file (46 bytes) contained plaintext RDP credentials for the `Admin` account on `10.0.0.119`. These credentials were used 2 hours later to establish the initial RDP foothold (Event 1149 — 2026-02-13 08:59:23). This FTP session is the **root cause** of the entire compromise chain.

---

## Detection Gap

- FileZilla logged the session, but **no alert was configured** for anonymous FTP downloads
- The small file size (46 bytes) and brief session (~14s) are low-noise — easy to miss without targeted monitoring
- `rdp-settings.txt` in an FTP-accessible directory is a critical misconfiguration (plaintext credentials at rest)

---

## Recommendations

1. **Disable anonymous FTP** — no legitimate use case should require unauthenticated file access
2. **Remove sensitive files from FTP directories** — credentials must never be stored in plaintext in shared locations
3. **Alert on any `RETR` command** from anonymous sessions
4. **Enable FileZilla logging to SIEM** with alerting on anonymous authentication events
