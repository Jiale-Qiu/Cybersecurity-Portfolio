**Completion date**: 2026-04-04
**Platform:** TryHackMe, [https://tryhackme.com/room/shockandsilence]\
**Skills and Tools Used:** Windows, NTFS Analysis, Forensics

## Overview
In the fifth part of the Honeypot Collapse on TryHackMe, I had to investigate a compromised Windows System. Using only a partial image of an NTFS hard drive and its metadata, I uncovered how the ransomware was delivered and what group targeted the organization. I:
- Used the Master File Table ($MFT) to read alternate data streams, uncovering the mark of the web, revealing the origin of the initial payload
- Used the Master File Table ($MFT) to find the ransomware executable downloaded to the host
- Used $UsnJrnl to correlate when the payload was intially delivered and when the files started being encrypted to uncover the executable file that initiated the encryption process
- Used OSINT to uncover the ransomware group behind the attack