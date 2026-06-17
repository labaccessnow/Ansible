# Ansible

Idempotent provisioning and configuration management for network and compute — environments
rebuilt from a repository, not hand-assembled. The thing I optimize for is being able to run it
again, safely, and get the same result.

## What it looks like in practice

One reusable role provisions different machine types from per-host variables, and the run is
idempotent — `--check` shows you the delta before anything happens:

```yaml
# host_vars/dc-demo.yml — the ONLY thing that differs between builds
vm_spec:
  vmid: 126
  cores: 2
  memory: 4096
  disk_bus: sata        # native driver, no virtio injection on the Windows install
  install_iso: "win2025.iso"
  build_answer_iso: true

# roles/proxmox_vm/tasks/main.yml — same task file builds any of them
- name: "Create VM {{ vm_spec.vmid }} ({{ vm_spec.name }})"
  community.general.proxmox_kvm:
    vmid: "{{ vm_spec.vmid }}"
    cores: "{{ vm_spec.cores }}"
    memory: "{{ vm_spec.memory }}"
    # ... API creds from SOPS, never in the repo
    state: present
```

The newer move is **Event-Driven Ansible** — instead of running playbooks on a schedule, a
rulebook reacts to events and remediates:

```yaml
- name: Alert-driven remediation
  hosts: all
  sources:
    - ansible.eda.alertmanager: { host: 0.0.0.0, port: 5000 }
  rules:
    - name: Quarantine on a critical port flap
      condition: event.payload.labels.severity == "critical"
      action:
        run_playbook: { name: playbooks/quarantine_port.yml }
```

## Best practices I follow
- **`--check` before apply, every time.** If a dry run can't tell me what'll change, the play
  isn't written well enough yet.
- **Secrets out of the repo.** SOPS-encrypted vars; nothing mutates unless explicitly applied.
- **Execution environments.** Containerized, version-pinned dependency bundles so my laptop,
  CI, and the controller run the *exact* same automation.
- **Roles + per-host vars over copy-paste playbooks.** One code path, many targets.

## Lessons learned
- **Pin your collections — version drift will break a working play silently.** `numa: true` had
  worked in `proxmox_kvm` for years; `community.general` 12.x changed `numa` to expect a *dict*,
  and every VM create started failing with a type error. Nothing in *my* code changed. Pin
  collections in `requirements.yml` and read the changelogs on upgrade.
- **Order matters more than the modules.** Standing up AD + ISE, ISE needs working DNS/NTP at
  setup — so the DC has to be fully promoted before ISE boots. The play encodes that ordering;
  ignoring it just moves the failure later.
- **Idempotency is a feature you test, not assume.** Run it twice in CI and diff.

## Where the platform is
- **EDA dev preview 2022 → GA in AAP 2.4 (2023).**
- **AAP 2.5 (Sep 30, 2024):** one control plane for **Automation Execution** (playbooks) and
  **Automation Decisions** (Event-Driven Ansible); execution environments standardized.
- **AAP 2.6 (Fall 2025):** continued platform consolidation.

## Reference
**[ise-demo-enclave](https://github.com/labaccessnow/ise-demo-enclave)** — one role builds both a
Windows Domain Controller and a Cisco ISE node, fully unattended, end to end.
