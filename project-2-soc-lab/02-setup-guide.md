# 02 – Setup Guide

Step-by-step build of the SOC Attack Simulation Lab. Use VMware Workstation, VirtualBox, or Hyper-V with a **host-only / internal** network.

---

## 1. Create the Virtual Network

- **VMware:** Virtual Network Editor → Add Network (e.g. VMnet2) → Host-only, no DHCP if you set static IPs.  
- **VirtualBox:** Host Network Manager → Create (e.g. 192.168.100.0/24).  
- **Hyper-V:** Internal virtual switch, no external access.

---

## 2. Domain Controller (Windows Server)

1. Install **Windows Server 2019 or 2022** (Evaluation or licensed).  
2. Set static IP: 192.168.100.10, mask 255.255.255.0, DNS = 127.0.0.1 (then self).  
3. Rename computer: `DC01`.  
4. Install AD DS: **Server Manager → Add roles → Active Directory Domain Services**.  
5. Promote to DC: **New forest**, domain name `LAB.local`.  
6. Set DSRM password and complete promotion.  
7. Create domain users (e.g. `labuser` with a weak lab password like `Password1!` for simulation).  
8. Optionally create `svc_backup` or another service account for privilege escalation scenarios.

---

## 3. Windows 10 Victim(s)

1. Install **Windows 10** (21H2 or later), default or custom.  
2. Set static IP: 192.168.100.20 (and .21 for second victim). DNS = 192.168.100.10.  
3. **Join domain:** Settings → Access work or school → Connect → Join to LAB.local (use domain admin).  
4. Enable **remote management** for simulation:
   - Enable WinRM: `winrm quickconfig` (admin PowerShell); allow in firewall.  
   - Or enable RDP: Settings → Remote Desktop → On.  
5. Ensure **Windows Security / Event Log** is running; optionally install **Sysmon** (SwiftOnSecurity or similar config).  
6. Create or use a local admin account if you want to simulate privilege escalation from a standard user to admin.

---

## 4. Kali Linux Attacker

1. Download **Kali Linux** from official site; create VM attached to same host-only network.  
2. Set static IP: 192.168.100.50, gateway/DNS as needed (e.g. 192.168.100.10 for DNS).  
3. Update: `sudo apt update && sudo apt upgrade -y`.  
4. Install tools used in simulations (many are pre-installed):
   - `hydra`, `crackmapexec`, `impacket` (e.g. `psexec`, `wmiexec`, `smbexec`), `mimikatz` (for lab only), `bloodhound` (optional).  
   - Impacket: `pip3 install impacket` or `apt install python3-impacket`.

**Do not** install Wazuh/Splunk agents on Kali; Kali is the attacker source only.

---

## 5. Wazuh (SIEM Option)

1. **Server:** Ubuntu 22.04 VM, IP 192.168.100.100.  
2. Install Wazuh single-node (or follow [Wazuh install docs](https://documentation.wazuh.com/current/installation-guide/wazuh-index.html)):
   - Indexer + Server + Dashboard, or all-in-one.  
3. **Agents:** Install Wazuh agent on DC and Win10 victims; point to Wazuh server IP:1514.  
4. Enable **Windows audit policy** (via GPO or locally) so 4624/4625, 4688, 4648, 4672, etc. are logged.  
5. Optionally deploy **Sysmon** and ensure Wazuh collects Sysmon logs.

---

## 6. Splunk (SIEM Option)

1. **Splunk Enterprise** on Ubuntu or Windows, IP 192.168.100.100.  
2. Install Universal Forwarder on DC and Win10; configure to send to Splunk:9997.  
3. Add inputs for:
   - Windows Event Log (Security, optionally System/Application).  
   - Sysmon if installed.  
4. Install **Splunk Enterprise Security** (optional) for built-in correlation and detection.

---

## 7. Verification

- From Kali: `ping 192.168.100.10`, `ping 192.168.100.20`.  
- From Kali: `crackmapexec smb 192.168.100.0/24` (should list DC and Win10).  
- From Win10: open `\\DC01\SYSVOL` to confirm domain connectivity.  
- In Wazuh/Splunk: confirm agents/forwarders are connected and Security events are ingesting.

Once verified, proceed to **03-attack-simulations.md**.
