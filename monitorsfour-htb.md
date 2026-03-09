# MonitorsFour — HackTheBox Writeup

## Overview

* Target: MonitorsFour
* Platform: Hack The Box
* Difficulty: Easy
* OS: Windows / WSL2 Environment
* Techniques: Subdomain discovery, PHP type juggling vulnerability, credential cracking, Cacti RCE exploitation, Docker Desktop privilege escalation

---

# Reconnaissance

## Nmap Scan

Initial port scan:

```bash
nmap -sV -A -T5 -p- <IP>
```

Important results:

```
80/tcp   open  http    nginx
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0
```

The HTTP service redirected to the domain:

```
http://monitorsfour.htb
```

The host was identified as a **Windows server** with a web application exposed.

---

# Web Enumeration

## Directory Discovery

Using Gobuster:

```bash
gobuster dir -u http://monitorsfour.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Discovered endpoints:

```
/contact
/login
/user
/static
/views
/forgot-password
/controllers
```

---

## Subdomain Discovery

Subdomain fuzzing with **ffuf**:

```bash
ffuf -c -u http://monitorsfour.htb -H "Host: FUZZ.monitorsfour.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -fw 3
```

Discovered subdomain:

```
cacti.monitorsfour.htb
```

---

## Environment File Discovery

The `.env` file was accessible:

```
http://monitorsfour.htb/.env
```

Contents revealed database credentials:

```
DB_HOST=mariadb
DB_PORT=3306
DB_NAME=monitorsfour_db
DB_USER=monitorsdbuser
DB_PASS=f37p2j8f4t0r
```

Attempts to authenticate using these credentials were unsuccessful.

---

# Enumeration

## Token Endpoint Analysis

The `/user` endpoint returned an error when accessed without parameters:

```
Missing token parameter
```

When invalid tokens were supplied:

```
Invalid or missing token
```

Testing different values eventually revealed that the value **0** returned user data.

This indicated a **PHP type juggling vulnerability**.

Request example:

```
http://monitorsfour.htb/user?token=0
```

Returned JSON data containing user information including password hashes and tokens.

---

# Credential Cracking

One of the recovered password hashes was cracked, revealing credentials:

```
admin : wonderful1
```

Logging in to the main web application provided access to the dashboard.

Within the dashboard a **changelog** revealed:

* SQL injection vulnerability patch
* Migration to **Docker Desktop 4.44.2**

These details suggested potential attack vectors.

---

# Further Research

Two relevant vulnerabilities were identified:

### Cacti Vulnerability

Cacti version **1.2.28** was vulnerable to:

```
CVE-2025-24367
```

Exploit:

```
https://github.com/TheCyberGeek/CVE-2025-24367-Cacti-PoC
```

### Docker Desktop Vulnerability

Docker Desktop version **4.44.2** was vulnerable to:

```
CVE-2025-9074
```

Exploit:

```
https://github.com/BridgerAlderson/CVE-2025-9074-PoC
```

---

# Initial Access

Accessing the Cacti instance:

```
http://cacti.monitorsfour.htb
```

Testing credentials from the leaked JSON revealed valid credentials:

```
marcus : wonderful
```

Using the Cacti exploit **CVE-2025-24367**, a reverse shell was obtained.

The shell was gained as the **www-data** user.

User flag was located in:

```
/home/marcus/user.txt
```

---

# Privilege Escalation

After obtaining a shell, enumeration revealed the environment was running inside **WSL2**.

The Docker Desktop vulnerability **CVE-2025-9074** was exploited.

Steps:

1. Host the exploit on the attacker machine
2. Download it using curl
3. Execute the exploit

This provided the ability to execute commands as **root**.

---

# Accessing the Windows Host

Because the environment was running inside WSL2, the Windows filesystem was mounted.

The mount point was located at:

```
/host_root/mnt/host/c
```

Navigating to the Windows administrator directory:

```
/host_root/mnt/host/c/Users/Administrator/Desktop
```

The root flag was found at:

```
root.txt
```

---

# Lessons Learned

* Always check for **environment configuration files** such as `.env`.
* Subdomain discovery can expose additional services.
* **PHP type juggling vulnerabilities** can lead to authentication bypass.
* Version research is crucial when identifying exploitation opportunities.
* Understanding containerized environments like **Docker and WSL2** can reveal additional privilege escalation paths.
