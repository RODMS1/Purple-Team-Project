# CyberCore Purple Team Security Exercise

Purple Team simulation: execute offensive actions first, then analyze detection visibility in SOC logs.

## Overview

Simulate external attacker with initial internal foothold. Focus: attacker behavior in logs, detection successes, and blind spots. No IR/prevention.

## Phases

- **Phase 1 – Red Team (Black Box)**  
- **Phase 2 – SOC Visibility & Detection Engineering**

## Scope

**In-scope**: 10.0.0.0/24  

**Out-of-scope (do NOT target)**:  
- 10.0.0.23  
- 10.0.0.100  
- 10.0.0.200  
- 10.0.0.176

**Initial access**: 10.0.0.219 (credentials provided separately)  

## Rules of Engagement – Prohibited

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


