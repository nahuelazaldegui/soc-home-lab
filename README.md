# SOC Home Lab

Personal cybersecurity home lab built to practice threat detection, 
log analysis, and incident response skills aligned with SOC Level 1 roles.

## Lab Architecture

| Host | IP | Role |
|------|----|------|
| Ubuntu Server 26.04 | 192.168.1.35 | Victim machine + Wazuh Agent |
| Ubuntu Server 24.04 | 192.168.1.39 | Wazuh Manager + Dashboard (SIEM) |
| Kali Linux | 192.168.1.40 | Attacker machine |

All VMs run on VMware Workstation on a Windows 11 host (AMD Ryzen 7 7840HS, 16GB RAM).

## Network Topology

[Kali Linux - Attacker] ──attack──► [Ubuntu Server - Victim]
192.168.1.40 192.168.1.35
│
Wazuh Agent
│
▼
[Wazuh SIEM - Monitor]
192.168.1.39
(Dashboard via browser)


## Tools Used

- **Wazuh v4.14.7** — SIEM and XDR platform
- **Hydra v9.7** — SSH brute force tool
- **Nmap** — Network reconnaissance
- **VMware Workstation** — Virtualization

## Scenarios

| # | Scenario | Tactic (MITRE ATT&CK) | Status |
|---|----------|-----------------------|--------|
| 01 | [SSH Brute Force Attack](scenarios/01-ssh-brute-force/README.md) | Credential Access - T1110 | ✅ Completed |

## Skills Demonstrated

- SIEM deployment and configuration (Wazuh)
- Wazuh Agent enrollment and monitoring
- Attack simulation in isolated lab environment
- Log analysis (`/var/log/auth.log`)
- Alert triage and rule interpretation
- MITRE ATT&CK framework mapping

  
