**Completion date**: 2026-04-02
**Platform:** TryHackMe, [https://tryhackme.com/room/lostinramslation]\
**Skills and Tools Used:** Linux, Volatility 3

## Overview
In the third part of the Honeypot Collapse on TryHackMe, I had to investigate a compromised windows System. Using an image of the system's memory, I investigated malicious processes and detected meterpreter. I:
- Found the first malicious file executed in the chain
- Correlated the file execution to a process
- Investigated the process chain
- Used malfind to spot a meterpreter shellcode in a process.