
# 📡 SMTP (Port 25) Testing Runbook – OSCP Style

> **Purpose**: Methodology for enumerating and testing services running on port `25/tcp` (SMTP) during a penetration test or OSCP-style assessment.

---

## 🛠️ Objective

Thoroughly enumerate and assess the attack surface of an SMTP server on TCP/25 using OSCP-style methodology.

---

## 🔍 1. Initial Service Enumeration

### 🔹 1.1 Nmap Scan

```bash
nmap -p 25 -sV --script smtp-commands,smtp-enum-users,smtp-open-relay -oN smtp_nmap.txt <TARGET_IP>
```

- `smtp-commands`: Lists supported SMTP commands  
- `smtp-enum-users`: Attempts to enumerate valid users  
- `smtp-open-relay`: Checks for open relay misconfiguration

---

### 🔹 1.2 Manual Banner Grabbing

```bash
nc <TARGET_IP> 25
```

Commands to try:

```
EHLO kali
VRFY root
VRFY admin
VRFY test
```

Responses to look for:
- `250 OK` ➝ user exists  
- `550` ➝ user does not exist

---

### 🔹 1.3 Identify Mail Server Software

```bash
nmap -p 25 -sV <TARGET_IP>
```

Alternative:

```bash
swaks --to test@target.com --server <TARGET_IP> --ehlo kali
```

---

## 🧑‍💻 2. User Enumeration

### 🔹 2.1 Using smtp-user-enum

Install:

```bash
sudo apt install smtp-user-enum
```

Usage:

```bash
smtp-user-enum -M VRFY -U users.txt -t <TARGET_IP>
smtp-user-enum -M EXPN -U users.txt -t <TARGET_IP>
```

Use targeted or common usernames in `users.txt`.

---

### 🔹 2.2 Metasploit Alternative

```bash
msfconsole
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS <TARGET_IP>
set USER_FILE /usr/share/seclists/Usernames/top-usernames-shortlist.txt
run
```

---

## 📤 3. Open Relay Testing

Test with `swaks`:

```bash
swaks --to victim@externaldomain.com --from attacker@spoofed.com --server <TARGET_IP>
```

If message is accepted with `250 OK`, the server is likely an **open mail relay**.

---

## 🧱 4. Exploiting Misconfigurations

### 🔹 4.1 Open Relay Exploitation

If the server allows relaying, you can:
- Send spoofed emails
- Simulate phishing attacks
- Enumerate infrastructure via bounce-backs
- Harvest credentials using phishing links/forms

> ⚠️ Only perform these actions **with explicit authorization**.

---

## 🕵️ 5. Check for Known Vulnerabilities

After identifying the SMTP software, search for CVEs and exploits.

### 🔹 Use SearchSploit

```bash
searchsploit exim
searchsploit postfix
searchsploit sendmail
searchsploit exchange
```

Example:

```bash
searchsploit exim 4.87
```

Also check:

- https://nvd.nist.gov
- https://cvedetails.com

---

## 📦 6. Brute-Force SMTP Authentication

> Typically more relevant on port 587 or 465, but test 25 as well.

### 🔹 Nmap Script

```bash
nmap -p 25 --script smtp-brute <TARGET_IP>
```

### 🔹 Hydra

```bash
hydra -l user -P /usr/share/wordlists/rockyou.txt -s 25 -S <TARGET_IP> smtp
```

---

## 📨 7. NTLM Authentication Checks (Exchange)

For Microsoft Exchange servers:

```bash
nmap -p 25 --script smtp-ntlm-info <TARGET_IP>
```

Possible leaks:
- Internal domain
- Hostname
- Exchange version

---

## 📂 8. OSINT-Based Email Discovery

If VRFY/EXPN are disabled, gather email addresses via OSINT.

Sources:
- Hunter.io
- LinkedIn
- GitHub
- FOCA

Then validate with:

```bash
swaks --to user@target.com --server <TARGET_IP>
```

---

## 🛑 9. No Results? Log and Move On

If:
- No open relay
- No useful SMTP commands
- No authentication
- No CVEs

✅ Log all findings  
✅ Continue to the next service

---

## 📋 10. Report-Worthy Findings

| Check                        | Description              | Risk Level |
|-----------------------------|--------------------------|------------|
| Open Mail Relay             | Mail relay allowed       | High       |
| VRFY/EXPN Enumeration       | User info disclosure     | Medium     |
| Exim/Postfix CVEs           | Remote Code Execution    | Critical   |
| NTLM Info Leak              | Domain/host info leak    | Medium     |
| SMTP Auth Brute-force       | Credential disclosure    | High       |

---

## 🧠 Tips

- Port 25 often leaks usernames even if it seems hardened.
- Also scan ports 465 and 587.
- Use banner info to fingerprint the mail server.
- Always log allowed SMTP commands.

---

## 🧰 Tools Summary

| Tool             | Purpose                          |
|------------------|----------------------------------|
| `nmap`           | Port scanning, scripts, version detection |
| `nc`             | Manual interaction with SMTP     |
| `swaks`          | Email spoofing & relay testing   |
| `smtp-user-enum` | User enumeration via VRFY/EXPN   |
| `hydra`          | Brute-force login attempts       |
| `searchsploit`   | Discover public exploits         |
| `metasploit`     | Enumeration/auxiliary modules    |

---

## 📎 References

- [RFC 5321 – SMTP](https://tools.ietf.org/html/rfc5321)
- [Swaks GitHub](https://github.com/jetmore/swaks)
- [Exploit-DB](https://www.exploit-db.com)
- [Seclists Usernames](https://github.com/danielmiessler/SecLists)

---

**Author**: *YourNameHere*  
**License**: Authorized penetration testing only  
**Last Updated**: 2025-10-22
