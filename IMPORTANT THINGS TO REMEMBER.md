# NOTE


1. In privilege escalation, check if we can poison imported libraries for binaries and scripts that are run by root.
2. If you can't make the script work always use the  virtualenv.
```
python3 -m pip install virtualenv
python3 -m virtualenv impkt
source impkt/bin/activate
(or on windows .\impkt\Scripts\activate.ps1)
pip install git+https://github.com/SecureAuthCorp/impacket

```
3. 
``
5. For foothold, if there are only php files such as login,home,logout,profile etc., look for directory traversal, LFI and RFI.
7. Look for /etc/knockd.conf when directory traversal, LFI or RFI exist.
8. After getting low priv shell, check for config.php, databases, sql file for user credentials.
9. Comparing files use comm <fileA> <fileB>
10. ALWAYS RUN MSFCONSOLE to SUDO for portfwd purposes.
11. In a local file inclusion always try to read the disable function under php.ini, e.g. (/etc/php.ini) or (/etc/php/7.2/apache2/php.ini)
	__*NOTE: backticks on php can execute commands(e.g. echo `whoami`)*__
8. test for web apps:
	- < >
	- ' 
	- "
	- { }
    - ;
9. Execution of the powershell script after compromising the server.
```powershell -ExecutionPolicy Bypass -File <file.ps1>```
	
10. Wordpress Exploitation
	
`wpscan --url http://10.11.1.234/ --enumerate u`
`wpscan --url http://10.11.1.234/ --passwords /usr/share/wordlists/rockyou.txt`	
`http://10.11.1.234/wp-content/themes/twentyeleven/404.php?cmd=id`
	
# TOOLS
1. tomcatWarDeployer
2. Drupalgeddon2
	
	
# Look for kitrap0d exploit for windows x86 kernel exploit
	
	
UAC BYPASS
[“Fileless” UAC Bypass Using eventvwr.exe and Registry Hijacking | enigma0x3](https://enigma0x3.net/2016/08/15/fileless-uac-bypass-using-eventvwr-exe-and-registry-hijacking/)
	
[Bypassing default UAC settings manually | Ivan's IT learning blog (wordpress.com)](https://ivanitlearning.wordpress.com/2019/07/07/bypassing-default-uac-settings-manually/)
	
[Powershell \*real\* UAC Bypass - Bash Bunny - Hak5 Forums](https://forums.hak5.org/topic/45439-powershell-real-uac-bypass/)
	
	
	

   
