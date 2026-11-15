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

- **22/tcp (SSH)**
- **80/tcp (HTTP)**

Checked the web server on port 80 and ran directory fuzzing using `ffuf`. The fuzzing revealed a `robots.txt` file. Inside `robots.txt`, the entry **"rockyou"** was found, which is a strong hint to use the `rockyou.txt` wordlist for password cracking later.

Exploring the main website revealed a blog post authored by a user named **meliodas**. This gave us a potential username for SSH.

## 2. Initial Access (User Flag)

Using the username **meliodas** and the `rockyou.txt` wordlist, I performed a brute-force attack against the SSH service using `hydra`.

```bash
hydra -l meliodas -P /usr/share/wordlists/rockyou.txt ssh://<TARGET_IP>
```

Hydra successfully found the password: **iloveyou1**.

Logged into the machine via SSH using these credentials:

```bash
ssh meliodas@<TARGET_IP>
```

Once logged in, navigated to the home directory and read the user flag in `user.txt`.

**User Flag:** **6d488cbb3f111d135722c33cb635f4ec**

## 3. Privilege Escalation (Root Flag)

After gaining initial access, the first step was to check for misconfigured sudo privileges.

```bash
sudo -l
```

Output:

```text
(ALL) NOPASSWD: /usr/bin/python* /home/meliodas/bak.py
```

This means the user `meliodas` can run the script `/home/meliodas/bak.py` as root without a password.

Reading the script revealed it imports the `zipfile` module to backup web files.

```python
import os
import zipfile
```

### The Vulnerability: Python Library Hijacking

When Python executes a script, it searches for imported modules in a specific order. The first place it looks is the current directory where the script is located (in this case, `/home/meliodas/`). Since `meliodas` owns this directory, we can create a malicious file named `zipfile.py`.

Created a fake `zipfile.py` containing a simple payload to spawn a bash shell:

```bash
echo 'import os; os.system("/bin/bash")' > /home/meliodas/zipfile.py
```

Executed the allowed sudo command. Because Python found our fake `zipfile.py` before the real system module, it executed our payload as root:

```bash
sudo /usr/bin/python3 /home/meliodas/bak.py
```

This successfully spawned a root shell. Navigated to the `/root` directory and read the final flag in `root.txt`.

**Root Flag:** **e8c8c6c256c35515d1d344ee0488c617**

## 💡 Key Learnings

- **Information Disclosure:** Always check `robots.txt` and web page content for hidden hints, usernames, and password patterns.
- **Python Library Hijacking:** When a script is executed with elevated privileges (like `sudo`), and the user has write access to the script’s directory, they can hijack imported modules. Python prioritizes the local directory over system libraries, allowing an attacker to execute arbitrary code as the privileged user.
