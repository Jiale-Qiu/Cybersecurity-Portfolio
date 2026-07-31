**Completion date**: 2026-04-05
**Platform:** TryHackMe, [https://tryhackme.com/room/thelasttrial]\
**Skills and Tools Used:** macOS, plistutil, apfs-fuse, forensics

## Overview
In the sixth and final part of the Honeypot Collapse on TryHackMe, I had to investigate a compromised MacOS system. Using an image of the drive, I uncovered how malware was delivered and how it kept persistence. I:
- Used plistutil and apfs-fuse to analyze artifacts related to user history, packages, and TCC permissions
- Analyzed the malicious package to find a C2 URL for exfiltration
- Analyzed LaunchAgents and LaunchDameons to uncover a persistence mechanism