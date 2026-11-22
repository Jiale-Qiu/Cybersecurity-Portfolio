# Homelab Entry #6: Improving Endpoint Visibility (WIP)

Although Elastic *technically* works, there is still much work to be done. Sysmon isn't configured correctly, and there are lots of visibility gaps. Therefore, I will try to identify log sources and information that I will need to detect relevant attacks on my Juice Shop endpoint.

To avoid high complexity dealing with privacy and data laws, they will not be considered here. First, I'll create a table with ATT&CK techniques I want to focus on and the data sources needed for them. For simplicity's sake, I will only choose one technique per tactic.

| ATT&CK Technique | Description | Sources Needed |
|------------------|-------------|----------------|
| T1190 | Initial Access: Exploit Public-Facing Application | Webserver Access Logs (Visibility Gap: POST bodies not visible) |
| T1059 | Execution: Command and Scripting Interpreter | Sysmon (Event Code 1), PowerShell Operational Log (Event Code 4104) |
| T1112 | Persistence: Modify Registry | Sysmon (Event Code 12,13,14) |
| T1685.005 | Defense Impairment: Clear Windows Event Logs | Windows Security Logs (Event Code 1102), Windows System Logs (Event Code 104) |
| T1082 | Discovery: System Information Discovery | Sysmon (Event Code 1) |
| T1560 | Collection: Archive Collected Data | Sysmon (Event Code 1) |
| T1048 | Exfiltration: Exfiltration over Alternative Protocol - Exfiltration over Unencrypted Non-C2 protocol (HTTP) | Network Logs, Sysmon (Event Code 3) |
## T1190 || Initial Access: Exploit Public-Facing Application 
### Overview <!-- Goals and Necessary Data Sources -->
The SIEM currently has zero visibility on the Juice Shop, posing a significant risk as responding to incidents and detecting attackers will be heavily delayed and hindered without proper visibility. The most immediate action is to ingest the Juice Shop's access logs into the SIEM.

### Current Visibility
**Missing 🔴**
I do not have visiblity in webserver logs. After clicking around on the Juice Shop, a broad search for GET requests in the SIEM produces zero results.

### Configuration Changes
The Juice Shop records GET, POST, and other requests in its access logs. These logs can be ingested into the SIEM using the Elastic Apache integration. I aim the integration towards the relevant Juice Shop access logs.

### Simulation
GET requests seem to now get logged. Awesome. This shows us that the events are coming in normally.

![alt text](Images/6/image3.png)

Lets see if it can also catch a POST request. I send a login request, and the respective event appears as expected. However, I note a gap in visiblity—the contents of the POST request are not logged.

![alt text](Images/6/image4.png)

Now, lets test some common attacks against the Juice Shop. First, I'll test a simple (successful) brute force attack against the login page of the webserver using ffuf with the Attacker VM. Looking back at Elastic SIEM, I immediately see a massive spike in the amount of logs.

![alt text](Images/6/image6.png)

Now, lets see if I have sufficient data to triage this attack if it were a real one. Looking through the timeframe from Nov 9, 2026 @ 01:25:50 to Nov 9, 2026 @ 01:26:00 in the logs, I observe:
- the most common IP address, seen in the source.ip field with 70 records, is `10.0.0.22`. This is the auto-assigned IP address for my Attacker VM.
- the targeted URL, seen in the url.path field, was `/rest/user/login`. The URL aligns with the brute force attack.
- the brute force is fast—the 70 requests were generated within the span of 10 seconds, so approximately 7 requests a second.
- the tool used was FFUF, seen in the user_agent.original field as `Fuzz Faster U Fool v2.1.0-dev`.
- the attacker was successful, as the attacker had a successful sign-in, seen through an HTTP response code 200 (success).

The main information of a brute force attack is collectible, and thus the visibility is acceptable against brute force attacks.

### Conclusion

The visibility of the Juice Shop has been significantly improved with the ingestions of access logs into the SIEM. It provides important surface-level data for most operations. However, access logs are insufficient for determining POST request bodies, and without a reverse proxy or another solution, a login bypass via SQL injection may look the same as a regular login.

## T1059 || Execution: Command and Scripting Interpreter

### Overview <!-- Goals and Necessary Data Sources -->
Although Sysmon is installed, I do not know the full extent of its current configuration. Many log sources are in their default configuration, which are typically less than ideal. In order to detect T1059 and properly investigate it, I will need data from multiple sources, primarily Sysmon and PowerShell logs.

### Current Visibility
**Needs Tuning 🟣**\
Testing visibility against T1059 primarily involves using command and scripting interpreters to execute malicious code. On a Windows endpoint, the most relevant command and scripting interpreters would include the Command Prompt, PowerShell, and installed programming languages such as Python.

**Testing PowerShell:**\
I want to simulate an attacker using `cmd.exe` to execute a Base64-encoded PowerShell command which subsequently runs an executable—Notepad, in this situation. I can use the following command in `cmd.exe`:

`powershell.exe -EncodedCommand "bgBvAHQAZQBwAGEAZAA="`

After executing this, I should have at least three events pop up:
1. `cmd.exe` opening
2. `powershell.exe` opening
3. `notepad.exe` opening

After executing it I look for each associated event.

EventCode 1 gets correctly logged, allowing me to see both the `cmd.exe` and `powershell.exe` events with sufficient information about the process, the user, and its parent.

![alt text](Images/6/image7.png)

However, it appears that to avoid noise, my current configuration of Sysmon did not log the `notepad.exe` opening event. Acceptable, but I will need to take a close look at my Sysmon config to see exactly where my visibility gaps will be with Sysmon.

What concerns me more though is the amount of unnecessary noise my current configuration is generating. Executing the encoded command created 12 Events, most of which are unnecessary.

Primarily, there is an excessive amount of events with Event Code 600, which despite potentially being useful in specific cases, does not provide enough extra data to justify its presence in the logs. In its place however, I will need to log PowerShell Script Block Logging in order, which generates Event Code 4104. This provides high-fidelity information on commands executed in PowerShell—information that Event Code 600 wouldn't tell me.

![alt text](Images/6/image8.png)

Thus, I have three things to do:
1. Stop ingesting event code 600
2. Enable Script Block Logging
3. Check Sysmon configuration

### Configuration Changes
I can't disable event code 600 without disabling the entire log channel, so I'll have to exclude it from ingestion in the Windows integration in Elastic. Alongside code 600, I also exclude code 403 and 800 to cut down on storage consumption, as Script Block Logging and Sysmon should be enough for my purposes.

![alt text](Images/6/image9.png)

Currently, Powershell only does Script Block Logging for commands it deems suspicious. An attacker can fairly easily bypass this, so I want it to log every event. For full visibility, I will need to make sure both Event Code 4103 and 4104 are active. Both Event Codes work together simultaneously for maximum coverage.

Using the help of AI, I'll need to run three different sets of commands. Of course, when it comes to commands its best to know what each does.

```
$sb = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"
New-Item -Path $sb -Force | Out-Null
Set-ItemProperty -Path $sb -Name EnableScriptBlockLogging -Value 1 -Type DWord
```

This first set of commands saves a registry path to a variable `sb`, and creates a new registry key with it. Afterwards, it adds a new value named `EnableScriptBlockLogging` and sets it to 1 (enables it).

```
$ml = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging"
New-Item -Path "$ml\ModuleNames" -Force | Out-Null
Set-ItemProperty -Path $ml -Name EnableModuleLogging -Value 1 -Type DWord
Set-ItemProperty -Path "$ml\ModuleNames" -Name "*" -Value "*" -Type String
```

This second set of commands also does something similar, except it forms two values.

```
wevtutil sl "Microsoft-Windows-PowerShell/Operational" /ms:536870912   # 512 MB
```

Finally, this third command increases the size of the Powershell log file, as Event Code 4103 and 4104 use a fair bit of a space.

After executing the commands, Script Block Logging and Module Logging should be fully enabled.

Finally, the last thing I want to do is get a better Sysmon configuration. Sysmon's default configuration is not great, so I'll download a config file from [online](https://github.com/olafhartong/sysmon-modular/releases/tag/configs-a45dedb13a43) to use, although I won't tune it for the environment as of now.


### Simulation
Finally, to test if the changes worked. This time, I'll run two commands: the same one as previously, and also a more benign command.

`powershell.exe -EncodedCommand "bgBvAHQAZQBwAGEAZAA="`

Unfortunately, event Code 600 and similar continued to appear despite the agent reporting itself is up-to-date with the newest version. After a half-hour of debugging, I found the root cause: the agent's tracked policy revision number was greater than the fleet server's, meaning the agent believed that it had a newer version than the server, causing it to refuse updates. While quirky, the fix was fairly simple: repeatedly make small changes to the fleet server's policy until the revision number exceeded that of the agent's, which finally led the agent to realize it's outdated and prompting an update.

After the fixes, the event codes 403, 600, and 800 no longer appear. Compared to the previous 12 events, this time only 5 events were logged, meaning that my changes are working as intended.

![alt text](Images/6/image10.png)

![alt text](Images/6/image12.png)

### Conclusion
With these changes, I have greatly improved visibility in PowerShell and have also cut out a fair amount of noise.

## T1112 || Persistence: Modify Registry

### Overview <!-- Goals and Necessary Data Sources -->
Modifying the registry is a common way to establish persistence on a machine, so ensuring that my SIEM can detect it is imperative.

### Current Visibility
**Acceptable 🟢**\
The most straightforward way to implant persistence is through the registry run keys, which will be our main target for this simulation. Using the windows `cmd.exe`, I will use `reg.exe` to plant a registry key that executes `notepad.exe` on startup.

I run the following command to set up the persistence mechanism:
```
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v "Secret Persistence" /t REG_SZ /d "\"C:\Windows\System32\notepad.exe\"" /f
```

Four events appear, two of which are from Sysmon, and the other two I'm assuming come from Elastic EDR. From Sysmon's event code 13, I can find a deep amount of information for the added registry key, including the name and location of the key (HKU\\...\Run\Secret Persistence), details of the key (C:\Windows\System32\notepad.exe), and even the process ID (4772). There is enough visibility already.

![alt text](Images/6/image13.png)

### Conclusion
No changes are needed.

## T1685.005 || Defense Impairment: Clear Windows Event Logs

### Overview <!-- Goals and Necessary Data Sources -->

### Current Visibility

In order to test visibility against this, I will clear the Windows System and Security logs using the command `Clear-EventLog -LogName System, Security`. Unforunately, once I run it, no results return.

![alt text](Images/6/image14.png)

### Configuration Changes

With some research, it appears that the `System` integration is missing from my Elastic policy. The System integration logs the essential system logs, including `Application`, `System`, and `Security`. It's surprising I didn't have this; this shows how important it is to thoroughly check visibility. I add the integration into my policy and save the changes.

### Simulation

Running the same exact command as before
![alt text](Images/6/image16.png)

### Conclusion
