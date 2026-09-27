# SSH Hardening

## Objective

Secure remote administration was configured using OpenSSH. The goal was to provide encrypted remote access while reducing the risks associated with password-based authentication.

## Initial Assessment

During the original server baseline, SSH was not installed and port 22 was not listening.

OpenSSH Server was installed using:

```bash
sudo apt install openssh-server
```

After installation, the SSH listener was verified using:

```bash
sudo ss -tlnp | grep ':22'
```

The server was listening on TCP port 22.

## VirtualBox SSH Access

The Ubuntu server uses a VirtualBox NAT network.

A VirtualBox port-forwarding rule was created:

```text
Windows 127.0.0.1:2222
        ↓
VirtualBox NAT
        ↓
Ubuntu Server TCP 22
```

Binding the forwarded port to `127.0.0.1` limits access to the Windows host rather than exposing the forwarded SSH port through other host network interfaces.

The server was accessed from Windows PowerShell using:

```powershell
ssh -p 2222 test@127.0.0.1
```

## SSH Host Verification

During the first SSH connection, the server presented its ED25519 host-key fingerprint.

The fingerprint presented to the Windows SSH client was independently compared with the fingerprint displayed directly on the Ubuntu server using:

```bash
sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

The fingerprints matched before the host key was trusted.

This provided verification that the SSH client was connecting to the intended server.

## SSH Key Authentication

An Ed25519 SSH key pair was generated on the Windows client:

```powershell
ssh-keygen -t ed25519 -C "linux-hardening-lab"
```

The private key remains on the Windows client and is protected with a passphrase.

Only the public key was transferred to the Ubuntu server and added to the user's `authorized_keys` file.

Key-based authentication was successfully tested before password authentication was disabled.

## Pre-Hardening Configuration

The effective SSH configuration was inspected using:

```bash
sudo sshd -T | grep -E 'passwordauthentication|pubkeyauthentication|permitrootlogin'
```

Initial configuration:

```text
permitrootlogin prohibit-password
pubkeyauthentication yes
passwordauthentication yes
```

Password authentication therefore remained available before hardening.

## Disable Password Authentication

The SSH configuration was modified in:

```text
/etc/ssh/sshd_config
```

The following setting was explicitly configured:

```text
PasswordAuthentication no
```

Before applying the configuration, its syntax was validated using:

```bash
sudo sshd -t
```

No errors were reported.

SSH was then reloaded:

```bash
sudo systemctl reload ssh
```

## Post-Hardening Verification

The effective SSH configuration was checked again.

Final configuration:

```text
permitrootlogin prohibit-password
pubkeyauthentication yes
passwordauthentication no
```

A new SSH session was then opened from Windows.

The server requested the SSH private-key passphrase rather than the Ubuntu account password, confirming that key-based authentication continued to function after password authentication was disabled.

## Before and After

### Before

```text
SSH installed
TCP 22 listening
Public-key authentication enabled
Password authentication enabled
```

### After

```text
SSH available through controlled VirtualBox port forwarding
Server host identity verified
Passphrase-protected Ed25519 key authentication configured
Password authentication disabled
New SSH connection successfully verified
```

## Security Impact

Disabling SSH password authentication reduces exposure to password guessing and credential-based attacks against the SSH service.

Authentication now requires possession of the authorized private SSH key. The private key is additionally protected by a passphrase.

## Status

**SSH installation:** Complete

**Host-key verification:** Complete

**SSH key authentication:** Complete

**Password authentication:** Disabled

**Post-hardening connection test:** Passed
