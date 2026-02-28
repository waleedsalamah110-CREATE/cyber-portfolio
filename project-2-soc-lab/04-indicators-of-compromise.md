# 04 – Indicators of Compromise (IOCs)

IOCs observed during the SOC lab attack simulation. Use these to tune detection rules and hunt in your SIEM.

---

## 1. Brute Force

| Type | IOC | Description |
|------|-----|-------------|
| **Behavioral** | Many 4625 (failed logon) from same source IP in short time | Password spraying or brute force |
| **Behavioral** | 4625 with Logon Type 3 (network) or 10 (RDP) | Network logon attempts |
| **Context** | Source IP = Kali (e.g. 192.168.100.50) targeting Win10/DC | Attacker origin in lab |
| **Event** | 4625 → 4624 from same IP for same account | Successful logon after failures |
| **Threshold** | e.g. >5 failed logons in 5 minutes from one IP to one host | Custom threshold for alerting |

### Key Windows events

- **4625** – Failed logon (TargetUserName, IpAddress, LogonType, Status)  
- **4624** – Successful logon (TargetUserName, IpAddress, LogonType)  
- **4648** – Explicit credentials (if attacker uses runas /netonly type actions)

---

## 2. Lateral Movement

| Type | IOC | Description |
|------|-----|-------------|
| **Process** | `psexec`, `PsExec`, `PAExec`, `wmiexec` in 4688 | Impacket/PsExec-style execution |
| **Process** | Parent: `services.exe` or `svchost`, child: `cmd.exe` or custom binary | Service-based lateral movement |
| **Process** | `WmiPrvSE.exe` spawning unexpected child (e.g. `cmd.exe`) | WMI execution |
| **Behavioral** | 4624 Logon Type 3 from same user from different machine in quick succession | Same account used from multiple hosts |
| **Service** | 7045 – New service install (e.g. PsExec creates a service) | Service creation for remote exec |
| **Network** | SMB/WinRM from workstation to another workstation or DC (internal) | Unusual internal connections |

### Key Windows events

- **4624** – Logon Type 3 (network), 10 (RDP)  
- **4688** – Process creation (CommandLine, ParentProcess, User)  
- **7045** – Service installed  
- **4634/4647** – Logoff / logoff initiated

---

## 3. Privilege Escalation

| Type | IOC | Description |
|------|-----|-------------|
| **Process** | Process name containing `mimikatz`, `sekurlsa`, `lsadump` | Credential dumping (lab) |
| **Process** | Access to `lsass.exe` (Sysmon 10 or EID 4688 parent) | LSASS access |
| **Behavioral** | 4672 – Special privileges assigned (e.g. SeDebugPrivilege) | Privilege escalation preparation |
| **Behavioral** | 4648 – Explicit credentials used (e.g. Pass-the-Hash style) | Credential use |
| **Account** | User added to local Administrators or Domain Admins (4720, 4728, 4756) | Group membership change |
| **Process** | High-integrity or SYSTEM token from a user logon session | Unusual privilege level |

### Key Windows events

- **4672** – Special privileges assigned to logon  
- **4688** – Process (mimikatz, etc.)  
- **4648** – Explicit credentials  
- **4720** – User account created  
- **4728** – Member added to security-enabled local group  
- **4756** – Member added to security-enabled global group  
- **Sysmon 10** – Process access (TargetImage = lsass.exe)

---

## 4. Persistence

| Type | IOC | Description |
|------|-----|-------------|
| **Registry** | `Run`, `RunOnce`, `RunServices` in HKCU/HKLM with suspicious path | Persistence via Run keys |
| **Task** | 4698 – Scheduled task created (TaskName, TaskContent) | New scheduled task |
| **Task** | Task running from `temp`, `AppData`, or writable user path | Suspicious task path |
| **Service** | 7045 – New service with path in temp or unusual location | Service-based persistence |
| **Behavioral** | 4688 – Same suspicious image at boot/logon repeatedly | Persistence execution |
| **File** | Executable in `C:\Windows\Temp`, `C:\Users\*\AppData\Local\Temp` | Dropped payload (with hash if available) |

### Key Windows events

- **4698** – Scheduled task created  
- **7045** – Service installed  
- **4688** – Process (at startup/logon)  
- **Sysmon 13** – Registry value set (Run/RunOnce)  
- **Sysmon 11** – FileCreate (executable in temp)

---

## 5. Lab-Specific Reference Table

| Phase | Primary events | Example values (lab) |
|-------|----------------|----------------------|
| Brute force | 4625, 4624 | IpAddress=192.168.100.50, LogonType=3/10 |
| Lateral movement | 4624, 4688, 7045 | CommandLine with psexec/wmiexec, Parent=services |
| Privilege escalation | 4672, 4688, Sysmon 10 | Process=mimikatz, TargetImage=lsass |
| Persistence | 4698, 7045, 4688, Sysmon 13 | TaskName=Updater, Run key, new service |

---

## 6. Using IOCs in the SIEM

- **Detection rules:** Translate behavioral IOCs into SIEM rules (count of 4625 by IP, 4688 command-line patterns, etc.).  
- **Hunting:** Search for source IP, user names, task names, and process paths from this document.  
- **Incident response:** When an alert fires, use the same event IDs and fields to scope and timeline the activity.

See **05-detection-rules** for Wazuh and Splunk examples.
