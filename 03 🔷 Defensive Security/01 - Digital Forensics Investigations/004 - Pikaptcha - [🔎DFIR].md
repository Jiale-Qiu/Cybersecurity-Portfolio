**Completion date:** 2026-06-26\
**Platform:** HackTheBox, [https://app.hackthebox.com/sherlocks/Pikaptcha]\
**Skills and Tools Used:** Wireshark, Google, Regipy, Regripper, Windows Analysis and Triage

## Overview
Post-Investigation Report:\
An employee named Happy Grunwald fell for a social engineering attack involving a malicious fake captcha. The attack involved copy-pasting a command into the Run menu, which downloaded a payload and executeed it, establishing a reverse shell and C2 connection. 

From HackTheBox:\
Happy Grunwald contacted the sysadmin, Alonzo, to notify him that he tried downloading the latest version of microsoft office from an update mail he received. He told that he visited the website and solved a captcha but no office download page came back. Alonzo, who himself was bombarded with phishing attacks last year and was now aware of attacker tactics, immediately notified the security team to isolate the machine as he suspected an intrusion.You are provided with network traffic and endpoint artifacts to answer few of concerning questions.

## Methodology and Thinking

### Q1: It is crucial to understand any payloads executed on the system for initial access. Analyzing registry hive for user happy grunwald. What is the full command that was run to download and execute the stager.

Given the hint to check the registry hive, I check the user `happy.grundwald`'s registry location, and retrieve `NTUSER.DAT`. After spending a lot of time setting up Regipy, I processed the transaction logs and with regripper, checked the RunMRU key as it will log commands executed from the Run menu. From there, I got my answer.

### Q2: At what time in UTC did the malicious payload execute?

Same spot as last question, but just the time the RunMRU entry was created.

### Q3: The payload which was executed initially downloaded a PowerShell script and executed it in memory. What is sha256 hash of the script?

This was a bit tougher, but I realized that I had to pivot to the network capture file that was provided and export the attacker's payload. From there, I used `sha256sum` to get the SHA256 hash of the file.

### Q4: To which port did the reverse shell connect?
I first had to decode the base64 in the payload, and from there I found an IP address and Port in the decoded string.

### Q5: For how many seconds was the reverse shell connection established between C2 and the victim's workstation?
I first thought there would be some sort of registry key or whatever that logs the time a process was active for, but it did not exist. If a certain audit setting was turned on I could read the Windows security logs, but I found out that the challenge did not provide Windows security logs. In the end, I realized I had to use the network capture again to find the first C2 communication and the last C2 communication. 

### Q6: Attacker hosted a malicious Captcha to lure in users. What is the name of the function which contains the malicious payload to be pasted in victim's clipboard?
Again with the Packet capture, but this time I checked the HTTP network connections and found the HTTP response that contained the fake captcha. Scrolling through the Javascript, I found my answer.