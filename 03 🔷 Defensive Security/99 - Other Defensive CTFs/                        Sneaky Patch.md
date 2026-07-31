**Completion date**: 2025-12-20
**Platform:** TryHackMe, [https://tryhackme.com/room/hfb1sneakypatch]\
**Skills and Tools Used:** lsmod, modinfo, strings, cyberchef, AI

## Overview
I utilized lsmod to see all loaded kernel modules in the system, and then utilized AI to spot kernel modules that were not present on a system by default. I manually verified the flagged anomalies with modinfo, and then used the strings command on the kernel object file to locate a hexadecimal encoded version of the flag.