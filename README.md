# Ansible

Idempotent provisioning and configuration management for network and compute.

## Highlights
- One reusable role that provisions multiple VM types from per-host variables
- Unattended OS install + Active Directory promotion, serial-driven appliance setup
- Secrets managed with SOPS; safe-by-default (nothing changes unless explicitly applied)

## Reference
[`ise-demo-enclave`](https://github.com/labaccessnow/ise-demo-enclave) is a working example —
a single role builds both a Windows Domain Controller and a Cisco ISE node end to end.
