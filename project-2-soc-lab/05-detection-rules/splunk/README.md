# Splunk Searches – SOC Lab

- **brute_force.spl** – Multiple 4625 from same IP; threshold count  
- **lateral_movement.spl** – PsExec/WMI in 4688; WmiPrvSE children; 7045  
- **privilege_escalation.spl** – Mimikatz/lsadump in 4688; 4672; Sysmon 10 (LSASS)  
- **persistence.spl** – 4698 (tasks); 7045 (services); Sysmon 13 (Run keys)  

Use as **Saved Searches** or **Correlation Searches** (Splunk ES). Adjust `index` and `sourcetype` to match your Windows Event Log ingestion.
