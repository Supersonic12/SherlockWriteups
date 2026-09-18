found these in chrome history:
* https://www.google.com/search?q=whoami&sourceid=chrome&ie=UTF-8whoami
* zRtpFguNo9kbMJD1BgDE1w-1760981468-1.0.1.1-do2_z48FQYPxqYpHmu1nervwbDkifMgiBFOaijZwAeggitlab.com/users/sign_in?__cf_chl_tk=9s6S_z9unFTWsI6CQE0U6zRtpFguNo9kbMJD1BgDE1w-1760981468-1.0.1.1-do2_z48FQYPxqYpHmu1nervwbDkifMgiBFOaijZwAegV
* https://gitlab.com/https://gitlab.com/users/sign_ingitlab.com/users/sign_in

NTUSER.dat artifacts:
* @mmres.dll in notification-mail
* there is bunch of game names in gameconfig registry
	* Run key has this:
		"C:\Users\T3M0\AppData\Local\Microsoft\OneDrive\OneDrive.exe" /background
	* RunOnce key has these:
		C:\Windows\system32\cmd.exe /q /c del /q "C:\Users\T3M0\AppData\Local\Microsoft\OneDrive\Update\OneDriveSetup.exe"
		C:\Windows\system32\cmd.exe /q /c del /q "C:\Users\T3M0\AppData\Local\Microsoft\OneDrive\StandaloneUpdater\OneDriveSetup.exe"
		C:\Windows\system32\cmd.exe /q /c rmdir /s /q "C:\Users\T3M0\AppData\Local\Microsoft\OneDrive\25.184.0921.0004"
			not that much of a suspicious
* RecentDocs 
	* ZIP:
		* creds_exfil_2025-10-01.zip
* Shell-Bags-1-Desktop: 
		* Payroll_Verification.cmd
		* ChatGPT Installer.exe

though i did mistake from the start i should have looked at chromes history file. SQLEcmd's maps being not compatible delayed the analysis but claude wrote a python script for me which queries history file in sqlite format. Here in downloads table i found the installed zip archive
https://8.222.205.174/scripts/Payroll_Update_January.zip
and now I'm looking at MFT table to see what has come to outside of zip archive in MFT I found zip file itself but couldn't find the extracted executable. After parsing $Extend\$J using MFTEcmd.exe and opening it in TimelineExplorer.exe I found **Payroll_Verification.cmd** 
Then ps.txt that is renamed as ps.ps1


in Powershell Operational logs:
* powershell -NoProfile -ExecutionPolicy Bypass -File "C:\Users\T3M0\Desktop\ps.ps1" -ProfileDir "C:\Users\T3M0\Desktop\SECRET" -Username "john.doe@example.com" -Password "SecretFromLab!"
	*  also there is multiple script bloggin 4104 events but i havent decoded them yet


First day I closed with only 5 flags found. Last one being certutil. I haven't recorded how did i record it, so i have forgotten. 
## Second Day

I started with analyzing access logs since powershell and certutil's prefetch didn't gave me any answers.
I quickly found that payroll_template and payroll_template[1].dat files has been installed. There I confirmed CertUtil and CryptoApi has been used. then I found exact internet resource in same log which was **http://8.222.205.174/update/payroll_template.dat** 
then task 7 was timestamp of first retrieval of external resource. And i saw there was multiple trial of downloads. using curl, mozilla firefox, certutil (cryptoapi) user-agents. i found the certutil's user-agent and success-code 200 and then took unix epoch timestamp number and converted it into human readable format and it was the answer.