# Baseline Security Assessment

## Purpose

A baseline security assessment was performed on the Ubuntu Server before security hardening was applied.

The purpose of this assessment is to document the initial state of the server so that security improvements can be measured after hardening.

## Lab System

| Category | Finding |
|---|---|
| Operating System | Ubuntu 26.04.1 LTS |
| Kernel | 7.0.0-31-generic |
| Architecture | x86_64 |
| Hostname | linuxhardening |
| User | test |
| Network Interface | enp0s3 |
| IP Address | 10.0.2.15/24 |
| Firewall | UFW inactive |

## Network Configuration

The primary network interface was identified as:

```text
enp0s3
