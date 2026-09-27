# linux-server-hardening-lab
A hands-on cybersecurity home lab focused on Linux server hardening, system security, monitoring, and vulnerability assessment.

## Security Evidence

### SSH Hardening

The final OpenSSH configuration was reviewed to verify that the intended controls were actually active.

![SSH Hardening](<img width="1055" height="210" alt="02-ssh-hardening" src="https://github.com/user-attachments/assets/74e56101-1d57-4b0c-ae55-3dae713c3816" />
)

Key controls include:

- Public-key authentication enabled
- Password authentication disabled
- Direct root SSH login disabled
- Maximum authentication attempts reduced to 3
- X11 forwarding disabled
- TCP forwarding disabled
- SSH agent forwarding disabled
- Verbose SSH logging enabled

---

### UFW Firewall

UFW was configured with a default-deny policy for incoming connections while allowing required SSH access.

![UFW Firewall](<img width="562" height="198" alt="04-ufw-firewall" src="https://github.com/user-attachments/assets/e5a437f6-1c5e-4530-b061-68634d9cf98b" />
)

Final firewall policy:

- Incoming traffic: deny by default
- Outgoing traffic: allow by default
- SSH: TCP port 22 explicitly allowed
- Firewall logging enabled

---

### Fail2Ban SSH Protection

Fail2Ban was installed and verified with an active `sshd` jail.

![Fail2Ban](<img width="573" height="269" alt="fail2ban" src="https://github.com/user-attachments/assets/16a09a1c-3448-411d-ad0a-a3e16efa7361" />
)

The SSH jail monitors authentication activity through the systemd journal.

A deliberate ban was not generated from the management connection because VirtualBox NAT causes the Windows host to share the lab's management path. Controlled attack testing is planned from a separate Kali Linux VM.

---

### SSH Authentication Auditing

SSH journal records were reviewed to verify authentication activity and configuration changes.

![SSH Authentication Logs](<img width="1056" height="266" alt="06-ssh-audit-log png" src="https://github.com/user-attachments/assets/a950a509-869e-4f5a-a4ff-12c0a2da6453" />
)

The logs demonstrate the progression from earlier password authentication to successful Ed25519 public-key authentication after SSH hardening.

They also provide evidence of SSH service reloads after configuration changes.

---

### Lynis Security Assessment

Lynis was used to perform an independent host security assessment after the initial hardening work.

![Lynis Final Assessment](<img width="413" height="96" alt="08-lynis-final" src="https://github.com/user-attachments/assets/911d8a65-04b7-43cb-a6d0-075408c752e8" />
)

The initial Lynis hardening index was:

**65**

After reviewing the findings and implementing selected SSH security improvements, the system was rescanned.

Final hardening index:

**69**

A total of **258 security tests** were performed.

The hardening index was used as an assessment indicator rather than treated as a percentage of overall system security.
