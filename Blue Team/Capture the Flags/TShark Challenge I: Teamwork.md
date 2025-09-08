# THM CTF: Summit
_Note: CTF Spoilers ahead!_

**Completion date**: 9/7/2025\
**Platform:** TryHackMe, [https://tryhackme.com/room/tsharkchallengesone]\
**Skills and Tools Used:** TShark, VirusTotal, Network Packet Analysis

## Preface
The objective here is to analyze a packet capture file to inspect a suspicious domain that has been found.

## Question 1: What is the full URL of the malicious/suspicious domain address?
I first get a basic idea of what the packet capture looks like by reading it normally with TShark.\
Next, I filter the packets for HTTP requests and attempt to identify any abnormalities.\
Eventually, I find a GET request to a suspicious file. I use the verbose flag (-V) to see the packet's details and find the domain address.

## Question 2: When was the URL of the malicious/suspicious domain address first submitted to VirusTotal?
I searched the URL in VirusTotal and used the details panel to identify when it was first submitted

## Question 3: Which known service was the domain trying to impersonate?
The name of the service was directly in the name of the suspicious URI.

## Question 4: What is the IP address of the malicious domain?
I take a look at the destination IP of the abnormal packet.

## Question 5: What is the email address that was used?
I filter the packets by HTTP POST requests and use the verbose parameter to identify the email used, finishing the CTF.
