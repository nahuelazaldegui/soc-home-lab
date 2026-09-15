# Scenario 01 — SSH Brute Force Attack Detection

## Overview

Simulated an SSH brute force attack from a Kali Linux machine against 
an Ubuntu Server, then analyzed the generated alerts in Wazuh SIEM.

## Objective

Demonstrate the ability to:
- Execute a credential-based attack using Hydra
- Detect the attack pattern through Wazuh alert correlation
- Identify the MITRE ATT&CK technique and tactic involved
- Analyze raw logs to reconstruct the attack timeline

## MITRE ATT&CK Mapping

| Field | Value |
|-------|-------|
| Tactic | Credential Access |
| Technique | Brute Force |
| ID | T1110 |

## Environment

| Host | IP | Role |
|------|----|------|
| Kali Linux | 192.168.1.40 | Attacker |
| Ubuntu Server | 192.168.1.35 | Victim (Wazuh Agent installed) |
| Wazuh SIEM | 192.168.1.39 | Detection |

## Attack Execution

### Tool used
**Hydra v9.7** — password brute force tool

### Steps

**1. Created target user on victim machine**
```bash
sudo adduser victima
# password set to: root1
```

**2. Created custom wordlist on Kali**
```bash
cat << 'EOF' > ~/passwords.txt
123456
password
admin
root
toor
kali
test
root1
letmein
welcome
EOF
```

**3. Launched brute force attack**
```bash
hydra -l victima -P ~/passwords.txt ssh://192.168.1.35 -t 4 -V
```

### Result
[22][ssh] host: 192.168.1.35 login: victima password: root1
1 of 1 target successfully cracked


## Detection — Wazuh Alerts

### Triggered Rules

| Rule ID | Level | Description |
|---------|-------|-------------|
| 5760 | 5 | sshd: authentication failed |
| 5557 | 5 | unix_chkpwd: password check failed |
| 2502 | **10** | User missed the password more than one time |
| 5715 | 3 | sshd: authentication success |
| 5501 | 3 | PAM: Login session opened |

### Key Alert — Rule 2502 (Level 10)

Wazuh correlated multiple failed authentication attempts and triggered 
a high-severity alert identifying the brute force pattern.

```json
{
  "agent": {
    "ip": "192.168.1.35",
    "name": "siem-server"
  },
  "data": {
    "srcip": "192.168.1.40",
    "dstuser": "victima"
  },
  "rule": {
    "level": 10,
    "id": "2502",
    "description": "User missed the password more than one time",
    "mitre.technique": "Brute Force",
    "mitre.id": "T1110",
    "mitre.tactic": "Credential Access"
  }
}
```

## Attack Timeline (reconstructed from auth.log)
22:43:31 Failed password for victima from 192.168.1.40
22:43:32 Failed password for victima from 192.168.1.40
22:43:33 Failed password for victima from 192.168.1.40
22:43:33 Accepted password for victima from 192.168.1.40 ← compromise
22:43:33 Session opened for user victima ← intruder inside
22:43:44 Session closed for user victima


## Raw Log Evidence

Extracted from `/var/log/auth.log` on the victim machine:
sshd-session: Failed password for victima from 192.168.1.40 port 56540 ssh2
sshd-session: Failed password for victima from 192.168.1.40 port 56500 ssh2
sshd-session: Accepted password for victima from 192.168.1.40 port 56540 ssh2
pam_unix(sshd:session): session opened for user victima(uid=1001)
systemd-logind: New session 7 of user victima


## SOC Analyst Notes

**What happened:** Attacker at 192.168.1.40 performed a dictionary-based 
brute force attack against SSH service on 192.168.1.35, successfully 
compromising user account `victima` using password `root1`.

**Indicators of Compromise (IOCs):**
- Source IP: `192.168.1.40`
- Target user: `victima`
- Attack time: `2026-09-14 22:43:31 UTC`
- Protocol: SSH (port 22)

**Why it succeeded:**
- Weak password (`root1`) present in common wordlists
- No account lockout policy configured
- SSH exposed without fail2ban or rate limiting

**Recommended mitigations:**
- Enforce strong password policy
- Implement SSH key-based authentication, disable password auth
- Deploy fail2ban to block IPs after N failed attempts
- Restrict SSH access by source IP via firewall rules
- Monitor rule 2502 level 10+ alerts for immediate response

## Screenshots

| Screenshot | Description |
|------------|-------------|
| ![hydra](screenshots/01-hydra-output.png) | Hydra finding valid credentials |
| ![wazuh-discover](screenshots/02-wazuh-discover-alert.png) | Wazuh Discover — Rule 2502 Level 10 |
| ![threat-hunting](screenshots/03-threat-hunting-brute-force.png) | Threat Hunting — MITRE Brute Force |
| ![json-detail](screenshots/04-alert-json-detail.png) | Alert JSON with full field mapping |
| ![auth-log](screenshots/05-auth-log-evidence.png) | Raw auth.log showing attack timeline |
| ![agent-active](screenshots/06-wazuh-agent-active.png) | Wazuh agent active on victim machine |
