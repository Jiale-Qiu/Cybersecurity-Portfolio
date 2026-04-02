**Completion date**: 2026-04-02
**Platform:** TryHackMe, [https://tryhackme.com/room/elevatingmovement]\
**Skills and Tools Used:** Windows, EZTools, EventViewer, Windows Forensics, Windows Registry, Pypykatz

## Overview
In the second part of the Honeypot Collapse on TryHackMe, I had to investigate a Windows System as the attacker moved throughout the network. I traced attacker actions from RDP access to lateral movement.
Tasks done:
- Correlated the enterprise network with RDP logons
- Detected the replacement of an IT authorized file for persistence, and identified the malware family the new file was under
- Used Powershell transcripts to spot the dumping of LSASS
- Used Pypykatz to retrieve the NTLM hash of a user's password