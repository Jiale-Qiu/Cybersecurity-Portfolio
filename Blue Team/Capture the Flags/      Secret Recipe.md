# THM CTF: Secret Recipe

**Completion date**: 10/2/2025\
**Platform:** TryHackMe, [https://tryhackme.com/room/registry4n6]\
**Skills and Tools Used:** Registry Forensics on Windows, Mark Zimmerman's EZTools

## Preface
An IT guy is suspected to have stolen a top secret coffee recipe and downloaded it on their computer. Using registry forensics, the goal here is to examine the computer

### Q1: What is the computer name of the machine found in the registry?

Using the provided registry cheatsheet, I go to `SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName`. Here, I get my answer.

![](Screenshots/secretrecipe/image19.png)

### Q2: When was the Administrator account created on this machine?

I go to `SAM\Domains\Account\Users`. Important information about users such as ID and name are stored here. 

![](Screenshots/secretrecipe/image.png)
![](Screenshots/secretrecipe/image2.png)

### Q3: What is the RID associated with the Administrator account?

Like before, important information about users are store in `SAM\Domains\Account\Users`. Looking at the row with the username "Administrator", I retrieved my answer from the User ID column. 

![](Screenshots/secretrecipe/image1.png)

### Q4: How many user accounts were observed on this machine?
Inside the Users folder, a list of all the account's hexadecimal ID is shown. The zeros at the beginning is padding, so if we convert the hex `1F4` to decimal, we get ID `500`, which is the administrator account. Anyways, I count the values inside the folder and get my answer.

![](Screenshots/secretrecipe/image3.png)

### Q5: There seems to be a suspicious account created as a backdoor with RID 1013. What is the account name?

Looking in the `Names` folder right under the `Users` folder, The names of accounts are listed instead of their ID. 

![](Screenshots/secretrecipe/image4.png)

### Q6+7: What is the VPN connection this host connected to? When was the first VPN connection observed?

I needed a hint for this one, but it told me to look inside NetworkList, and here I found both answers.

![](Screenshots/secretrecipe/image5.png)

### Q8: There were three shared folders observed on his machine. What is the path of the third share?

Using a quick google search, I learned that network shares and stuff are stored at `SYSTEM\CurrentControlSet\Services\LanmanServer\Shares`. Here I got my answer

![](Screenshots/secretrecipe/image6.png)

### Q9: What is the last DHCP IP assigned to this host?
I checked network interfaces and past networks located at `SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces`.

![](Screenshots/secretrecipe/image7.png)

### Q10: The suspect seems to have accessed a file containing the secret coffee recipe. What is the name of the file?

I looked at `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` as this registry key stores recent files.
![](Screenshots/secretrecipe/image8.png)

### Q11: The suspect executed multiple commands using the Run window. What command was used to enumerate the network interfaces?

Up until now I forgot that there was a search box. So from here on out, I mostly just searched for all of the registry keys. For this question, there is a registry key called `runmru` which stores recently run commands in the run box.

![](Screenshots/secretrecipe/image9.png)

### Q12: The user searched for a network utility tool to transfer files using the file explorer. What is the name of that tool?

The WordWheelQuery registry key usually stores search queries in the file explorer. I identified network tool in question immediately, as I have used it for reverse shells in offensive security CTFs.

![](Screenshots/secretrecipe/image10.png)
![](Screenshots/secretrecipe/image11.png)

### Q13: What is the recent text file opened by the suspect?

Seems like we're going back to RecentDocs

![](Screenshots/secretrecipe/image8.png)

### Q14: How many times was PowerShell executed on this host?

Appcompatcache (SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache) stores good evidence of execution, so I went there first. From there, I searched up "Powershell"

![](Screenshots/secretrecipe/image13.png)
![](Screenshots/secretrecipe/image14.png)
![](Screenshots/secretrecipe/image15.png)

### Q15: The suspect also executed a network monitoring tool. What is the name of the tool? For how many seconds was ProtonVPN executed? What is the full path from which everything.exe was executed?

Although Appcompatcache would probably have this, I found it to have too many events for me to look through. I decided to use the UserAssist registry key instead. Here I found my answers.

![](Screenshots/secretrecipe/image16.png)
![](Screenshots/secretrecipe/image17.png)
![](Screenshots/secretrecipe/image18.png)
![](Screenshots/secretrecipe/image19.png)