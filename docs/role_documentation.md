# Role Documentation

Documentation for the `rhel_cis_with_rollback` Ansible role.

## Table of Contents

- [Overview](#overview)
- [Role Structure](#role-structure)
- [Phases](#phases)
- [Variables](#variables)
- [Using the Role](#using-the-role)

---

## Overview

The `rhel_cis_with_rollback` role automates Center for Internet Security (CIS) Benchmark hardening for RHEL 7, 8, 9, and 10 with:

- **Selective or Full Auditing** - Audit specific rules or complete profiles
- **Multiple CIS Profiles** - Level 1 & Level 2 Server and Workstation profiles
- **Pre-Hardening Backup** - Snapshot of critical configs, PAM, SSH, and services
- **Intelligent Rollback** - Auto-recovery on connection, PAM, or service failures
- **Native Ansible Parsing** - Fast, secure XML results processing via `community.general.xml`

---

## Role Structure

```
roles/rhel_cis_with_rollback/
├── defaults/
│   └── main.yml                  # Default variables & profile mappings
├── handlers/
│   └── main.yml                  # System service handlers
├── tasks/
│   ├── main.yml                  # Workflow orchestrator
│   ├── load_tailoring.yml        # Organizational tailoring loader
│   ├── phase1_validation.yml     # Pre-flight host & datastream validation
│   ├── phase2_backup.yml         # Backup snapshot creation
│   ├── phase3_baseline_audit.yml # Baseline OSCAP CIS audit
│   ├── phase4_remediation.yml    # Fix generation & execution
│   ├── phase5_verification.yml   # Operability tests & post-audit
│   ├── phase6_summary.yml        # Compliance report & delta summary
│   ├── parse_oscap_score.yml     # Native XML compliance calculation
│   ├── verify_operational.yml    # System health checks
│   └── rollback.yml              # Emergency restore logic
├── vars/
│   ├── RedHat-7.yml              # RHEL 7 package definitions
│   ├── RedHat-8.yml              # RHEL 8 package definitions
│   ├── RedHat-9.yml              # RHEL 9 package definitions
│   └── RedHat-10.yml             # RHEL 10 package definitions
└── README.md
```

---

## Phases

### Phase 1: Pre-Hardening Validation
- Asserts host OS is RHEL 7, 8, 9, or 10.
- Loads OS-specific variables (`RedHat-{{ major }}.yml`) using controller file verification.
- Verifies OpenSCAP datastream exists (`ssg-rhel<version>-ds.xml`).
- Installs `openscap-scanner`, `scap-security-guide`, and the proper `lxml` package if missing.

### Phase 2: Backup
- Archives `/etc/passwd`, `/etc/shadow`, `/etc/group`, `/etc/sudoers`, `/etc/pam.d/`, `/etc/ssh/sshd_config`, `/etc/sysctl.*`, `/etc/audit/`, `/etc/selinux/`, and `/etc/firewalld/`.
- Records active and enabled systemd services.
- Compresses to `/opt/cis_backups/pre_cis_<host>_<timestamp>.tar.gz`.
- Fetches archive to AAP controller if `rhel_cis_fetch_to_controller: true`.

### Phase 3: Baseline Audit
- Executes `oscap xccdf eval` against target CIS profile ID.
- Evaluates rule outcomes natively from XML results.
- In selective mode, filters failures down to user-specified rules.

### Phase 4: Remediation
- Generates remediation script via `oscap xccdf generate fix --profile <id>`.
- Executes remediation and logs full stdout/stderr to `/opt/cis_work/`.
- Reconnects and handles target reboots gracefully.

### Phase 5: Verification & Post-Audit
- Executes operability checks (SSH listening on port 22, PAM files valid, root filesystem writable, critical daemons active).
- Triggers `rollback.yml` automatically if verification fails and auto-rollback is enabled.
- Runs post-remediation CIS audit to capture updated compliance score.

### Phase 6: Summary
- Compares baseline vs. post-hardening compliance percentages.
- Outputs human-readable delta and reports paths.

---

## Role Variables

| Variable | Default | Type | Description |
|----------|---------|------|-------------|
| `rhel_cis_benchmark_profile` | `cis_server_l1` | string | Target profile key (`cis_server_l1`, `cis_server_l2`, etc.) |
| `rhel_cis_rules_to_apply` | `[]` | list | List of rule IDs to audit/remediate (empty = full profile) |
| `rhel_cis_apply_remediations` | `true` | bool | Apply fixes after baseline audit |
| `rhel_cis_enable_auto_backup` | `true` | bool | Create pre-hardening snapshot |
| `rhel_cis_auto_rollback_on_failure` | `true` | bool | Auto-restore on operability failure |
| `rhel_cis_tailoring_profile` | `""` | string | Name of directory under `tailoring/` |
| `rhel_cis_backup_root` | `/opt/cis_backups` | string | Backup archive location |
| `rhel_cis_audit_dir` | `/opt/cis_audit` | string | Audit report directory |
| `rhel_cis_work_dir` | `/opt/cis_work` | string | Remediation work directory |

---

## Using the Role

```yaml
---
- hosts: rhel_servers
  roles:
    - role: rhel_cis_with_rollback
      rhel_cis_benchmark_profile: cis_server_l1
      rhel_cis_tailoring_profile: production_servers
      rhel_cis_apply_remediations: true
```