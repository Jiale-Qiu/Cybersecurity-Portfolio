# Homelab Entry #6: Improving Endpoint Visibility

Although Elastic *technically* works, there is still much work to be done. Sysmon isn't configured correctly, and there are lots of visibility gaps. Therefore, I will try to identify log sources and information that I will need to detect relevant attacks on my Juice Shop endpoint.

To avoid high complexity dealing with privacy and data laws, they will not be considered here. First, I'll create a table with ATT&CK techniques I want to focus on and the data sources needed for them. For simplicity's sake, I will only choose one technique per tactic.

| ATT&CK Technique | Description | Sources Needed |
|------------------|-------------|----------------|
| T1190 | Initial Access: Exploit Public-Facing Application | Webserver Access Logs (Visibility Gap: POST bodies not visible) |
| T1059 | Execution: Command and Scripting Interpreter | Sysmon (EventID 1), Powershell Operational Log (EventID 4104) |
| T1112 | Persistence: Modify Registry | Sysmon (EventID 12,13,14) |
| T1078 | Privilege Escalation: Valid Accounts | Windows Security Logs (EventID 4624, 4625) |
| T1685.005 | Defense Impairment: Clear Windows Event Logs | Windows Security Logs (EventID 1102), Windows System Logs (EventID 104) |
| T1003.001 | Credential Access: OS Credential Dumping - LSASS Memory | Sysmon (EventID 10) |
| T1082 | Discovery: System Information Discovery | Sysmon (EventID 1) |
| T1560 | Collection: Archive Collected Data | Sysmon (EventID 1) |
| T1105 | C2: Ingress Tool Transfer | Sysmon (EventID 3, 11), Network Logs |
| T1048 | Exfiltration: Exfiltration over Alternative Protocol - Exfiltration over Unencrypted Non-C2 protocol (HTTP) | Network Logs, Sysmon (EventID 3) |
| T1485 | Impact: Data Destruction | Sysmon (EventID 1, 9, 23, 26, 28) |