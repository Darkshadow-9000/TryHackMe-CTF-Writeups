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

The `rockyou.txt` clue and username were used to identify credentials in the authorized lab. The password-guessing command is omitted.

Logged into the machine via SSH using the lab credentials:

```bash
ssh meliodas@<TARGET_IP>
```

Once logged in, navigated to the home directory and read the user flag in `user.txt`.

- **User flag:** **`6d488cbb3f111d135722c33cb635f4ec`**

## 3. Privilege Escalation (Root Flag)

After gaining initial access, checked for sudo privileges:

```bash
sudo -l
```

Output:

```text
(ALL) NOPASSWD: /usr/bin/python* /home/meliodas/bak.py
```

This means the user `meliodas` can run `/home/meliodas/bak.py` as root without a password.

Reading the script revealed it imports the `zipfile` module to back up web files:

```python
import os
import zipfile
```

### The Vulnerability: Python Library Hijacking

When Python executes a script, it searches for imported modules along its import path. If an untrusted user can write a module in a location searched before the trusted system module, the script may load the untrusted file. In this case, the script's location was user-writable, creating a Python module-hijacking risk when the script was run with elevated privileges.

The commands that create and execute a privileged payload are omitted. To prevent this vulnerability, do not run privileged scripts from user-writable directories, and ensure the script and all imported modules resolve only to trusted files.

After obtaining root access in the lab, the final flag was read from `/root/root.txt`.

- **Root flag:** **`e8c8c6c256c35515d1d344ee0488c617`**

## 💡 Key Learnings

- **Information Disclosure:** Check `robots.txt` and website content for hints that may reveal usernames or other clues.
- **Credential Security:** Use strong, unique passwords and protect SSH access with appropriate authentication controls.
- **Python Library Hijacking:** Keep privileged scripts and their import paths under trusted ownership; avoid user-writable locations for code executed as root.
