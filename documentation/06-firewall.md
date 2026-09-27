# Firewall Hardening

## Objective

A host-based firewall was configured using UFW (Uncomplicated Firewall) to reduce unnecessary network exposure.

The firewall was configured according to a least-privilege approach: deny unsolicited inbound traffic by default and explicitly permit only required services.

## Initial Assessment

During the baseline assessment, UFW was checked using:

```bash
sudo ufw status verbose
```

Initial status:

```text
Status: inactive
```

Existing user-defined rules were checked using:

```bash
sudo ufw show added
```

No custom firewall rules were present.

## Default Firewall Policy

The following default policies were configured:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

This configuration denies unsolicited inbound connections while allowing the server to initiate outbound connections.

## SSH Exception

Because the server is administered remotely through SSH, SSH traffic was explicitly permitted before enabling the firewall.

```bash
sudo ufw allow 22/tcp
```

The rule was verified using:

```bash
sudo ufw show added
```

Result:

```text
ufw allow 22/tcp
```

The Ubuntu server listens on TCP port 22 even though the Windows client connects to `127.0.0.1:2222`. VirtualBox forwards Windows TCP port 2222 to TCP port 22 on the Ubuntu guest.

## Firewall Activation

UFW was enabled using:

```bash
sudo ufw enable
```

The final configuration was verified using:

```bash
sudo ufw status verbose
```

Final status:

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
```

Allowed inbound traffic:

```text
22/tcp       ALLOW IN    Anywhere
22/tcp (v6)  ALLOW IN    Anywhere (v6)
```

## Post-Hardening Verification

SSH was confirmed to still be listening on TCP port 22:

```bash
sudo ss -tlnp | grep ':22'
```

A connectivity test was also performed from the Windows host:

```powershell
Test-NetConnection 127.0.0.1 -Port 2222
```

Result:

```text
TcpTestSucceeded : True
```

A new SSH connection was then established:

```powershell
ssh -p 2222 test@127.0.0.1
```

The connection successfully authenticated using the previously configured SSH key.

This verified that legitimate remote administration remained available after firewall activation.

## Before and After

### Before

```text
UFW: inactive
Custom rules: none
Default inbound filtering: not enforced by UFW
```

### After

```text
UFW: active
Default incoming: deny
Default outgoing: allow
SSH TCP/22: explicitly allowed
Firewall logging: enabled
Remote SSH access: verified
```

## Security Impact

The firewall reduces the server's network attack surface by denying unsolicited inbound connections unless they are explicitly permitted.

Only the SSH service required for remote administration is currently allowed through UFW.

## Status

**Firewall:** Active

**Default inbound policy:** Deny

**Default outbound policy:** Allow

**SSH exception:** Configured

**Post-hardening connectivity test:** Passed
