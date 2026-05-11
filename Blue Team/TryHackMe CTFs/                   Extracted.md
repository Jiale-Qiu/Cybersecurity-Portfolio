**Completion date**: 11/23/2025\
**Platform:** TryHackMe, [https://tryhackme.com/room/extractedroom]\
**Skills and Tools Used:** Cyberchef, TShark, Wireshark, Linux CLI, Reverse Engineering, Hashcat, Encoding/Decoding

## Overview
I analyzed a packet capture to find a file being delivered to a suspicious IP address. I successfully extracted the powershell file used to retrieve it, as well as the file itself. Using the powershell file, I decoded the delivered files and then utilized a CVE to extract a password.