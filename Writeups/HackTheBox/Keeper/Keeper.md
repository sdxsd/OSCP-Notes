# Keeper

## Nmap scan:
```
Nmap scan report for tickets.keeper.htb (10.129.17.187)
Host is up (0.017s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 35:39:d4:39:40:4b:1f:61:86:dd:7c:37:bb:4b:98:9e (ECDSA)
|_  256 1a:e9:72:be:8b:b1:05:d5:ef:fe:dd:80:d8:ef:c0:66 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-title: Login
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Writeup:
Port `80` and port `22` are open.
There is a request tracker instance available over the subdomain `tickets.keeper.htb`.
The version is `RT 4.4.4+dfsg-2ubuntu1` 
 
Login with default credentials works, thus `root:password`.
We find a user called `lnoorgaard` with a password given in the description: `Welcome2023!`.

These credentials provide access over `ssh`.

There we find a keepass dump file hidden in `RT30000.zip` in the home directory of `lnorgaard`
This along with a `.kdbx` file.

This dump file is vulnerable to `CVE-2023-32784`, and using a Rust PoC, I receive the following output:
- `●{', ,, -, :, =, A, I, M, ], _, `, c}dgrd med flde`

Throwing this into a search engine reveals:
- `Rødgrød Med Fløde`
Which when converted to lowercase opens the database.

From there we can use the Putty key found in the keepass database to login as `root`
