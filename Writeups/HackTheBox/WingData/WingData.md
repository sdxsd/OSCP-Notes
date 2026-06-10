# WingData

## Nmap scan:
```
Nmap scan report for wingdata.htb (10.129.244.106)
Host is up (0.018s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey: 
|   256 a1:fa:95:8b:d7:56:03:85:e4:45:c9:c7:1e:ba:28:3b (ECDSA)
|_  256 9c:ba:21:1a:97:2f:3a:64:73:c1:4c:1d:ce:65:7a:2f (ED25519)
80/tcp open  http    Apache httpd 2.4.66
|_http-server-header: Apache/2.4.66 (Debian)
|_http-title: WingData Solutions
Service Info: Host: localhost; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Writeup:
I start with an nmap scan, revealing the following open ports:
- `22 ssh`
- `80 http`

I then do a quick directory enumeration scan with gobuster, which produces the following results:
```
.htaccess            (Status: 403) [Size: 317]
.htpasswd            (Status: 403) [Size: 317]
assets               (Status: 301) [Size: 353] [--> http://wingdata.htb/assets/]
server-status        (Status: 403) [Size: 317]
vendor               (Status: 301) [Size: 353] [--> http://wingdata.htb/vendor/]
```

Navigating to the main page I'm met with a largely static site.
However, I do notice a link to a client portal, which reveals a new subdomain: `ftp.wingdata.htb`.
I quickly add this to my `/etc/hosts` and access the page.
This reveals that `Wing FTP Server v7.4.3` is running, which is vulnerable to `CVE-2025-47812`.
This vulnerability allows for unauthenticated remote code execution. Nice!

After gaining a reverse shell as `wingftp`, I take a look around.
I locate a directory called `Data/1/users`. This seems to contain user informations, including password hashes!
```
d67f86152e5c4df1b0ac4a18d3ca4a89c1b12e6b748ed71d01aeb92341927bca
c1f14672feec3bba27231048271fcdcddeb9d75ef79f6889139aa78c9d398f10
a70221f33a51dca76dfd46c17ab17116a97823caf40aeecfbc611cae47421b03
5916c7481fa2f20bd86f4bdb900f0342359ec19a77b7e3ae118f3b5d0d3334ca
32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca
```

After finding the salt, `wingftp` I was able to crack the hash using `hashcat`.
- `hashcat -m 1410 hash ~/SecLists/Passwords/Leaked-Databases/rockyou.txt`

Which revealed the following:
- `32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca:WingFTP:!#7Blushing^*Bride5`

With this password belonging to the user `wacky` I gain access over `ssh`.
Quickly running `sudo -l`, I see that I can run the following with `root` privileges:
- `/usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py *`

The version of python running is vulnerable to`CVE-2025-4517`, which allows for arbitrary file write with maliciously crafted tarballs.

I find a public PoC online (see submodule), transfer it to the victim machine and run the script targeting the malicious tarball, which immediately grants me `root` access.
