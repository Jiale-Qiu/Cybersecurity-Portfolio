**Completion date:** 2026-06-15\
**Platform:** HackTheBox, [https://app.hackthebox.com/sherlocks/MangoBleed]\
**Skills and Tools Used:** Built-in linux tools, Google

## Overview
Post-Investigation Report:\
The attacker attempted to exploit CVE-2025-14847, also known as MongoBleed on the MongoDB server. Furthermore, the attacker launched a brute force attack on the server and signed into the `mongoadmin` user, attempting to perform privilege escalation and exfiltrate data.

From HackTheBox:\
"You were contacted early this morning to handle a high‑priority incident involving a suspected compromised server. The host, mongodbsync, is a secondary MongoDB server. According to the administrator, it's maintained once a month, and they recently became aware of a vulnerability referred to as MongoBleed. As a precaution, the administrator has provided you with root-level access to facilitate your investigation. You have already collected a triage acquisition from the server using UAC. Perform a rapid triage analysis of the collected artifacts to determine whether the system has been compromised, identify any attacker activity (initial access, persistence, privilege escalation, lateral movement, or data access/exfiltration), and summarize your findings with an initial incident assessment and recommended next steps."

## Methodology and Thinking

### Q1: What is the CVE ID designated to the MongoDB vulnerability explained in the scenario?
This one was fairly simple. All that was needed was to search up "MongoBleed" in google.

### Q2: What is the version of MongoDB installed on the server that the CVE exploited?
First, I will figure out what im dealing with. I execute `ls *` to quickly view what is in each directory. 

Seeing that root and its subdirectories are given, I check the MongoDB logs. I filter the output for strings with "version", and I find a line containing MongoDB's version.

### Q3: Analyze the MongoDB logs to identify the attacker's remote IP address used to exploit the CVE.
This will require knowledge of the CVE and how it is exploited. With more research, I stumble upon [this article.](https://blog.ecapuano.com/p/hunting-mongobleed-cve-2025-14847)

It seems that MongoBleed exploitation involves sending tens of thousands of connections to the server per minute. It connects, then disconnects, but never recieves any metadata. Thus, I can now search for MongoBleed

I use a somewhat janky, but working linux command: `cat \[root\]/var/log/mongodb/mongod.log | grep '"id":22943' | grep -oP '(?<=remote":").*(?=","isLoadBalanced)' | cut -d: -f1 | sort | uniq`

This linux command reads the mongodb logs, greps for connection events (id 22943), extracts the IP address, strips away the port number, and filters for unique occurences. This all leads to one single IP address. If there were multiple IP addresses then further investigation is warranted, but this singular IP should most likely be the attacker.

### Based on the MongoDB logs, determine the exact date and time the attacker’s exploitation activity began (the earliest confirmed malicious event)
This will need slightly more investigation. First, I filter the logs to the malicious IP only. What I want is the first event that goes like this:
1. Event 22943 -- Connection accepted
2. Event 51800 MISSING -- Metadata
3. Event 22944 -- Connection closed

So first, I will filter for event 22943, then use the `-A` flag to see the event right after. I expect this to be event 22944, so I grep for that, and then use the `-B` flag to keep the previous event. Finally, I use the `head` command to get the first few lines. In the end, I get this commandline command and get my answer.

`cat \[root\]/var/log/mongodb/mongod.log | grep "65.0.76.43" | grep '"id":22943' -A 1 | grep '"id":22944' -B 1 | head`

### Using the MongoDB logs, calculate the total number of malicious connections initiated by the attacker.
First, I filter for metadata events, to see if there were any legit connections. There were none, making my job a lot easier. I first filter for the attacker IP address, and then filter for all connection accepted and disconnect events, and then use `wc -l` to count the amount of occurences. In the end, I get this command:

`cat \[root\]/var/log/mongodb/mongod.log | grep "65.0.76.43" | grep -E "22944|22943" | wc -l`

### The attacker gained remote access after a series of brute‑force attempts. The attack likely exposed sensitive information, which enabled them to gain remote access. Based on the logs, when did the attacker successfully gain interactive hands-on remote access?

It seems like its time to pivot to `auth.log`. After skimming the contents, it seems that our attacker's IP address is present in the logs, signifying that they may not have taken the time to change it. So therefore, I filter the log for the attacker's IP address. 

Looking towards the bottom of the list, I see a line that starts with `Accepted keyboard-interactive/pam...from 65.0.76.43 port 46062 ssh2`, meaning that the attacker has successfully logged in. The time of this event is our most likely answer.

### Identify the exact command line the attacker used to execute an in‑memory script as part of their privilege‑escalation attempt.
The most immediate artifact to look for is the `.bash_history` of the compromised account. Very simple step, and it reveals our answer.

### The attacker was interested in a specific directory and also opened a Python web server, likely for exfiltration purposes. Which directory was the target?
Continuing with `.bash_history`, the attacker switches directories and starts a python web server there. The directory they switched to gives us the answer.