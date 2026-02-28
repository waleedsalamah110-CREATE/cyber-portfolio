# 01 – Lab Architecture

## Network Design

Use an **isolated host-only or private virtual network**. No NAT to your real LAN during attacks.

| Host | OS | IP (example) | Role |
|------|-----|--------------|------|
| **DC** | Windows Server 2019/2022 | 192.168.100.10 | Domain Controller, DNS, AD |
| **Win10-Victim** | Windows 10 (21H2+) | 192.168.100.20 | Domain-joined workstation, attack target |
| **Win10-Victim2** | Windows 10 (optional) | 192.168.100.21 | Second host for lateral movement |
| **Kali** | Kali Linux | 192.168.100.50 | Attacker VM only |
| **Wazuh/Splunk** | Ubuntu 22.04 or Windows | 192.168.100.100 | SIEM (agent on DC + Win10) |

- **Subnet:** 192.168.100.0/24  
- **Gateway:** None required for host-only; use 192.168.100.1 if you use a virtual router.  
- **DNS:** Point all VMs to DC at 192.168.100.10.

## High-Level Diagram

```
                    [ Host-only / Private vSwitch ]
                                    │
    ┌───────────────┬───────────────┼───────────────┬───────────────┐
    │               │               │               │               │
  [Kali]      [Win10-Victim]   [Win10-Victim2]   [DC]        [Wazuh/Splunk]
 .100.50          .100.20           .100.21      .100.10          .100.100
 Attacker         Target 1          Target 2    AD/DNS           SIEM
```

## Active Directory (Minimal)

- **Domain:** `LAB.local` (or `SOCLAB.local`)  
- **DC hostname:** `DC01`  
- **Users (examples):**
  - `labuser` – standard domain user (low privilege on workstations)  
  - `svc_backup` – service account (optional; for privilege escalation scenario)  
  - `Administrator` – domain admin (do not use for daily logon; target for escalation)  
- **Groups:** Domain Users, Domain Admins, local Administrators on workstations as needed for lab scenarios.

## Data Flow for Detection

- **Win10 + DC** → Wazuh/Splunk agents → **SIEM**  
- Enable: Windows Security events (4688, 4624/4625, 4648, 4672, 4698, etc.), Sysmon (optional but recommended), and Wazuh/Splunk agent logs.

## Ports to Allow (Lab Only)

| From | To | Port/Service | Purpose |
|------|-----|--------------|---------|
| Kali | Win10/DC | 445/TCP (SMB) | Brute force, lateral movement |
| Kali | Win10/DC | 3389/TCP (RDP) | Brute force, lateral movement |
| Kali | Win10/DC | 5985/TCP (WinRM) | Lateral movement |
| Win10 | DC | 88, 389, 636, 445, 135, 139 | Kerberos, LDAP, SMB, RPC |
| All | Wazuh/Splunk | 1514 (Wazuh) / 9997 (Splunk) | Agent to SIEM |
