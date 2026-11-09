# CIS Benchmark Tailoring Guide

This guide explains how to customize CIS Benchmark security control variables for your organization using tailoring files.

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [How Tailoring Works](#how-tailoring-works)
- [Common Variables](#common-variables)
- [Examples](#examples)
- [Troubleshooting](#troubleshooting)

---

## Overview

CIS tailoring allows you to:
- **Set Corporate Baselines**: Define custom password lengths, timeouts, and lockout settings.
- **Support Multi-OS Fleets**: Set shared settings in `common.yml` while overriding version-specific nuances in `rhel7.yml`, `rhel8.yml`, `rhel9.yml`, or `rhel10.yml`.
- **Maintain Version Control**: Manage organizational security baselines in Git.

---

## Quick Start

### 1. Create Tailoring Profile Directory

```bash
mkdir -p tailoring/myorg
cp tailoring/example_tailoring.yml tailoring/myorg/common.yml
```

### 2. Configure Your Variables

Edit `tailoring/myorg/common.yml`:

```yaml
---
# Organization: My Company
# Baseline CIS Overrides

var_password_pam_minlen: 16
var_accounts_passwords_pam_faillock_deny: 5
var_accounts_tmout: 900
sshd_idle_timeout_value: 600
```

### 3. Apply the Profile

```bash
ansible-playbook -i inventory playbooks/site.yml \
  -e rhel_cis_benchmark_profile=cis_server_l1 \
  -e rhel_cis_tailoring_profile=myorg
```

---

## How Tailoring Works

### Directory Structure

```
tailoring/
├── README.md                  # Documentation
├── example_tailoring.yml      # Template
└── myorg/                     # Organization profile name
    ├── common.yml             # Applied to all RHEL systems
    ├── rhel7.yml              # (Optional) RHEL 7 overrides
    ├── rhel8.yml              # (Optional) RHEL 8 overrides
    ├── rhel9.yml              # (Optional) RHEL 9 overrides
    └── rhel10.yml             # (Optional) RHEL 10 overrides
```

### Precedence Order

Variables are merged in the following order (later overrides earlier):
1. Role defaults (`roles/rhel_cis_with_rollback/defaults/main.yml`)
2. Tailoring common file (`tailoring/<profile>/common.yml`)
3. Tailoring OS file (`tailoring/<profile>/rhel<version>.yml`)
4. Inventory `group_vars` / `host_vars`
5. Extra variables (`-e`)

---

## Common Variables

### Password Complexity & PAM
| Variable | Default | Description |
|----------|---------|-------------|
| `var_password_pam_minlen` | 14 | Minimum password length |
| `var_password_pam_dcredit` | -1 | Required digit characters |
| `var_password_pam_ucredit` | -1 | Required uppercase characters |
| `var_password_pam_lcredit` | -1 | Required lowercase characters |
| `var_password_pam_ocredit` | -1 | Required special characters |
| `var_password_pam_unix_remember` | 5 | Password history count |

### Account Lockout
| Variable | Default | Description |
|----------|---------|-------------|
| `var_accounts_passwords_pam_faillock_deny` | 5 | Failed attempts before lockout |
| `var_accounts_passwords_pam_faillock_unlock_time` | 900 | Lockout duration (seconds) |
| `var_accounts_passwords_pam_faillock_fail_interval` | 900 | Failure accumulation window (seconds) |

### Session Management
| Variable | Default | Description |
|----------|---------|-------------|
| `var_accounts_tmout` | 900 | Shell idle timeout (seconds) |
| `sshd_idle_timeout_value` | 300 | SSH idle timeout (seconds) |

### Auditing & Logging
| Variable | Default | Description |
|----------|---------|-------------|
| `var_auditd_max_log_file` | 8 | Audit log file size (MB) |
| `var_auditd_num_logs` | 5 | Number of rotated logs to keep |
| `var_auditd_space_left` | 25 | Space-left threshold alert (%) |

---

## Examples

### Enterprise Server Baseline (`tailoring/enterprise/common.yml`)
```yaml
---
var_password_pam_minlen: 16
var_password_pam_dcredit: -1
var_password_pam_ucredit: -1
var_password_pam_lcredit: -1
var_password_pam_ocredit: -1
var_accounts_passwords_pam_faillock_deny: 3
var_accounts_passwords_pam_faillock_unlock_time: 1800
var_accounts_tmout: 600
sshd_idle_timeout_value: 300
```

### RHEL 9 Specific Adjustments (`tailoring/enterprise/rhel9.yml`)
```yaml
---
# Enforce authselect profile on modern systems
var_authselect_profile: "sssd"
```

---

## Troubleshooting

### Tailoring Profile Directory Not Found
Ensure your profile name matches the directory under `tailoring/`:
```bash
ls -ld tailoring/myorg
```
If executing via AAP, ensure `tailoring/` is committed and pushed to your source Git repository.

### Variable Values Not Taking Effect
1. Run syntax verification:
   ```bash
   ansible-playbook playbooks/test_tailoring.yml -e rhel_cis_tailoring_profile=myorg
   ```
2. Verify that host variables or CLI `-e` flags are not overriding tailoring variable names.