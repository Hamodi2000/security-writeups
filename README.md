# Security Writeups

Collection of security lab writeups documenting enumeration, exploitation, and privilege escalation techniques practiced during penetration testing labs and CTF environments.
All activities were performed in **authorized lab environments**.

---

# Methodology

Most machines follow a structured methodology:

1. **Reconnaissance**

   * Port scanning
   * Service identification

2. **Enumeration**

   * Web enumeration
   * Service enumeration
   * Credential discovery

3. **Initial Access**

   * Exploiting vulnerabilities
   * Authentication bypass
   * Remote code execution

4. **Privilege Escalation**

   * Local enumeration
   * Misconfiguration exploitation
   * Kernel / service vulnerabilities

5. **Post Exploitation**

   * Flag discovery
   * System analysis

---

# Machines

| Machine                             | Platform   | OS             | Techniques                                                |
| ----------------------------------- | ---------- | -------------- | --------------------------------------------------------- |
| [Archetype](archetype-htb.md)       | HackTheBox | Windows        | SMB enumeration, MSSQL exploitation, credential reuse     |
| [Expressway](expressway-htb.md)     | HackTheBox | Linux          | IKE enumeration, PSK cracking, SUID privilege escalation  |
| [MonitorsFour](monitorsfour-htb.md) | HackTheBox | Windows / WSL2 | PHP type juggling, Cacti RCE, Docker privilege escalation |

---

# Tools Used

Common tools used during these labs:

* Nmap
* Gobuster
* ffuf
* SMBClient
* Impacket
* ike-scan
* psk-crack
* WinPEAS / LinPEAS
* searchsploit
* Metasploit

---

# Disclaimer

These writeups are intended for **educational purposes only**.

---

# Author

Mohammed Alaa
Cybersecurity | Network Security | Penetration Testing
