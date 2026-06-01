# Kevin Writeup

## 192.168.173.45
```
PORT      STATE SERVICE       VERSION
80/tcp    open  http          GoAhead WebServer
| http-title: HP Power Manager
|_Requested resource was http://192.168.173.45/index.asp
|_http-server-header: GoAhead-Webs
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows 7 Ultimate N 7600 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server Microsoft Terminal Service
| ssl-cert: Subject: commonName=kevin
| Not valid before: 2026-03-25T12:12:47
|_Not valid after:  2026-09-24T12:12:47
|_ssl-date: 2026-03-26T12:16:00+00:00; +1s from scanner time.
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49158/tcp open  msrpc         Microsoft Windows RPC
49159/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: KEVIN; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: KEVIN, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:9e:b6:c6 (VMware)
| smb2-time: 
|   date: 2026-03-26T12:15:52
|_  start_date: 2026-03-26T12:13:33
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 7 Ultimate N 7600 (Windows 7 Ultimate N 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::-
|   Computer name: kevin
|   NetBIOS computer name: KEVIN\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-03-26T05:15:52-07:00
|_clock-skew: mean: 1h45m01s, deviation: 3h30m00s, median: 0s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```

## Writeup:
Creds for the HP Power Manager interface on port 80 are admin:admin.
The HP Power Manager version 4.2 (build 7) is vulnerable to CVE-2009-2685, exploiting this grants initial access.
Initial access is as System. The flag is found at C:\Users\Administrator\Desktop\Proof.txt
