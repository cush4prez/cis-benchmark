# CIS Tailoring Quick Start

Get started with CIS Benchmark tailoring in 5 minutes.

## 1. Create Your Profile

```bash
# Create organization directory
mkdir -p tailoring/myorg

# Copy template
cp tailoring/example_tailoring.yml tailoring/myorg/common.yml
```

## 2. Edit Your Variables

Edit `tailoring/myorg/common.yml`:

```yaml
---
# Your Organization Name
# CIS Benchmark Baseline Overrides

# Essential variables (most commonly customized)
var_password_pam_minlen: 14                         # CIS recommendation is >=14
var_accounts_tmout: 900                             # Shell session timeout (seconds)
var_accounts_passwords_pam_faillock_deny: 5         # Failed login attempts before lockout
sshd_idle_timeout_value: 300                        # SSH idle timeout (seconds)
var_auditd_max_log_file: 32                         # Audit log size (MB)
var_system_crypto_policy: "DEFAULT"                 # Crypto policy (e.g., DEFAULT or FIPS)
```

## 3. Use Your Profile

### Option A: Command Line

```bash
ansible-playbook -i inventory playbooks/site.yml \
  -e rhel_cis_benchmark_profile=cis_server_l1 \
  -e rhel_cis_tailoring_profile=myorg
```

### Option B: Group Variables

```yaml
# group_vars/all.yml
rhel_cis_benchmark_profile: "cis_server_l1"
rhel_cis_tailoring_profile: "myorg"
```

### Option C: AAP Survey

Select your CIS Benchmark profile (e.g., `cis_server_l1`) and enter `myorg` in the **Tailoring Profile** survey prompt when launching the job.

## 4. Test First

```bash
# Test variable loading syntax without running audits
ansible-playbook playbooks/test_tailoring.yml \
  -e rhel_cis_tailoring_profile=myorg

# Audit only (evaluate compliance without making changes)
ansible-playbook -i inventory playbooks/rhel_cis_audit_only.yml \
  -e rhel_cis_benchmark_profile=cis_server_l1 \
  -e rhel_cis_tailoring_profile=myorg
```

## Common CIS Variables

| Variable | CIS Default | Description |
|----------|-------------|-------------|
| `var_password_pam_minlen` | 14 | Minimum password length |
| `var_accounts_tmout` | 900 | Shell session idle timeout (seconds) |
| `var_accounts_passwords_pam_faillock_deny` | 5 | Lockout threshold for failed authentications |
| `var_accounts_passwords_pam_faillock_unlock_time` | 900 | Lockout duration (seconds) |
| `sshd_idle_timeout_value` | 300 | SSH `ClientAliveInterval` idle timeout (seconds) |
| `var_auditd_max_log_file` | 8 | Maximum audit log file size (MB) |
| `var_auditd_num_logs` | 5 | Rotated audit log retention count |
| `var_system_crypto_policy` | DEFAULT | System-wide cryptographic subpolicy |

## Version-Specific Overrides (Optional)

Create `tailoring/myorg/rhel9.yml` or `tailoring/myorg/rhel10.yml` for release-specific tweaks:

```yaml
---
# RHEL 9/10 specific tuning
var_authselect_profile: "sssd"
var_password_pam_minlen: 16
```

## Need Help?

- **Full Tailoring Guide**: See [docs/TAILORING_GUIDE.md](../docs/TAILORING_GUIDE.md)
- **Role Details**: See [docs/ROLE_DOCUMENTATION.md](../docs/ROLE_DOCUMENTATION.md)
- **Playbook Guide**: See [docs/PLAYBOOK_REFERENCE.md](../docs/PLAYBOOK_REFERENCE.md)
- **Test Loading**: Run `ansible-playbook playbooks/test_tailoring.yml`

## Validation Checklist

- [ ] Created `tailoring/myorg/` directory
- [ ] Copied and populated `common.yml`
- [ ] Documented justification for any organizational deviations
- [ ] Validated syntax via `test_tailoring.yml`
- [ ] Ran audit-only baseline scan in non-production
- [ ] Committed tailored profiles to Git version control

## Quick Reference

- **Load Profile**: Set `rhel_cis_tailoring_profile`
- **Select CIS Level**: Set `rhel_cis_benchmark_profile` (`cis_server_l1`, `cis_server_l2`, `cis_workstation_l1`, `cis_workstation_l2`)
- **Dry Run Audit**: Use `playbooks/rhel_cis_audit_only.yml`