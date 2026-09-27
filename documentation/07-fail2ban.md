# Fail2Ban Intrusion Prevention

## Objective

Fail2Ban was implemented as an additional defensive security control for the SSH service.

Fail2Ban monitors system logs for repeated authentication failures and can automatically ban source IP addresses that exceed configured thresholds.

This provides an additional layer of protection against repeated authentication attempts and brute-force activity.

## Initial Assessment

Fail2Ban was not installed on the initial Ubuntu Server deployment.

Installation status was checked using:

```bash
dpkg -l | grep fail2ban
```

No installed Fail2Ban package was initially returned.

## Installation

Fail2Ban was installed using:

```bash
sudo apt install fail2ban
```

After installation, the service status was verified using:

```bash
sudo systemctl status fail2ban
```

The service reported:

```text
Active: active (running)
```

Fail2Ban was also enabled to start automatically with the system.

## Active Jail Review

The active Fail2Ban configuration was inspected using:

```bash
sudo fail2ban-client status
```

The server reported one active jail:

```text
Number of jail: 1
Jail list: sshd
```

The `sshd` jail monitors authentication activity associated with the OpenSSH service.

## SSH Jail Status

The SSH jail was inspected using:

```bash
sudo fail2ban-client status sshd
```

At the time of the assessment:

```text
Currently failed: 0
Total failed: 0
Currently banned: 0
Total banned: 0
```

No IP addresses were banned.

## Ban Policy

The SSH jail's active thresholds were reviewed using:

```bash
sudo fail2ban-client get sshd maxretry
sudo fail2ban-client get sshd findtime
sudo fail2ban-client get sshd bantime
```

The configuration returned:

```text
maxretry = 5
findtime = 600 seconds
bantime = 600 seconds
```

This configuration means that an IP address generating five qualifying SSH authentication failures within a ten-minute period can be banned for ten minutes.

## Log Monitoring

The journal filter used by the SSH jail was inspected using:

```bash
sudo fail2ban-client get sshd journalmatch
```

The SSH jail was configured to monitor SSH activity through the systemd journal.

The returned journal match included:

```text
_SYSTEMD_UNIT=ssh.service + _COMM=sshd
```

This creates the following detection workflow:

```text
SSH authentication activity
        ↓
systemd journal
        ↓
Fail2Ban sshd filter
        ↓
Failure threshold detection
        ↓
Temporary source IP ban
```

## Filter Validation

The SSH Fail2Ban filter was tested against available systemd journal data using:

```bash
sudo fail2ban-regex systemd-journal sshd
```

At the time of testing, no qualifying failed SSH authentication events were available for the filter to match.

The test reported:

```text
Failregex: 0 total
Lines: 1 lines, 0 ignored, 0 matched, 1 missed
```

This result did not indicate a Fail2Ban service failure. The active SSH jail remained operational and no failed authentication activity had triggered the configured threshold.

## Defense in Depth

SSH had previously been hardened to use public-key authentication with password authentication disabled.

Fail2Ban provides an additional defensive layer by monitoring SSH authentication activity and responding to repeated qualifying failures.

The combined controls include:

- SSH key-based authentication
- Password authentication disabled
- UFW host-based firewall
- Fail2Ban SSH monitoring
- Automatic temporary banning based on repeated authentication failures

## Future Attack Simulation

An intentional ban was not triggered from the Windows management connection because the connection reaches the Ubuntu guest through VirtualBox NAT.

A future Kali Linux testing system will be used to safely generate controlled failed SSH authentication attempts from a separate lab system.

The future test will demonstrate:

1. Failed SSH authentication attempts generated from Kali Linux
2. Authentication events recorded by Ubuntu
3. Fail2Ban detecting the configured failure threshold
4. The Kali source IP being temporarily banned
5. Verification of the ban using Fail2Ban status and system logs
6. Restoration of access after the ban expires or is administratively removed

This will provide a controlled attacker-versus-defender validation of the Fail2Ban configuration.

## Security Impact

Fail2Ban adds automated detection and response capability to the server.

Rather than relying exclusively on preventive controls, the server can monitor authentication activity and automatically respond when configured failure thresholds are exceeded.

## Status

**Fail2Ban installation:** Complete

**Fail2Ban service:** Active

**SSH jail:** Active

**Failure threshold:** 5 attempts within 10 minutes

**Ban duration:** 10 minutes

**Journal monitoring:** Verified

**Controlled attack simulation:** Planned
