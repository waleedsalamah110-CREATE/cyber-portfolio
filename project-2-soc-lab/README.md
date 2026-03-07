# PROJECT 2 –# 🛡 Cyber Security Portfolio – Waleed Salamah
 Full SOC Attack Simulation Lab

**Blue Team capability | SOC / DFIR portfolio piece**

A controlled lab to simulate real-world attack chains, tune detection (Wazuh/Splunk), and practice incident response.

---

## Lab Overview

| Component | Role |
|-----------|------|
| **Kali Linux** | Attacker VM – offensive tools only in this environment |
| **Windows 10** | Victim workstation – joined to lab domain |
| **Active Directory** | Domain controller (Windows Server) – users, GPOs, Kerberos/LDAP |
| **Wazuh or Splunk** | SIEM – collect logs, run detection rules, support IR |

---

## Attack Chain Simulated

1. **Brute force** – Password attacks against RDP/SMB/WinRM or web login  
2. **Lateral movement** – After initial access, move to other hosts (e.g. PsExec, WMI, RDP)  
3. **Privilege escalation** – From standard user to local admin or domain admin  
4. **Persistence** – Survive reboot (scheduled tasks, registry, services, etc.)

---

## Repository Structure

```
project-2-soc-lab/
├── README.md                 # This file
├── 01-lab-architecture.md    # Network diagram, IPs, roles
├── 02-setup-guide.md         # Step-by-step VM & AD & SIEM setup
├── 03-attack-simulations.md  # How to run each attack phase
├── 04-indicators-of-compromise.md   # IOCs from the simulations
├── 05-detection-rules/       # Wazuh + Splunk rules
│   ├── wazuh/
│   └── splunk/
└── 06-incident-response.md   # IR steps and playbook
```

---

## Quick Start

1. Build the lab using **02-setup-guide.md**.  
2. Run attacks in order using **03-attack-simulations.md**.  
3. Confirm events in Wazuh/Splunk and map to **04-indicators-of-compromise.md**.  
4. Deploy **05-detection-rules** and validate alerts.  
5. Follow **06-incident-response.md** for containment and recovery.

---

## Legal & Safety

- **Isolated lab only** – No production networks or internet-facing systems.  
- **Authorized use only** – All activity must be in your own lab or with explicit permission.  
- Use strong, unique passwords and restrict lab network access.

---

## Detection Tuning & False Positive Reduction

Detection engineering is not only about triggering alerts — it is about balancing sensitivity and precision.

The following tuning strategies were considered:

### Brute Force
- Threshold-based alerting (failed attempts within time window)
- Exclusion of known service accounts
- Geo/IP reputation enrichment
- Rate-based anomaly detection

### Lateral Movement
- Baseline comparison of normal administrative behavior
- Detection of rare parent-child process relationships
- Correlation across multiple hosts

### Privilege Escalation
- Monitoring abnormal token manipulation
- Identifying uncommon service creations
- Flagging suspicious scheduled task creation

### Persistence
- Detecting new registry run keys
- Service creation anomalies
- Startup folder modifications

---

## Detection Improvement Roadmap

Future enhancements:

- Behavioral baselining per user
- Integration with threat intelligence feeds
- Risk scoring model
- Automated response playbooks
- Sigma rule export compatibility

This lab demonstrates both detection creation and detection refinement strategy.