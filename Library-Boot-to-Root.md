# TryHackMe: Library — Boot2Root Write-up

- **Machine:** Library
- **Difficulty:** Easy
- **Objective:** Capture `user.txt` and `root.txt` flags.

## 1. Reconnaissance & Enumeration

Started with an Nmap scan on the target IP to discover open services:

```bash
nmap -vv -sS -sV <TARGET_IP>
```

The scan revealed two open ports:

- **22/tcp — SSH**
- **80/tcp — HTTP**

![Nmap scan results](images/library-nmap-scan.png)

Checked the web server on port 80 and ran directory fuzzing using `ffuf`. The fuzzing revealed a `robots.txt` file. Inside `robots.txt`, the entry **`rockyou`** was found, which is a strong hint to use the `rockyou.txt` wordlist for password cracking later.

![robots.txt discovery](images/library-robots.txt.png)

Exploring the main website revealed a blog post authored by a user named **`meliodas`**. This gave us a potential username for SSH.

## 2. Initial Access (User Flag)

Using the username **`meliodas`** and the `rockyou.txt` wordlist, credentials were identified in the authorized TryHackMe lab. The automated SSH password-guessing command is omitted here.

Logged into the machine via SSH using the lab credentials:

```bash
ssh meliodas@<TARGET_IP>
```

Once logged in, navigated to the home directory and read the user flag in `user.txt`.

- **Username:** `meliodas`
- **Password found in the lab:** `iloveyou1`
- **User flag:** **`6d488cbb3f111d135722c33cb635f4ec`**

## 3. Privilege Escalation (Root Flag)

After gaining initial access, the first step was to check for misconfigured sudo privileges:

```bash
sudo -l
```

Output:

```text
(ALL) NOPASSWD: /usr/bin/python* /home/meliodas/bak.py
```

This means the user `meliodas` can run the script `/home/meliodas/bak.py` as root without a password.

Reading the script revealed it imports the `zipfile` module to back up web files:

```python
import os
import zipfile
```

### The Vulnerability: Python Library Hijacking

When Python executes a script, it searches for imported modules in a specific order. A module in a user-writable location on Python's import path can be loaded in place of the intended system module. Since `meliodas` owns the script's directory, creating a file named `zipfile.py` there can cause the elevated script to load that file.

The original write-up used this issue to obtain a root shell. The executable payload and invocation that trigger the privilege escalation are omitted. In a real environment, do not run privileged scripts from directories writable by untrusted users, and ensure Python imports resolve only to trusted modules.

After obtaining root access in the lab, the final flag was read from `/root/root.txt`.

- **Root flag:** **`e8c8c6c256c35515d1d344ee0488c617`**

## 💡 Key Learnings

- **Information Disclosure:** Always check `robots.txt` and web page content for hidden hints, usernames, and password patterns.
- **Credential Security:** Weak passwords can put SSH accounts at risk; use strong, unique passwords and appropriate authentication controls.
- **Python Library Hijacking:** When a script runs with elevated privileges, a user-writable import location can allow an untrusted module to be loaded. Keep privileged scripts and their import paths under trusted ownership.
