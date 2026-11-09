# CIS Benchmark Playbook Survey Configuration Guide

This guide explains how to configure and use the interactive survey feature for selective CIS Benchmark auditing and remediation in AAP / AWX.

## Table of Contents
- [Overview](#overview)
- [Ansible Automation Platform Configuration](#ansible-automation-platform-configuration)
- [CLI Interactive Mode](#cli-interactive-mode)
- [Survey Variables](#survey-variables)
- [Common CIS Rule IDs](#common-cis-rule-ids)

---

## Overview

The CIS hardening workflow supports **selective auditing and remediation**:

1. Choose between **full** or **selective** CIS audit mode.
2. Select target CIS Profile (`cis_server_l1`, `cis_server_l2`, `cis_workstation_l1`, `cis_workstation_l2`).
3. Specify specific CIS rules to audit when in selective mode.
4. Remediate ONLY the failed subset of selected rules.
5. Control automated rollback behavior if system checks fail.

---

## Ansible Automation Platform Configuration

In AAP 2.7, add these questions under the **Survey** tab on your Job Template:

### Question 1: CIS Audit Mode
- **Variable**: `cis_audit_mode`
- **Question**: `CIS Audit Mode`
- **Description**: `Select full audit (all rules in profile) or selective (specific rules only)`
- **Type**: `Multiple Choice (single select)`
- **Choices**: `full`, `selective`
- **Default**: `full` (Required: Yes)

### Question 2: CIS Profile
- **Variable**: `rhel_cis_benchmark_profile`
- **Question**: `CIS Benchmark Profile`
- **Description**: `Select target CIS Benchmark profile level`
- **Type**: `Multiple Choice (single select)`
- **Choices**:
  ```
  cis_server_l1
  cis_server_l2
  cis_workstation_l1
  cis_workstation_l2
  ```
- **Default**: `cis_server_l1` (Required: Yes)

### Question 3: CIS Rules to Audit
- **Variable**: `cis_rules_input`
- **Question**: `CIS Rules to Audit (comma-separated, selective mode only)`
- **Description**: `Comma-separated list of rule IDs (e.g. sysctl_crypto_fips_enabled,audit_rules_login_events). Leave blank for full mode.`
- **Type**: `Textarea`
- **Default**: *(blank)* (Required: No)

### Question 4: Apply Remediations
- **Variable**: `rhel_cis_apply_remediations`
- **Question**: `Apply Remediations?`
- **Type**: `Multiple Choice (single select)`
- **Choices**: `true`, `false`
- **Default**: `true` (Required: Yes)

### Question 5: Auto Rollback on Failure
- **Variable**: `rhel_cis_auto_rollback_on_failure`
- **Question**: `Auto Rollback on Failure?`
- **Type**: `Multiple Choice (single select)`
- **Choices**: `true`, `false`
- **Default**: `true` (Required: Yes)

### Question 6: Organization Tailoring Profile
- **Variable**: `rhel_cis_tailoring_profile`
- **Question**: `Organization Tailoring Profile (optional)`
- **Description**: `Directory name under tailoring/ (leave blank for defaults)`
- **Type**: `Text`
- **Default**: *(blank)* (Required: No)

---

## CLI Interactive Mode

When invoking outside AAP, run:

```bash
ansible-playbook -i inventory playbooks/rhel_cis_with_rollback_interactive.yml
```

The interactive playbook will prompt for the profile level, audit mode, target rule list, and remediation confirmation.

---

## Common CIS Rule IDs

### Filesystem & Partitions
- `partition_for_tmp` - Ensure /tmp is a separate partition
- `partition_for_var` - Ensure /var is a separate partition
- `partition_for_var_log` - Ensure /var/log is a separate partition
- `partition_for_var_log_audit` - Ensure /var/log/audit is a separate partition

### System Access & Authentication
- `accounts_password_pam_minlen` - Minimum password length
- `accounts_passwords_pam_faillock_deny` - Account lockout threshold
- `accounts_password_pam_unix_remember` - Prevent password reuse
- `sshd_disable_root_login` - Disable direct SSH root login
- `sshd_set_idle_timeout` - SSH client idle timeout

### Logging & Auditing
- `service_auditd_enabled` - Enable auditd service
- `audit_rules_login_events` - Record login/logout events
- `audit_rules_usergroup_modification` - Audit user and group modifications
- `audit_rules_privileged_commands` - Audit privileged commands