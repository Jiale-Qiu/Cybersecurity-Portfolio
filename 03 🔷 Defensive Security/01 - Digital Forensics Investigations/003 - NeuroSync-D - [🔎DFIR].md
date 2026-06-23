**Completion date:** 2026-06-22\
**Platform:** HackTheBox, [https://app.hackthebox.com/sherlocks/NeuroSync-D]\
**Skills and Tools Used:** Built-in linux tools, Google

## Overview
Post-Investigation Report:\
The attacker first exploited CVE-####-##### on the vulnerable next.js server to bypass authentication. The attacker then used an SSRF attack to enumerate the compromised host and discover an internal API. After discovering the internal API, the attacker brute forced for directories and discovered an endpoint with an LFI vulnerability. Using sensitive information gathered from the LFI vulnerability, the attacker gained RCE through a compromise of Redis.

From HackTheBox:\
"NeuroSync™ is a leading suite of products focusing on developing cutting edge medical BCI devices, designed by the Korosaki Coorporaton. Recently, an APT group targeted them and was able to infiltrate their infrastructure and is now moving laterally to compromise more systems. It appears that they have even managed to hijack a large number of online devices by exploiting an N-day vulnerability. Your task is to find out how they were able to compromise the infrastructure and understand how to secure it."

## Methodology and Thinking

### Q1: What version of Next.js is the application using?
Upon extracting the `.zip` file, I wanted to understand what each file contained. Therefore, I executed the command `head *`. Here, I find that a file `interface.log` contains information on the next.js version as well as its interface on the network. Here, I get my answer.

![screenshot](screenshots/003/Q1+2.png)

### Q2: What local port is the Next.js-based application running on?

I found this answer in the same location as the previous question.

### Q3: A critical Next.js vulnerability was released in March 2025, and this version appears to be affected. What is the CVE identifier for this vulnerability?

This will require research on google. Thus, I used this search query: `"march" 2025 next.js 15.1.0 cve`. The first link provided me the answer i needed.

### Q4: The attacker tried to enumerate some static files that are typically available in the Next.js framework, most likely to retrieve its version. What is the first file he could get?

It is time to pivot to the webserver logs. I read `access.log` and found that every single event came from only one IP address, and that the whole log file was relatively short. Therefore, I could simply look at the first few entries. It appears that the attacker attempted to access webserver files to fingerprint it better, but was met with 404 not found. A few lines down the attacker finds an existing file and recieves code 200, meaning that they are able to retrieve the file.

![screenshot](screenshots/003/Q4.png)

### Q5: Then the attacker appears to have found an endpoint that is potentially affected by the previously identified vulnerability. What is that endpoint?

After finishing enumerating and fingerprinting the web server, a very long series of PUT and GET requests were sent by the attacker, all on the same endpoint (and the only endpoint that is not webserver enumeration). This is where I found my answer.

### Q6: How many requests to this endpoint have resulted in an "Unauthorized" response?

This is fairly simple. I just have to count the amount of 401 responses. I simply filter the logs for 401 requests and count the number of lines.

### Q7: When is a successful response received from the vulnerable endpoint, meaning that the middleware has been bypassed?

This one is also fairly simple. Once authentication has been bypassed, next.js will return HTTP 200 OK. I find the first instance of this and note the timestamp as my answer.

### Q8: Given the previous failed requests, what will most likely be the final value for the vulnerable header used to exploit the vulnerability and bypass the middleware?

This will probably need research on Google. I search the query `CVE-2025-29927 poc` and read this article: [https://securitylabs.datadoghq.com/articles/nextjs-middleware-auth-bypass/](https://securitylabs.datadoghq.com/articles/nextjs-middleware-auth-bypass/). I find out that the CVE works by including a header in the client's response, tricking next.js into believing the request came from the middleware. However, in this vulnerable version of next.js, an addition was made to the code to prevent infinite recursive loops. If the request recurses too long, the middlware is skipped, bypassing authentication. Therefore, by providing the header with enough middleware calls, the exploit can be done. The POC can be seen in the website.

Or, you could read `interface.log` and also find the headers the attacker included in their attack. Both methods work.

### Q9: The attacker chained the vulnerability with an SSRF attack, which allowed them to perform an internal port scan and discover an internal API. On which port is the API accessible?

Looking at the files again, I notice `data-api.log`, which most likely logs data about the internal API. Upon reading it, the answer can be found.

![screenshot](screenshots/003/Q6.png)

### Q10: After the port scan, the attacker starts a brute-force attack to find some vulnerable endpoints in the previously identified API. Which vulnerable endpoint was found?

Looking further down the logs, it is seen that the attacker is attempting to exploit an LFI vuln. The directory was enumerated in the brute force attack, so this must be the answer.

![screenshot](screenshots/003/Q10.png)

### Q11: When the vulnerable endpoint found was used maliciously for the first time?

We have already identified the attack vector, so we just have to find the first occurance of it. This is fairly simple.

### Q12: What is the attack name the endpoint is vulnerable to?

We have identified the attack vector already in Q10.

### Q13: What is the name of the file that was targeted the last time the vulnerable endpoint was exploited?

Same artifacts to look at. Pretty much the same methodology as the previous questions.

### Q14: Finally, the attacker uses the sensitive information obtained earlier to create a special command that allows them to perform Redis injection and gain RCE on the system. What is the command string?

The format of the question prompts to use `redis.log`. Immediately it is visible what the special command is.

![screenshot](screenshots/003/Q14.png)

### Q14: Finally, the attacker uses the sensitive information obtained earlier to create a special command that allows them to perform Redis injection and gain RCE on the system. What is the command string?

The format of the question prompts to use `redis.log`. Immediately it is visible what the special command is.

### Q15: Once decoded, what is the command?

For this final question, it seems that part of the string fits the Base64 alphabet. Once decoding, I get my final answer and complete the sherlock.