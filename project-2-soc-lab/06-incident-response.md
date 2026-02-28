# 06 – Incident Response Steps

Incident response playbook for the SOC lab attack chain: brute force → lateral movement → privilege escalation → persistence. Use in conjunction with **04-indicators-of-compromise.md** and **05-detection-rules**.

---

## 1. Detection & Triage

| Step | Action |
|------|--------|
| 1.1 | **Alert** – Note alert source (Wazuh/Splunk), rule name, severity, and time. |
| 1.2 | **Scope** – Identify affected hosts (IP/hostname), user accounts, and time range. |
| 1.3 | **Correlate** – Search SIEM for same source IP, same user, and related event IDs (4624/4625, 4688, 4698, 7045). |
| 1.4 | **Classify** – Map to phase: Initial access (brute) / Execution & lateral / Privilege escalation / Persistence. |
| 1.5 | **Document** – Open incident ticket; record IOCs (IPs, users, processes, tasks, hashes if available). |

---

## 2. Containment

| Step | Action |
|------|--------|
| 2.1 | **Network** – Isolate affected hosts from the rest of the lab (disable NIC or move to isolated VLAN). In production: segment or block at firewall. |
| 2.2 | **Credentials** – If compromise is confirmed: reset passwords for involved accounts (labuser, any abused service accounts). In AD: consider disabling account until reset. |
| 2.3 | **Sessions** – Terminate active RDP/sessions to affected systems (e.g. `logoff` or restart). |
| 2.4 | **Preserve** – Before reimaging: snapshot or copy logs (Security, Sysmon), RAM if needed, and any suspected malware samples. |

---

## 3. Eradication

| Step | Action |
|------|--------|
| 3.1 | **Persistence** – Remove scheduled tasks, Run key entries, and malicious services created by the attacker (see 03-attack-simulations Phase 4). Use autoruns or manual checks. |
| 3.2 | **Artifacts** – Delete dropped files (e.g. from `C:\Windows\Temp`, user Temp). Verify with IOCs from 04. |
| 3.3 | **Accounts** – Remove any unauthorized accounts or group memberships (e.g. user added to local/domain admins). |
| 3.4 | **Rebuild** – In lab, reimage or rebuild VMs for a clean state; in production, follow org policy (rebuild vs. harden and monitor). |

---

## 4. Recovery

| Step | Action |
|------|--------|
| 4.1 | **Restore** – Bring systems back from known-good backup or clean build; restore data from trusted backup if needed. |
| 4.2 | **Harden** – Apply patches, enforce strong passwords/MFA, restrict RDP/WinRM (e.g. allow-list IPs), disable unnecessary services. |
| 4.3 | **Verify** – Confirm persistence mechanisms are gone (no suspicious tasks/services/registry). |
| 4.4 | **Monitoring** – Re-enable or add detection rules (05-detection-rules); watch for recurrence. |

---

## 5. Post-Incident

| Step | Action |
|------|--------|
| 5.1 | **Timeline** – Build attack timeline (first 4625 → first 4624 → lateral → priv esc → persistence) for report. |
| 5.2 | **IOCs** – Update 04-indicators-of-compromise with any new IOCs (hashes, paths, IPs). |
| 5.3 | **Rules** – Tune or add detection rules (05-detection-rules) to reduce false positives and catch similar TTPs. |
| 5.4 | **Lessons learned** – Document what was missed, what worked, and lab improvements (e.g. Sysmon, GPOs, password policy). |

---

## 6. Quick Reference – Event IDs to Search

| Phase | Event IDs / Data |
|-------|-------------------|
| Brute force | 4625, 4624 |
| Lateral movement | 4624 (type 3/10), 4688, 7045 |
| Privilege escalation | 4672, 4688, Sysmon 10 (LSASS) |
| Persistence | 4698, 7045, 4688 (startup), Sysmon 13 |

Use these in Wazuh/Splunk to scope the incident and validate containment/eradication.
