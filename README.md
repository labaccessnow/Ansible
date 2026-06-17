# Ansible

Idempotent provisioning and configuration management for network and compute —
environments that are rebuilt from a repository, not hand-assembled.

## Patterns
- **One role, many machines** — a single reusable role provisions multiple VM types
  from per-host variables (Windows DC, Linux, network appliances) on Proxmox.
- **Unattended bring-up** — automated OS install, Active Directory forest promotion,
  and serial-driven setup of appliances that have no API on day zero.
- **Safe by default** — no change is applied unless explicitly requested; secrets are
  kept out of the repo with SOPS.
- **Network device config** — `cisco.ios` / `community.general` against live gear.

## Reference
[`ise-demo-enclave`](https://github.com/labaccessnow/ise-demo-enclave) is a working
end-to-end example — one role stands up both a Windows Domain Controller and a Cisco ISE
node, fully unattended.
