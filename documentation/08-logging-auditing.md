# Logging and Auditing

## Objective

The server's logging and session-tracking capabilities were reviewed to determine how an administrator could investigate:

- User login history
- Active login sessions
- SSH authentication activity
- Successful authentication methods
- Privileged `sudo` activity
- Administrative commands executed on the system

Logging and auditing are important because security controls are not only intended to prevent attacks. Administrators must also be able to reconstruct activity after an event occurs.

## Login History

The `last` command was used to review historical login activity.

The Ubuntu Server installation required the `wtmpdb` package to provide the login history database.

It was installed using:

```bash
sudo apt install wtmpdb
```

After installation:

```bash
last
```

returned multiple login records for the `test` account.

Example information included:

```text
test    pts/0    10.0.2.2
test    pts/1    10.0.2.2
```

The login records showed:

- User account: `test`
- Remote source: `10.0.2.2`
- Remote terminal sessions such as `pts/0` and `pts/1`
- Login timestamps
- Session state

The database reported that available `wtmpdb` history began on:

```text
Sun Sep 27 01:55:37 2026
```

This establishes the beginning of the currently available login history and should not be interpreted as proof that no activity occurred before that time.

## Current Session Investigation

The traditional command:

```bash
who
```

returned no session entries in this environment.

Because the server uses systemd session management, current sessions were also inspected using:

```bash
loginctl list-sessions
```

This returned multiple sessions associated with the `test` account.

The current session was then examined with:

```bash
loginctl session-status
```

The active SSH session showed information including:

```text
User: test
Remote: 10.0.2.2
Service: sshd
Type: tty
State: active
```

This confirmed that the current shell was an active remote SSH session.

## SSH Connection Environment

The SSH connection environment variable was reviewed using:

```bash
echo $SSH_CONNECTION
```

The current session returned:

```text
10.0.2.2 63250 10.0.2.15 22
```

The values represent:

```text
Client IP    Client Port    Server IP    Server Port
10.0.2.2     63250          10.0.2.15   22
```

The client port is a temporary source port selected for the connection, while TCP port `22` is the SSH service listening on the Ubuntu server.

## SSH Authentication Logs

SSH service logs were reviewed using:

```bash
sudo journalctl -u ssh --since today
```

The logs showed the progression of SSH hardening performed during the lab.

Earlier sessions included password authentication:

```text
Accepted password for test from 10.0.2.2
```

After public-key authentication was configured, later sessions showed:

```text
Accepted publickey for test from 10.0.2.2
```

This provides log-based evidence that the server transitioned from password-based SSH authentication to public-key authentication.

The journal also recorded SSH configuration reload activity after the hardening changes were made.

Examples included:

```text
Reloading ssh.service
Received SIGHUP; restarting
Reloaded ssh.service
```

A connection timeout was also observed:

```text
Timeout before authentication for connection from 10.0.2.2
```

A timeout alone does not establish malicious activity. It indicates that a connection reached SSH but authentication was not completed within the allowed period.

## Privileged Command Auditing

Privileged activity was reviewed through the system journal using:

```bash
sudo journalctl _COMM=sudo --since today
```

The records contained information such as:

```text
USER=root
PWD=/home/test
COMMAND=...
```

These fields provide useful auditing information:

- `test` — user who invoked `sudo`
- `TTY` — terminal associated with the command
- `PWD` — working directory at execution time
- `USER=root` — account the command was elevated to
- `COMMAND` — privileged command that was executed

The journal also recorded the opening and closing of privileged sessions through PAM.

Example:

```text
pam_unix(sudo:session): session opened for user root
pam_unix(sudo:session): session closed for user root
```

## Filtering Administrative Commands

To reduce log noise and focus specifically on privileged commands, the sudo journal was filtered using:

```bash
sudo journalctl _COMM=sudo --since today --no-pager | grep 'COMMAND='
```

The resulting audit trail included commands associated with several stages of the hardening lab, including:

- Removing unnecessary LXD group membership
- Reviewing password aging
- Installing OpenSSH Server
- Validating and reloading SSH configuration
- Configuring UFW firewall rules
- Installing and inspecting Fail2Ban
- Installing `wtmpdb`
- Querying system journals

This demonstrates that privileged administrative activity can be reconstructed from system logs.

## Boot Boundaries

The journal displayed boot separators similar to:

```text
-- Boot <identifier> --
```

These boundaries identify where one system boot ended and another began.

This information can be useful during incident investigation because administrators can correlate security events with system restarts.

## Security Impact

Logging and auditing provide visibility into activity occurring on the server.

The reviewed logs can help an administrator determine:

- Who logged into the system
- Where a remote connection originated
- Which authentication method was used
- Whether SSH configuration was changed or reloaded
- Which user invoked privileged commands
- What administrative commands were executed
- When activity occurred
- Whether system reboots occurred between events

These capabilities support troubleshooting, incident response, forensic investigation, and accountability.

## Key Commands

```bash
last
who
loginctl list-sessions
loginctl session-status
echo $SSH_CONNECTION
sudo journalctl -u ssh --since today
sudo journalctl _COMM=sudo --since today
sudo journalctl _COMM=sudo --since today --no-pager | grep 'COMMAND='
```

## Status

**Login history review:** Complete

**Current session review:** Complete

**SSH connection verification:** Complete

**SSH authentication log review:** Complete

**Privileged command auditing:** Complete

**Boot boundary review:** Complete

**Logging and auditing assessment:** Complete
