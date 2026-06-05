# Core Network Services & Exploitation Matrix: HTTP & SSH

> **An advanced reference repository tracking the architecture, configuration, defensive controls, and offensive methodologies associated with core Linux networking services.** Designed as a dual-purpose reference for Information Security practitioners and Cyber Law scholars, this matrix bridges the gap between technical execution (OSI layer manipulation, cryptography, daemon configurations) and legal accountability (authorization boundaries, forensic evidence generation, and dual-use tool implications).

---

## 🏗️ Repository Architecture

This repository is modularized into dedicated service deep-dives:

* 📄 **[`HTTP_SERVICE.md`](./HTTP_SERVICE.md)**
  * **Core Concepts:** Process isolation, daemon invocation, and Web Root (`/var/www/html/`) mechanics.
  * **Architectural Analysis:** Process-Driven (Apache2) vs. Event-Driven (Nginx) scaling and security profiles.
  * **Attack Vectors:** Credential harvesting (phishing), tactical payload delivery (droppers), and out-of-band exfiltration.
  * **Tool Matrix:** Operational breakdown of Burp Suite, sqlmap, Nikto, and defensive WAFs (ModSecurity).

* 📄 **[`SSH_SERVICE.md`](./SSH_SERVICE.md)**
  * **Core Concepts:** The 3-phase cryptographic pipeline (Asymmetric identity, Symmetric tunnels, HMAC integrity).
  * **Daemon Hardening:** Deep-dive into `/etc/ssh/sshd_config` directives (disabling root login, enforcing keys).
  * **Attack Vectors:** Cryptographic key theft, brute-forcing, and advanced Local Port Forwarding (Pivoting/Tunneling) to bypass stateful firewalls.
  * **Tool Matrix:** Operational breakdown of THC Hydra, Nmap (NSE), Fail2ban, and PAM-based MFA.

---

## ⚙️ Local Deployment & Verification Setup

To follow along with the configurations and verify these services within a local, authorized Kali Linux environment, utilize the following core commands.

### 1. Network Socket Enumeration
Validating which service is actively bound to a port requires kernel-level process isolation bypass (root privileges).

```bash
# Map all listening TCP sockets and reveal daemon PIDs
sudo netstat -antp

# Specific filter for Apache2 / Nginx port bindings
sudo netstat -antp | grep :80```

###2. Service Management (Resolving Port Conflicts)
Apache2 and Nginx inherently conflict over Port 80. Manage the daemons using standard service controls:

```Bash
# Transition from Apache2 to an Nginx architecture
sudo service apache2 stop
sudo service nginx start```

###3. Deploying Custom Web Roots
To verify functional server redirection within the standard Debian web root:

```Bash
cd /var/www/html/
sudo mv index.html apache-backup.html
echo "<h1>Core Network Services Matrix | cyberdm22</h1>" | sudo tee index.html```

---

##⚖️ Legal Disclaimer & Dual-Use Technology Notice
The materials, commands, and operational workflows documented across this repository are intended strictly for educational purposes, defensive network hardening, and authorized security research.

Tools described within this matrix (such as THC Hydra, sqlmap, and Burp Suite) constitute Dual-Use Technologies. While their possession and use in authorized audits are industry standard, the unauthorized execution of these techniques against systems without explicit, written contractual authorization violates statutory laws (e.g., the Computer Fraud and Abuse Act and international equivalents). Unauthorized application transforms a security assessment into a criminal breach, generating distinct forensic evidentiary trails.
