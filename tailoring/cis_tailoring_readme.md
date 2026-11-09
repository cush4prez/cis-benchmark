# CIS Benchmark Tailoring Files

This directory contains organization-specific tailoring files for customizing Center for Internet Security (CIS) Benchmark security control variables across RHEL 7, 8, 9, and 10.

## Purpose

Tailoring files allow organizations to:
- Override default CIS control variable values (e.g., password lengths, session timeouts, lockout limits)
- Document business justifications and compliance mappings
- Maintain uniform security baselines across heterogeneous RHEL environments
- Manage configuration baselines cleanly in Git

## Directory Structure

```text
tailoring/
├── README.md                    # This file
├── QUICKSTART.md                # 5-minute setup guide
├── example_tailoring.yml        # Example template
└── <org_name>/                  # Your organization's profile
    ├── common.yml               # Variables applied across all RHEL releases
    ├── rhel7.yml                # RHEL 7 specific overrides (optional)
    ├── rhel8.yml                # RHEL 8 specific overrides (optional)
    ├── rhel9.yml                # RHEL 9 specific overrides (optional)
    └── rhel10.yml               # RHEL 10 specific overrides (optional)
```

## Creating a Tailoring Profile

### Step 1: Create Organization Directory

```bash
mkdir -p tailoring/myorg
```

### Step 2: Define Common Variables

Create `tailoring/myorg/common.yml`:

```yaml
---
# Organization: My Organization
# Purpose: CIS Benchmark baseline configuration

# PASSWORD POLICIES
var_password_pam_minlen: 14                    # Minimum password length (CIS recommendation: >=14)
var_password_pam_dcredit: -1                   # At least 1 digit
var_password_pam_ucredit: -1                   # At least 1 uppercase letter
var_password_pam_lcredit: -1                   # At least 1 lowercase letter
var_password_pam_ocredit: -1                   # At least 1 special character
var_password_pam_difok: 8                      # Difference from previous password
var_password_pam_maxrepeat: 3                  # Max consecutive identical characters
var_password_pam_unix_remember: 5              # Remember last 5 passwords

# ACCOUNT LOCKOUT (pam_faillock)
var_accounts_passwords_pam_faillock_deny: 5    # Lockout after 5 failed attempts
var_accounts_passwords_pam_faillock_unlock_time: 900  # Lockout duration: 15 minutes
var_accounts_passwords_pam_faillock_fail_interval: 900 # Attempt window: 15 minutes

# SESSION MANAGEMENT
var_accounts_tmout: 900                        # Shell idle timeout: 15 minutes
sshd_idle_timeout_value: 300                   # SSH ClientAliveInterval: 5 minutes
var_sshd_set_keepalive: 0                      # ClientAliveCountMax: 0

# AUDITING & LOGGING (auditd)
var_auditd_max_log_file: 32                    # Max audit log size in MB
var_auditd_max_log_file_action: rotate         # Rotate logs when full
var_auditd_num_logs: 5                         # Keep 5 rotated logs
var_auditd_space_left: 25                      # Alert threshold at 25% free space
var_auditd_space_left_action: email            # Action when space threshold hit

# FILE PERMISSIONS & NETWORK
var_accounts_user_umask: '027'                 # Default umask for new users
firewalld_default_zone: drop                   # Default firewall drop policy
```

### Step 3: Add Version-Specific Overrides (Optional)

Create `tailoring/myorg/rhel9.yml` or `tailoring/myorg/rhel10.yml` to apply release-specific policies:

```yaml
---
# RHEL 9/10 release adjustments
var_authselect_profile: "sssd"
var_password_pam_minlen: 16
```

## Using Tailoring Files

### Method 1: Specify in Playbook Wrapper

```yaml
---
- hosts: all
  roles:
    - role: rhel_cis_with_rollback
      rhel_cis_benchmark_profile: cis_server_l1
      rhel_cis_tailoring_profile: myorg
```

### Method 2: Command Line Extra Vars

```bash
ansible-playbook -i inventory playbooks/site.yml \
  -e rhel_cis_benchmark_profile=cis_server_l1 \
  -e rhel_cis_tailoring_profile=myorg
```

### Method 3: AAP Survey Variable

In Ansible Automation Platform, set the `rhel_cis_tailoring_profile` survey question value to `myorg`.

### Method 4: Group Variables

Add to `group_vars/all.yml` or `group_vars/production.yml`:

```yaml
rhel_cis_benchmark_profile: "cis_server_l1"
rhel_cis_tailoring_profile: "myorg"
```

## Variable Loading Precedence

Variables are merged deterministically in this order (later items take precedence):

1. Role defaults (`roles/rhel_cis_with_rollback/defaults/main.yml`)
2. Tailoring common file (`tailoring/<profile>/common.yml`)
3. Tailoring version-specific file (`tailoring/<profile>/rhel{{ version }}.yml`)
4. Inventory group variables (`group_vars/`)
5. Inventory host variables (`host_vars/`)
6. Playbook variables
7. Extra variables passed via CLI / survey (`-e`)

## Available CIS Control Variables

Extract the full list of configurable SCAP variables directly from your datastream:

```bash
oscap xccdf generate fix \
  --profile cis_server_l1 \
  --fix-type ansible \
  --output /tmp/cis_fixes.yml \
  /usr/share/xml/scap/ssg/content/ssg-rhel$(rpm -E %rhel)-ds.xml

grep -E "^(var_|sshd_|firewalld_|sysctl_)" /tmp/cis_fixes.yml | sort -u
```

## Validation & Syntax Check

Always validate tailoring configurations before rolling out hardening:

```bash
# 1. Check YAML syntax and profile discovery
ansible-playbook playbooks/test_tailoring.yml \
  -e rhel_cis_tailoring_profile=myorg

# 2. Run a non-destructive audit scan
ansible-playbook -i inventory playbooks/rhel_cis_audit_only.yml \
  -e rhel_cis_benchmark_profile=cis_server_l1 \
  -e rhel_cis_tailoring_profile=myorg
```

## Troubleshooting

### Profile Not Found
Ensure your directory name directly matches the `rhel_cis_tailoring_profile` variable:
```bash
ls -ld tailoring/myorg/
```

### Settings Not Applied
Verify variable precedence using `-vv`:
```bash
ansible-playbook playbooks/site.yml \
  -e rhel_cis_tailoring_profile=myorg \
  -vv | grep "var_password_pam_minlen"
```