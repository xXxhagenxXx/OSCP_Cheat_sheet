# Reconnaisance

## theharvester
- -b is a data source
`theharvester -d <domain> -b google` 

- Forward Lookup Bruteforce
` for word in $(cat list.txt) | do host $word.xxxx.com; done `

- Reverse Lookup Brute Force
`for ip in $(seq 50 100); do host xx.xxx.xxx.$ip; dne | grep -v "not found"` 
- DNS Zone Transfer
1. Look for  ns of a domain.
`host -t ns <domain>`
2. Iterate which namserver allows to do zone transfers.
`host -l <domain> <ns1.domain>`

- Subdomain Enumeration
- -D is the subdomains text string -t means bruteforcing
`dnsrecon -d megacorpone.com -D ~/list.txt -t brt`

- Netbios Name Enumeration
`sudo nbtscan -r 10.11.1.0/24`