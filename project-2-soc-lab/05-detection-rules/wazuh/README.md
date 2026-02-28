# Wazuh Rules – SOC Lab

- **brute-force-rules.xml** – 4625/4624, multiple failures from same IP  
- **lateral-movement-rules.xml** – PsExec, WMI, service-based execution  
- **privilege-escalation-rules.xml** – Mimikatz, LSASS, special privileges  
- **persistence-rules.xml** – Scheduled tasks, services, Run keys  

## Deployment

1. Copy `.xml` files to `/var/ossec/etc/rules/` on the Wazuh server.  
2. Ensure `<include>` in `ossec.conf` loads these files (or add them).  
3. Restart Wazuh: `systemctl restart wazuh-manager`.  

## Note on rule IDs

`<if_sid>` values (61500, 61501, 61602, 61603, 61607, 61608) refer to default Wazuh Windows decoder rules. Verify IDs in your Wazuh version; adjust if your default rule set differs.
