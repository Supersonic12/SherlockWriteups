Medium Sherlock here. Woohoo, let's get started.
**Scenario**:
- With help from D.I. Lestrade, Holmes acquires logs from a compromised MSP connected to the city’s financial core. The MSP’s AI helpdesk bot looks to have been manipulated into leaking remote access keys - an old trick of Moriarty’s.

We have kape output artifacts, a kdbx file and a network dump in hand.
## Task 1
What was the IP address of the decommissioned machine used by the attacker to start a chat session with MSP-HELPDESK-AI?

I opened network dump on wireshark and filtered for http.request.method\=\=Post. Which showed multiple POST requests from 2 IP addresses. These IP addresses are 10.32.43.31 and 10.0.69.45. 

![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task1.png)
This request content made from 10.0.69.45 was asking for RMM creds which suggests malicious intent
Answer:10.0.69.45

## Task 2
What was the hostname of the decommissioned machine?


Here I filtered for attacker's ip address and bunch of protocols that might contain hostname of the victim machine. These protocols are DHCP, NetBIOS, mDNS,llmnr.
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task2.png)
Fortunately there is our answer.
Answer:WATSON-ALPHA-2
## Task 3
What was the first message the attacker sent to the AI chatbot?


![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task3.png)
When filtered for http POST request and attacker's ip address, The first request I see shows the first ever message of attacker to ai chatbot.
Answer:Hello Old Friend

## Task4
When did the attacker's prompt injection attack make MSP-HELPDESK-AI leak remote management tool info?


![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task4.png)
Answer:2025-08-19 12:02:06

## Task 5
What is the Remote management tool Device ID and password?


![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task5.png)When followed HTTP stream we can see, ai chatbot leaking rmm credentials.
Answer:565963039:CogWork_Central_97&65

## Task 6
What was the last message the attacker sent to MSP-HELPDESK-AI?


In same HTTP stream we can see attacker's last message.
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task6.png)
Answer:JM WILL BE BACK

## Task 7
When did the attacker remotely access Cogwork Central Workstation?


For this I looked at Kape output, specifically C:\Program Files\TeamViewer.
There was 2 files. Answer was in Connections_incoming.txt file.
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task7.png)
As we can see after RMM credentials leakage there is only one user that logged in to central workstation.
Answer:2025-08-20 09:58:25

## Task 8
What was the RMM Account name used by the attacker?

Answer is in the screenshot of Task 7.
Answer:James Moriarty

## Task 9
What was the machine's internal IP address from which the attacker connected?

For this I looked at the TeamViewer15_Logfile.log. I filtered it for ip addresses and "James Moriarty" for easy review using this command:
`cat .\TeamViewer15_Logfile.log | Select-String -Pattern '(^2025\/08\/20 10).+((\d{1,3}\.){3}\d{1,3}|James Moriarty)'`

This command basically prints what is inside the .log file and then filters it for the attack's day and hour and shows only lines with ip address or "James Moriarty" literal string.

![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task9.png)

And here is our output. In this output there is multiple IP addresses. But we need the line with "UDPv4: punch received", which shows an internal IP address. 
Answer: 192.168.69.213

## Task 10
The attacker brought some tools to the compromised workstation to achieve its objectives. Under which path were these tools staged?


in TeamViewer15_Logfile.log in the same connection that we looked at task 9 there is bunch of files being downloaded to system. 

![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task10.png)

Answer:C:\Windows\Temp\safe

## Task 11
The attacker staged a browser credential harvesting tool on the compromised system. How long did this tool run before it was terminated? (Provide your answer in milliseconds, rounded to the nearest thousand)

I have already seen in Task 10, that there is bunch of files being downloaded to system. One of these files were webbrowserpassview.zip which was downloaded to C:\Windows\Temp\safe via TeamViewer. 

![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task11.png)

In C:\$Extend\$J there should be records about this file. I quickly turned USNJRNL records into csv using MFTEcmd.exe(EZ tools) and loaded it into timeline explorer. When filtere for name of the file without extension there was bunch of records about the tool used.
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task11-2.png)
Here I saw that webbrowserpassview.zip was decompressed to WebBrowserPassView.exe. Though here there wasn't any execution records. This were where I really was stuck. I really looked thoroughly for this task. I looked at all event logs using get-winevent and (evtxcmd.exe and timeline explorer) from Eric Zimmerman tools, there wasn't anything related to it inside them. Actually none of the executed file records was in event logs. Then I checked NTUSER.dat registry hives(in C:\Users\Cogwork_Admin\Default, C:\Windows\ServiceProfiles\NetworkService and C:\Windows\ServiceProfiles\LocalService) , but i guess i have missed it the first time. Then I checked the TeamViewer15_Logfile.log again thorougly for the exe file. there wasn't anything related to it at all. I checked both powershell logs in C:\Windows\System32\winevt.(Microsoft-Windows-Powershell%4Operational.evtx and Windows Powershell.evtx). Again there wasn't anything related to specific exe file. Then I checked registry records again. It was in front of me all along in the C:\Users\Cogwork_Admin\Default\NTUSER.dat's available bookmarks that is shown on Registry Explorer(EZ tools). 

![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task11-3.png)
In UserAssist there are usually last executed executables. And I have completely missed it. here we see the specified executable was ran for 8 seconds before it was terminated. 
Answer:8000

## Task 12
The attacker executed a OS Credential dumping tool on the system. When was the tool executed?

There was a record of mimikatz.exe file being downloaded to the central workstation on the TeamViewer15_Logfile.log. 
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task12-2.png)
Since there wasn't any record about mimikatz.exe(downloaded via TeamViewer) in UserAssist registry list, I was fixated on CredHistView.exe. 
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task12-3.png)
Because of this it took me really huge amount to find the answer. So much, that I have found answers of questions till task 16 and returned back to this task finding answer. Each time a process executed there is going to be created prefetch record of it on the system. Thinking about this I filtered for prefetch files that has "mimikatz" in it on \$Extend\\$J's csv formatted version in Timeline Explorer. The first prefetch file's creation time was our answer.
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task12.png)
Answer:2025-08-20 10:07:08
## Task 13
The attacker exfiltrated multiple sensitive files. When did the exfiltration start? (UTC)


I already saw bunch of files in NTUSER.dat's Recent Docs registry. 
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task13.png)
I just searched their names in TeamViewer15-Logfile.log.
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task13-2.png)
Here time shows up as 2025-08-20 11:12:07 but it is GMT +1. 
Answer:2025-08-20 10:12:07

## Task 14
Before exfiltration, several files were moved to the staged folder. When was the Heisen-9 facility backup database moved to the staged folder for exfiltration?


For this I used timeline explorer again filtering for the "Heisen" word on the csv version of USNJRNL. It showed a file with .kdbx extension. 
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task14.png)
First 2 files Update Reason was ObjectIdChange but the highlighted event's Update Reason was FileCreate which happens also when a file is moved to another directory.
Answer:2025-08-20 10:11:09

## Task 15
When did the attacker access and read a txt file, which was probably the output of one of the tools they brought, due to the naming convention of the file?

In Recent Docs registry list there was a file named dump.txt. The last open time was 2025-08-20 10:08:14. In UserAssist registry list there was notepad.exe which last time executed at 2025-08-20 10:08:14.
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task15.png)
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task15-2.png)
Surprisingly this wasn't the answer. I have lost notorious time for this strange problem. At the end I looked at the \$Extend/\$J/ using Timeline Explorer and looked specifically filtering for dump.txt.
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task15-3.png)
First 5 record and second 5 record had different Parent Sequence number And Entry Number. I still don't exactly know why but the second different timestamp was our answer. 
Answer: 2025-08-20 10:08:06.

## Task 16
The attacker created a persistence mechanism on the workstation. When was the persistence setup?


These question was hard for me genuinely. I looked for Run, RunOnce keys first. The persistence weren't setup with it. I looked at other keys for clues what possible key has been used but there was no outcome. While looking for the answer of Task 19 I checked Microsoft/Windows/Winlogon/ for possible login details, and I saw the JM.exe as second executable to Userinit registry key, which was downloaded to Central Workstation using  TeamViewer.
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task16.png)
I should have guessed there should have been a use for JM.exe. That was entirely my misjudgement.
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task16-2.png)
This is a legit technique to keep persistence to compromised system. I definitely will keep this in mind. 
Answer:2025-08-20 10:13:57
## Task 17
What is the MITRE ID of the persistence subtechnique?

![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task17.png)

## Task 18
When did the malicious RMM session end?

For this question I just looked through TeamViewer log file. 
![](Holmes%202025%202;%20The%20Watchman's%20Residue-img/task18.png)
As we can see at the 2025/08/20 11:14:27 GMT +1 the ReceivedEndSession function is called. Which is the end of malicious RMM session. So in UTC the hour is going to be 10:14:27.
Answer: 2025-08-20 10:14:27

## Task 19
The attacker found a password from exfiltrated files, allowing him to move laterally further into CogWork-1 infrastructure. What are the credentials for Heisen-9-WS-6?

Unfortunately I have already spent 20 hours for this sherlock. I guess my knowledge wasn't enough. I will look at the official writeup for last question. 

Well I opened and read the official writeup and never been this much disappointed that answer was just brute force with rockyou.txt. I knew it was possible, but I thought there should be some record of key file for kdbx file or password for it. Anyway this itself was a lesson that always start from the basic way to not lose time.
Answer:Werni:Quantum1!

Thanks for reading.
