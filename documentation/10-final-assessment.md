# Final Security Assessment

## Objective

The final assessment summarizes the security improvements made to the Ubuntu Server during the Linux Server Hardening Lab.

The project followed a practical security workflow:

```text
Baseline Assessment
        ↓
System Updates
        ↓
User and Privilege Hardening
        ↓
SSH Hardening
        ↓
Firewall Configuration
        ↓
Fail2Ban Deployment
        ↓
Logging and Auditing
        ↓
Vulnerability Assessment
        ↓
Final Validation
```

The goal was to reduce unnecessary exposure, strengthen administrative access, improve visibility into system activity, and validate the resulting configuration.

## Environment

The lab environment consisted of:

```text
Host OS:        Windows
Hypervisor:     Oracle VirtualBox
Guest OS:       Ubuntu Server 26.04.1 LTS
Architecture:   x86_64
VM RAM:         4096 MB
VM CPUs:        2
Virtual Disk:   25 GB
Primary User:   test
Hostname:       linuxhardening
Network Mode:   VirtualBox NAT
Guest IPv4:     10.0.2.15
SSH Guest Port: 22
Host SSH Port:  2222
```

VirtualBox NAT port forwarding was used to allow the Windows host to securely administer the Ubuntu guest through SSH.

## Initial Security State

The baseline assessment identified several areas that required attention.

Initial observations included:

- UFW firewall inactive
- OpenSSH Server not initially installed
- User `test` was a member of the privileged `lxd` group
- SSH password authentication was initially available after SSH installation
- Root account password was locked
- System updates were available
- No Fail2Ban installation
- No dedicated host vulnerability assessment had been performed
- Login and administrative activity had not yet been reviewed

These observations established the baseline used to measure the hardening work.

## Security Updates

The Ubuntu package repositories were refreshed and available updates were reviewed.

The system kernel was updated from:

```text
7.0.0-31-generic
```

to:

```text
7.0.0-34-generic
```

Package updates reduced exposure to known software issues and brought the server closer to the current repository patch level.

## User and Privilege Hardening

User privileges were reviewed using Linux account and group-management tools.

The `test` account was initially a member of the `lxd` group.

Because LXD access was not required for the purpose of this server, the unnecessary group membership was removed.

Before:

```text
test ... sudo ... lxd
```

After:

```text
test ... sudo
```

The root password remained locked.

Interactive user accounts were also reviewed and no unexpected interactive accounts were identified.

Password-aging configuration was inspected but arbitrary periodic password expiration was not enabled solely for the purpose of increasing a security score.

## SSH Deployment and Hardening

OpenSSH Server was installed to provide remote administrative access.

VirtualBox NAT port forwarding was configured as:

```text
Windows 127.0.0.1:2222
        ↓
VirtualBox NAT
        ↓
Ubuntu 10.0.2.15:22
```

The SSH host key fingerprint was verified before the first connection was trusted.

An Ed25519 SSH key pair was generated on the Windows host and public-key authentication was successfully configured.

Password authentication was then disabled.

Additional SSH hardening was later implemented in:

```text
/etc/ssh/sshd_config.d/00-hardening.conf
```

Final selected SSH controls included:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
X11Forwarding no
AllowTcpForwarding no
AllowAgentForwarding no
LogLevel VERBOSE
```

SSH configuration syntax was validated using:

```bash
sudo sshd -t
```

The SSH service was reloaded only after the configuration passed validation.

A second SSH session was successfully opened after the changes, confirming that legitimate administrative access remained functional.

## Firewall Hardening

UFW was configured as the host-based firewall.

Final policy:

```text
Default incoming: deny
Default outgoing: allow
```

SSH was explicitly permitted on TCP port 22.

The resulting firewall configuration limited unsolicited inbound traffic while preserving required administrative access.

## Fail2Ban Protection

Fail2Ban was installed as an additional defensive layer for SSH.

The service was verified as active and enabled.

The active `sshd` jail used the following thresholds:

```text
Max retry: 5
Find time: 600 seconds
Ban time: 600 seconds
```

This means five qualifying authentication failures within ten minutes can result in a ten-minute temporary source-address ban.

The SSH jail was successfully confirmed to be active and monitoring systemd journal activity.

A deliberate live ban test was not performed from the Windows management connection because VirtualBox NAT causes that management traffic to appear from the same NAT-side address.

A future Kali Linux VM will be used to safely perform controlled attack simulation from a separate system.

## Logging and Auditing

Login and administrative activity were reviewed using:

```bash
last
loginctl
journalctl
```

The server was able to provide information about:

- Historical login sessions
- Active systemd login sessions
- Remote SSH source addresses
- SSH authentication methods
- SSH service activity
- Privileged `sudo` commands
- System boot boundaries

SSH logs demonstrated the transition from:

```text
Accepted password
```

to:

```text
Accepted publickey
```

after public-key authentication was implemented.

Privileged command history also demonstrated that administrative activity could be reconstructed from journal records.

## Vulnerability Assessment

Lynis was installed and used to perform a host security audit.

The initial assessment performed:

```text
258 tests
Hardening index: 65
```

Lynis detected:

```text
Firewall:           Yes
Intrusion software: Yes
Malware scanner:    No
```

Scanner findings were reviewed rather than automatically implemented.

Several appropriate SSH recommendations were selected for remediation.

After the SSH improvements, Lynis was run again.

Post-remediation result:

```text
258 tests
Hardening index: 69
```

The hardening index increased:

```text
65 → 69
```

The increase provided additional evidence that the selected hardening changes were recognized by the auditing tool.

The hardening index was not treated as a percentage of security or as a target that needed to reach 100.

## Residual Findings

Some Lynis recommendations were intentionally left unchanged after review.

Examples included:

- Changing the SSH service port
- Reducing `MaxSessions`
- Modifying `TCPKeepAlive`
- Installing a malware scanner
- Separating `/home` and `/var` onto dedicated partitions
- Restricting compiler access
- Adding legal login banners
- Additional kernel `sysctl` tuning

These findings were retained for future consideration rather than automatically applied.

Security recommendations must be evaluated against system purpose, usability requirements, operational risk, and expected threat model.

## Before and After Summary

| Security Area | Initial State | Final State |
|---|---|---|
| System patching | Updates available | System updated |
| Kernel | 7.0.0-31 | 7.0.0-34 |
| UFW | Inactive | Active |
| Incoming firewall policy | Unrestricted by UFW | Default deny |
| SSH server | Not initially installed | Installed and hardened |
| SSH passwords | Initially permitted | Disabled |
| SSH key authentication | Not configured | Ed25519 key authentication |
| Root SSH access | Restricted by default | Explicitly disabled |
| SSH authentication attempts | 6 | 3 |
| X11 forwarding | Enabled | Disabled |
| TCP forwarding | Enabled | Disabled |
| Agent forwarding | Enabled | Disabled |
| SSH logging | INFO | VERBOSE |
| LXD group access | `test` was a member | Removed |
| Fail2Ban | Not installed | Active |
| Login auditing | Not reviewed | Reviewed |
| Sudo auditing | Not reviewed | Reviewed |
| Vulnerability assessment | Not performed | Lynis assessment completed |
| Lynis hardening index | 65 | 69 |

## Defense-in-Depth Architecture

The final server uses several complementary security controls.

```text
                Internet / Network Traffic
                         ↓
                    UFW Firewall
                         ↓
                       SSH
                Key Authentication
                Passwords Disabled
                Root Login Disabled
                         ↓
                     Fail2Ban
                Authentication Monitoring
                         ↓
                 Linux User Controls
                  Least Privilege
                         ↓
               Logging and Journaling
                         ↓
                 Lynis Assessment
```

No single control is expected to provide complete security.

Instead, multiple layers reduce the likelihood that failure of one control results in complete system compromise.

## Skills Demonstrated

This project provided hands-on experience with:

- Ubuntu Server administration
- Linux command-line navigation
- Package management with APT
- Linux users and groups
- Principle of least privilege
- SSH installation and configuration
- Ed25519 public-key authentication
- SSH host-key verification
- VirtualBox NAT and port forwarding
- UFW firewall administration
- Fail2Ban
- systemd service management
- systemd journal analysis
- Login-session investigation
- Sudo auditing
- Linux environment variables
- Security configuration validation
- Lynis security auditing
- Vulnerability remediation
- Risk-based finding review
- Markdown documentation
- GitHub project documentation
- Troubleshooting and verification

## Lessons Learned

One of the major lessons from the lab was that security hardening is not simply a process of applying every recommendation produced by a scanner.

Effective system hardening requires:

1. Understanding the existing environment
2. Establishing a baseline
3. Identifying unnecessary exposure
4. Selecting appropriate controls
5. Making changes carefully
6. Validating configuration before applying changes
7. Testing legitimate access afterward
8. Reviewing logs for evidence
9. Running independent security assessments
10. Evaluating remaining findings based on risk

The lab also demonstrated the importance of understanding concepts rather than relying entirely on memorized commands.

Linux commands can be referenced when needed, but understanding what information is required and how to validate a result is more important than memorizing exact syntax.

## Future Improvements

Future expansion of this lab may include:

- Controlled Fail2Ban testing from a Kali Linux VM
- Network-based vulnerability scanning
- `auditd` deployment
- File-integrity monitoring
- Additional kernel hardening
- Malware/rootkit scanning
- Centralized log collection
- Automated hardening scripts
- Configuration backups
- Incident-response exercises
- Comparison against CIS Benchmarks

These improvements can be added without changing the core structure of the project.

## Final Result

The Ubuntu Server progressed from a basic installation into a documented and layered hardened-server environment.

The final system includes:

- Updated software
- Reduced unnecessary privilege
- Key-based SSH administration
- Disabled SSH password authentication
- Disabled root SSH login
- Restricted SSH functionality
- Host-based firewall protection
- Fail2Ban intrusion protection
- Login and privileged-command auditing
- Vulnerability assessment and remediation
- Verified post-change administrative access

The project demonstrates not only the implementation of security controls, but also the process of assessing, validating, documenting, and reviewing those controls.

## Project Status

**Linux Server Hardening Lab: COMPLETE**
