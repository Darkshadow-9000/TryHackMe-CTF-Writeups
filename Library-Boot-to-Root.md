# TryHackMe: Library — Boot2Root Write-up

> **Machine:** Library  
> **Difficulty:** Easy  
> **Objective:** Capture `user.txt` and `root.txt` in the authorized TryHackMe lab.

## 1. Reconnaissance and Enumeration

An Nmap scan was run against the lab target to identify exposed services. The scan found **SSH on port 22** and **HTTP on port 80**; the web service identified itself as Apache on Ubuntu.

![Nmap scan results](images/library-nmap-scan.png)

Checked the web server on port 80 and ran directory fuzzing using ffuf. The fuzzing revealed a `robots.txt` file. Inside `robots.txt`, the entry `rockyou` was found, which is a strong hint to use the `rockyou.txt` wordlist for password cracking later.

![robots.txt discovery](images/library-robots.txt.png)

Exploring the main website revealed a blog post authored by **`meliodas`**, providing a potential username for the SSH service.

## 2. Initial Access — User Flag

In the lab, the username and the `rockyou.txt` hint were used to identify valid SSH credentials. After logging in as `meliodas`, the user flag was found in the account's home directory at **`user.txt`**.

- **Username:** `meliodas`
- **User flag:** Not included in the supplied source; add the value here if you want it displayed.

## 3. Privilege Escalation — Root Flag

The user's sudo permissions showed that a Python backup script could be run with elevated privileges. Reviewing the script revealed that it imports Python modules, and the script's location was writable by the user. This created a **Python module-hijacking** risk: Python may load a local module before the intended system module.

In the lab, this unsafe configuration was used to obtain elevated access. The root flag was located in `/root/root.txt`.

- **Root flag:** Not included in the supplied source; add the value here if you want it displayed.

## Key Takeaways

- **Information disclosure:** Check `robots.txt` and public website content for clues that may reveal usernames or other hints.
- **Credential security:** Avoid weak or reused passwords, and protect SSH access with strong authentication and appropriate controls.
- **Python module security:** Do not run scripts with elevated privileges from user-writable locations or allow user-controlled files to shadow trusted modules.
- **Flag values:** The original text did not contain the actual `user.txt` or `root.txt` values, so they have not been fabricated here.
