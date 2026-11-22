**Completion date:** 2026-05-11\
**Platform:** HackTheBox, [https://app.hackthebox.com/sherlocks/Brutus]\
**Skills and Tools Used:** Built-in linux tools, python

## Overview
Post-Investigation Report:\
The attacker, `[REDACTED IP]`, carried out a brute force attack to gain access to the `[REDACTED]` account and established a terminal session on `[REDACTED TIME] UTC`. Afterwards, the attacker gained persistence by creating a new user account under the name `[REDACTED]`. Afterwards, they downloaded `linper.sh` on the machine.

From HackTheBox:\
"In this Sherlock, you will familiarize yourself with Unix auth.log and wtmp logs. We'll explore a scenario where a Confluence server was brute-forced via its SSH service. After gaining access to the server, the attacker performed additional activities, which we can track using auth.log. Although auth.log is primarily used for brute-force analysis, we will delve into the full potential of this artifact in our investigation, including aspects of privilege escalation, persistence, and even some visibility into command execution."

## Methodology and Thinking
Upon extracing the zip file, it seems that we have three files.
1. auth.log
2. utmp.py
3. wtmp

Upon catting wtmp, I get gibbersh. utmp.py seems to be a tool to decode wtmp, so I will run it through and see what is outputted. Upon parsing wtmp, I get a human-readable file. I save this to a file called wtmp.txt.

Now for our first question: **Analyze the auth.log. What is the IP address used by the attacker to carry out a brute force attack?**

To identify a brute force attack, I will need to identify if there is a large burst of failed password attempts on ssh. To do this, I run the following command:\

`cat auth.log | grep -i "failed password"`

Upon doing so, I notice that all password failed attempts originate from one ip address, and all of the failed password attempts span two minutes. Therefore, I can be confident that this is the IP address running the brute force attack.

For our second question, **The bruteforce attempts were successful and attacker gained access to an account on the server. What is the username of the account?**

To figure this out, we can use our IP from the previous question and search for entries starting with "Accepted password". Therefore, I run this command:

`cat auth.log | grep "[REDACTED]" | grep -i "accepted"`

However, this reveals that two different accounts have been signed onto. The first one that appears is most likely the answer, but to be confident, I do some additional grepping to see which logins were due to brute force attacks. I run these two commands:

`cat auth.log | grep "[REDACTED]" | grep -i "[FIRST USER]"`

`cat auth.log | grep "[REDACTED]" | grep -i "[SECOND USER]"`

This reveals that the second user did not have any password failed attempts, whilst the first user has multiple. Therefore, it is most likely certain that the first user is the one to be compromised by the brute force attack.

For our third question, **Identify the UTC timestamp when the attacker logged in manually to the server and established a terminal session to carry out their objectives. The login time will be different than the authentication time, and can be found in the wtmp artifact.**

This time, I will use the parsed wtmp file to answer the question. I will first make the rows better to read by using `column -t`. Next, I will grep for the attacker IP address to trim down the rows. 

However, upon entering the time returned for root, it was incorrect. After a bit of investigating, I found out that the python code converts the time to localtime using time.localtime. After changing it to time.gmtime, I was able to get the correct answer. This was the command I used:

`cat wtmp.txt | column -t | grep "[REDACTED]"`

For our fourth question, **SSH login sessions are tracked and assigned a session number upon login. What is the session number assigned to the attacker's session for the user account from Question 2?**

First, I attempted to see if the sshd entry on auth.log revealed the session number. However, the session number was nowhere to be found. After doing research online, I learned that systemd-logind shows the session number instead. Therefore, I ran this command:

`cat auth.log | grep "systemd-logind"`

I find that there are two session numbers for root. One of the sessions last for less than one second, so I input the other one as my answer.

For our fifth question, **The attacker added a new user as part of their persistence strategy on the server and gave this new user account higher privileges. What is the name of this account?**

Auth.log will definitely track new user creation, so I simply ran this command:

`cat auth.log | grep "new user"`

For our sixth question, **What is the MITRE ATT&CK sub-technique ID used for persistence by creating a new account?**

This one is also fairly simple. After going to the MITRE ATT&CK website, under persistence there shows "Create Account". Expanding it shows the subtechnique.

For our seventh question, **What time did the attacker's first SSH session end according to auth.log?**

This one can be done by grepping for "session closed", and then grepping again for "sshd". This isolates the logs to only session closed events, and trims the log down further to show only sshd events, which is how the attacker gained initial access.

`cat auth.log | grep -i "session closed" | grep "sshd"`

The first entry is the session that ended after less than one second, so the second entry must be the correct answer.

For our final question, **The attacker logged into their backdoor account and utilized their higher privileges to download a script. What is the full command executed using sudo?**

Sudo events are typically also logged in auth.log, so I simply grepped for term "sudo". Here, it is possible to see the full command executed.

`cat auth.log | grep -i "sudo"`