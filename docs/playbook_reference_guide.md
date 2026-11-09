# Playbook Reference Guide

Complete reference for all available playbooks in the RHEL CIS Benchmark role.

## Table of Contents

- [Production Playbooks](#production-playbooks)
- [Diagnostic Playbooks](#diagnostic-playbooks)
- [Utility Playbooks](#utility-playbooks)
- [Test Playbooks](#test-playbooks)

---

## Production Playbooks

### rhel_cis_with_rollback.yml / site.yml

**Purpose:** Main CIS Benchmark hardening workflow with full or selective audit and rollback protection.

**Use Case:** Production deployments via AAP or CLI.

**Features:**
- AAP survey-compatible
- Selective or full CIS profile audit
- Native XML score parsing
- Automatic backup creation
- Post-remediation operability verification
- Automated rollback on connection/service failure

**Usage:**

```bash
# Full CIS audit & remediation (CLI default)
ansible-playbook -i inventory playbooks/site.yml

# Selective mode: remediate specific rules only
ansible-playbook -i inventory playbooks/site.yml \
  -e rhel_cis_rules_to_apply='["sysctl_crypto_fips_enabled","audit_rules_login_events"]'

# Target CIS Server Level 2 with tailoring
ansible-playbook -i inventory playbooks/site.yml \
  -e rhel_cis_benchmark_profile=cis_server_l2 \
  -e rhel_cis_tailoring_profile=myorg
```

**Variables:**

| Variable | Default | Description |
|----------|---------|-------------|
| `rhel_cis_benchmark_profile` | `cis_server_l1` | CIS profile (`cis_server_l1`, `cis_server_l2`, `cis_workstation_l1`, `cis_workstation_l2`) |
| `rhel_cis_rules_to_apply` | `[]` | List of rule IDs (empty = full profile) |
| `rhel_cis_tailoring_profile` | `""` | Tailoring directory name under `tailoring/` |
| `rhel_cis_use_native_parsing` | `true` | Use native Ansible XML parsing |
| `rhel_cis_apply_remediations` | `true` | Apply generated fixes |
| `rhel_cis_auto_rollback_on_failure` | `true` | Trigger rollback if checks fail |
| `rhel_cis_enable_auto_backup` | `true` | Create pre-hardening snapshot |

---

### rhel_cis_with_rollback_interactive.yml

**Purpose:** Interactive CLI playbook with terminal prompts.

**Use Case:** Manual deployments requiring guided interactive configuration.

**Prompts:**
1. CIS Profile selection (`cis_server_l1`, `cis_server_l2`, etc.)
2. Audit mode (full or selective)
3. CIS rules list (if selective)
4. Remediation confirmation
5. Auto-rollback enablement

---

### rhel_cis_audit_only.yml

**Purpose:** Run OpenSCAP audit against the selected CIS profile without modifying target systems.

**Use Case:** Baseline compliance discovery and periodic audit checks.

```bash
ansible-playbook -i inventory playbooks/rhel_cis_audit_only.yml \
  -e rhel_cis_benchmark_profile=cis_server_l1
```

**Output:**
- XML results: `/opt/cis_audit/baseline_*.xml`
- HTML report: `/opt/cis_audit/baseline_*.html`
- Compliance score printed directly to stdout

---

### final_audit_report.yml

**Purpose:** Generate a detailed compliance report categorizing results and evaluating system readiness.

**Use Case:** Post-hardening compliance auditing and security verification.

```bash
ansible-playbook -i inventory playbooks/final_audit_report.yml
```

---

### perform_rollback.yml

**Purpose:** Restore system configuration and service states from a pre-hardening backup archive.

**Use Case:** Recovering from an interrupted hardening run or reverting policy changes.

```bash
ansible-playbook -i inventory playbooks/perform_rollback.yml \
  -e rhel_cis_backup_name=pre_cis_hostname_20261102T120000
```

---

## Diagnostic Playbooks

### diagnose_cis_remediation.yml
Checks OpenSCAP packages, verifies datastream XML availability, reviews remediation logs in `/opt/cis_work/`, and identifies why individual controls failed.

### full_diagnostic.yml
Validates host OS compatibility (RHEL 7–10), verifies disk space, tests local repos for `python3-lxml`/`python-lxml`, and checks system service statuses.

### verify_system_operational.yml
Standalone health check assessing SSH responsiveness, PAM module integrity, root filesystem writability, and status of `sshd` and `NetworkManager`.

---

## Utility Playbooks

| Playbook | Purpose | Changes System? |
|----------|---------|-----------------|
| `rhel_cis_backup_state.yml` | Generates a standalone configuration backup | No |
| `emergency_restore_from_controller.yml` | Restores snapshot stored on AAP controller | Yes |
| `rhel_cis_query.yml` | Queries current CIS compliance score and backup status | No |
| `test_tailoring.yml` | Validates tailoring YAML syntax without running scans | No |