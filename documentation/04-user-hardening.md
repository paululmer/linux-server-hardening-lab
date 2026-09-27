# User and Account Hardening

## Objective

The server's local user accounts and privileges were reviewed to identify unnecessary access and reduce the risk associated with excessive permissions.

The primary security principle applied during this phase was the **Principle of Least Privilege**, which states that users should only have the permissions necessary to perform their required tasks.

## Administrative Account Review

The current account was inspected using:

```bash
whoami
id
```

The primary administrative account was identified as:

```text
test
```

The account has a UID and GID of `1000` and is a member of the `sudo` group, allowing administrative commands to be executed when required.

## Root Account Review

The status of the root account was checked using:

```bash
sudo passwd -S root
```

The result showed:

```text
root L
```

The `L` status indicates that the root password is locked.

No changes were made because direct password authentication to the root account was already disabled.

Administrative tasks will instead be performed through the `test` account using `sudo`.

## Interactive Account Review

Accounts configured with interactive shells were identified using:

```bash
getent passwd | grep -E '/bin/(bash|sh)$'
```

The assessment identified:

```text
root
test
```

No unexpected interactive user accounts were discovered.

## Unnecessary LXD Privileges

Initial group membership showed that the `test` account belonged to the `lxd` group:

```text
lxd:x:101:test
```

The lab does not require LXD container administration. Therefore, this group membership provided unnecessary privileges.

The account was removed from the group using:

```bash
sudo deluser test lxd
```

The change was verified using:

```bash
getent group lxd
```

After removal:

```text
lxd:x:101:
```

A new login session was then established and group membership was verified using:

```bash
id
```

The `lxd` group was no longer present.

## Password Aging Review

Password aging information was reviewed using:

```bash
sudo chage -l test
```

The account was configured with no forced password expiration.

No password-aging changes were applied during this lab. Password policy was reviewed rather than modified solely for the purpose of introducing periodic password expiration.

## Security Improvements

The following security controls were verified or implemented:

- Root password remains locked
- Administrative access is performed through `sudo`
- No unexpected interactive accounts were identified
- Unnecessary LXD group membership was removed
- Final account privileges were verified after establishing a new login session
- Password-aging configuration was reviewed

## Before and After

### Before

```text
test -> sudo + lxd privileges
```

### After

```text
test -> sudo privileges
        unnecessary lxd privileges removed
```

This change reduces unnecessary administrative capability while preserving the access required to manage the server.

## Status

**User and account hardening:** Complete

**Least privilege review:** Complete
