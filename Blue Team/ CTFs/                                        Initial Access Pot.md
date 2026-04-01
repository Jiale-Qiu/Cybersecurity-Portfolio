**Completion date**: 2026-04-01
**Platform:** TryHackMe, [https://tryhackme.com/room/initialaccesspot]\
**Skills and Tools Used:** Linux, Auditd, Forensics, Endpoint Investigations, Live System Analysis, Web application analysis, Log Analysis

## Overview
In the first part of the Honeypot Collapse on TryHackMe, I had to investigate a Linux System. I traced attacker actions from initial access to persistence.
Tasks done:
- Identify a brute force attack on a webpage
- Find a php file being used as a backdoor
- Trace attacker actions using auditd and bash history
- Identify privilege escalation from an SSH key
- Analyze bash history to identify a port scanning attempt
- Look through services to find a service masquerading as a legitamate one, which was revealed to be malware