# Deep-Dive: SSH Service & Cryptographic Infrastructure Security

## 1. Architectural Placement & Cryptographic Handshake

The Secure Shell (SSH) protocol operates over **Port 22** on standard TCP connections. It provides an encrypted, authenticated transport tunnel across untrusted perimeters, replacing insecure legacy cleartext protocols like Telnet.

### Cryptographic Handshake Pipeline

When establishing a session with the SSH daemon (`sshd`), the protocol cascades across three distinct cryptographic layers:

```text
[Client]                                                         [Server (sshd)]
   │                                                                    │
   ├────── 1. Asymmetric Key Exchange (Diffie-Hellman / RSA) ───────────┤
   │       * Identity Validation & Verification                         │
   │       * Generation of Session Key                                  │
   │                                                                    │
   ├────── 2. Symmetric Encryption Tunnel (AES-GCM / ChaCha20) ─────────┤
   │       * Bulk Data Transmission Enclosed                            │
   │                                                                    │
   └────── 3. Cryptographic Data Integrity (HMAC SHA-256) ──────────────┘
           * Tamper-Proof Packet Validation
```

1. **Asymmetric Cryptography (Identity Verification):** Uses public/private key pairs to verify identity during initialization. This computationally heavy phase constructs a secure shared secret without transmitting the key over the wire.
2. **Symmetric Cryptography (Session Tunnel):** Once identity validation completes, the pipeline switches to fast symmetric encryption algorithm variations (e.g., AES) using a single-use session key to encrypt bulk command traffic.
3. **Hashing (Integrity Seals):** Uses keyed-hash message authentication codes (HMAC) to ensure that packets are not modified or injected by third parties during transmission.

---

## 2. Hardening the Daemon Config (`/etc/ssh/sshd_config`)

Securing the SSH daemon requires modifying parameters within its primary configuration block.

```ini
# /etc/ssh/sshd_config Hardening Directive Set

# 1. Defeat automated scanning noise by altering default port binding
Port 2222

# 2. Prevent direct high-privilege access vectors
PermitRootLogin no

# 3. Enforce cryptographic authentication; completely disable password entry
PasswordAuthentication no

# 4. Limit connection scope to authorized explicit usernames
AllowUsers cyberdm22
```

---

## 3. Offensive Attack Vectors & Lateral Movement

### Vector A: Brute-Force & Credential Stuffing
* **Mechanics:** If password authentication remains exposed, attackers use multi-threaded dictionary processors to map corporate usernames against broad password sets.
* **Mitigation:** Implementation of `fail2ban` defensive filters which parse log histories (`/var/log/auth.log`) and instantly generate temporary system firewall blocks on offending IPs.

### Vector B: Private Key Exfiltration (Lateral Movement)
* **Mechanics:** During post-exploitation phases inside a compromised network, attackers prioritize searching user home folders for hidden configuration environments: `ls -la ~/.ssh/`.
* **Execution:** If a non-passphrase protected private key (`id_rsa` or `id_ed25519`) is discovered, the attacker copies the file. Using `ssh -i id_rsa user@internal-target`, they can pivot laterally deeper into restricted operational environments without triggering classic multi-factor controls.

### Vector C: SSH Port Forwarding (Pivoting Tunnels)
* **Mechanics:** Attackers convert compromised dual-homed edge servers into operational proxy hops to slip traffic past restrictive stateful internal firewalls.
* **Execution (Local Port Forwarding):** ```bash
  ssh -L 8080:internal-database:3306 attacker-user@compromised-web-server
  ```
  This command securely tunnels Port 8080 on the local Kali environment directly through the encrypted SSH session to communicate with a restricted database node sitting completely deep inside an isolated inner ring network.

---

## 4. Tool Matrix: SSH Exploitation & Defense

| Tool Name | Operation Mode | Core Function | Structural Limitation |
| :--- | :--- | :--- | :--- |
| **THC Hydra** | Offensive | Executes fast, parallel network credential-guessing attacks against active Port 22 interfaces. | Instantly neutralized if key-only authentication is strictly enforced. |
| **Nmap (NSE)** | Offensive | Utilizes the Nmap Scripting Engine to audit supported algorithms, hostkeys, and daemon versions. | Passive identification profile only; does not execute direct active exploitation. |
| **Metasploit** | Offensive | Contains pre-built operational exploit code arrays targeting legacy unpatched OpenSSH vulnerabilities. | Modern production Linux platforms run highly patched instances, minimizing utility. |
| **Fail2ban** | Defensive | Dynamically processes auth logs to write real-time iptables blocks on abusive brute-force source IPs. | Operates reactively; can be bypassed via highly distributed, slow rotation botnets. |
| **PAM Multi-Factor**| Defensive | Hooks into the Pluggable Authentication Module framework to require a secondary out-of-band Token verification. | Increases administrative management and user enrollment friction. |
