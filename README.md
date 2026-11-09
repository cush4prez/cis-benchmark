# RHEL CIS Benchmark Hardening with Rollback

Ansible automation for applying CIS Benchmark hardening to RHEL 7/8/9/10 systems with intelligent rollback protection and selective audit capabilities.

## Features

- ✅ **Selective or Full CIS Auditing** - Audit specific rules or all benchmark rules
- ✅ **Dynamic CIS Profiles** - Supports CIS Server Level 1 & 2 and Workstation Level 1 & 2
- ✅ **Native XML Parsing** - No Python scripts, pure Ansible using `community.general.xml`
- ✅ **Organization Tailoring** - Customize CIS benchmark control variables for your organization
- ✅ **Automatic Backup** - Pre-hardening configuration snapshots with disaster recovery
- ✅ **Intelligent Rollback** - Auto-rollback on operational failures (SSH, PAM, critical services)
- ✅ **AAP Integration** - Survey-based configuration for easy enterprise deployment
- ✅ **RHEL 7/8/9/10 Support** - Multi-version support across standard Enterprise Linux fleets

## Quick Start

### Option 1: Interactive Mode (CLI)

```bash
ansible-playbook -i inventory playbooks/rhel_cis_with_rollback_interactive.yml
```

Prompts you for:
- Target CIS Profile (`cis_server_l1`, `cis_server_l2`, `cis_workstation_l1`, etc.)
- Full or selective CIS audit mode
- Which CIS rules to audit (if selective)
- Whether to apply remediations
- Auto-rollback settings

### Option 2: Using the Role (Recommended for Production)

The cleanest approach - invokes the role directly:

```bash
ansible-playbook -i inventory playbooks/site.yml
```

Pass variables via extra vars or `group_vars`:

```bash
ansible-playbook -i inventory playbooks/site.yml \
  -e rhel_cis_benchmark_profile=cis_server_l1 \
  -e rhel_cis_tailoring_profile=myorg \
  -e rhel_cis_apply_remediations=true
```

Or create your own minimal playbook:

```yaml
---
- hosts: all
  roles:
    - role: rhel_cis_with_rollback
      rhel_cis_benchmark_profile: cis_server_l1
      rhel_cis_rules_to_apply:
        - sysctl_crypto_fips_enabled
        - audit_rules_login_events
      rhel_cis_apply_remediations: true
```

**Why site.yml?**
- ✅ Uses role (native Ansible parsing)
- ✅ Clean, maintainable code
- ✅ No duplicate logic
- ✅ Recommended for AAP deployments

### Option 3: Ansible Automation Platform (AAP)

1. **Create Project** pointing to this repository.
2. **Create Job Template** using `playbooks/site.yml` ⭐ **(recommended - uses role)**.
3. **Add Survey** with prompt variables:
   - Question 1: `cis_audit_mode` - Multiple Choice: `full` or `selective`
   - Question 2: `rhel_cis_benchmark_profile` - Dropdown: `cis_server_l1`, `cis_server_l2`, `cis_workstation_l1`, `cis_workstation_l2`
   - Question 3: `cis_rules_input` - Textarea (comma-separated rule IDs)
   - Question 4: `rhel_cis_apply_remediations` - Multiple Choice: `true` or `false`
   - Question 5: `rhel_cis_auto_rollback_on_failure` - Multiple Choice: `true` or `false`
   - Question 6: `rhel_cis_tailoring_profile` - Text (optional)
4. **Enable Survey** on the job template and launch.

## Available Playbooks

### Production Playbooks

| Playbook | Purpose | Use Case |
|----------|---------|----------|
| `rhel_cis_with_rollback.yml` | Main CIS workflow (AAP compatible) | Production hardening with survey |
| `rhel_cis_with_rollback_interactive.yml` | Interactive CLI version | CLI-based deployment with prompts |
| `perform_rollback.yml` | Restore from backup | Rollback after failed hardening |
| `rhel_cis_audit_only.yml` | Audit without remediation | Compliance reporting only |
| `final_audit_report.yml` | Detailed audit breakdown | Post-hardening validation |

### Diagnostic Playbooks

| Playbook | Purpose |
|----------|---------|
| `diagnose_cis_remediation.yml` | Diagnose remediation script execution issues |
| `full_diagnostic.yml` | Complete system diagnostic |
| `verify_system_operational.yml` | Check system operability and critical services |

### Utility Playbooks

| Playbook | Purpose |
|----------|---------|
| `rhel_cis_backup_state.yml` | Create backup archive only |
| `emergency_restore_from_controller.yml` | Restore from AAP controller backup archive |
| `rhel_cis_query.yml` | Query current system CIS compliance status |

## Selective CIS Workflow

Audit and remediate only specific CIS controls instead of running the entire benchmark profile:

1. **Specify Rules**: List target CIS rule IDs to audit.
2. **Audit Selected**: Only those controls are evaluated against the datastream.
3. **Identify Failures**: Inspect which selected rules pass or fail.
4. **Remediate Failures**: Remediate only the failed rules.

### Example

```yaml
# Audit only these 3 specific rules
rhel_cis_rules_to_apply:
  - sysctl_crypto_fips_enabled
  - audit_rules_login_events
  - sshd_set_idle_timeout

# Leave empty for full benchmark audit
rhel_cis_rules_to_apply: []
```

## Organization Tailoring

Customize CIS benchmark control variables (timeouts, lockout limits, password lengths) without altering upstream roles.

### Quick Example

```bash
# Create your organization's tailoring profile directory
mkdir -p tailoring/myorg
cp tailoring/example_tailoring.yml tailoring/myorg/common.yml

# Edit with your organization's standard values
vim tailoring/myorg/common.yml

# Run hardening with tailoring enabled
ansible-playbook -i inventory playbooks/site.yml \
  -e rhel_cis_tailoring_profile=myorg
```

### Common Tailoring Variables

- `var_password_pam_minlen` - Minimum password length
- `var_accounts_tmout` - Shell session idle timeout (seconds)
- `var_accounts_passwords_pam_faillock_deny` - Failed login lockout attempts
- `sshd_idle_timeout_value` - SSH client idle timeout
- `var_auditd_max_log_file` - Audit log size limit

## Native Ansible Parsing

XML parsing uses native Ansible modules (`community.general.xml`) instead of custom Python scripts.

### Benefits

- 🔒 **Better Security** - No helper script distribution or raw script execution on targets
- 🚀 **Easier Maintenance** - Uses Ansible tasks and native facts
- ✅ **Built-in Resilience** - Pure XML node traversal directly within task flow
- 📦 **No Python Dependency Skews** - Handles `python-lxml` (RHEL 7) and `python3-lxml` (RHEL 8/9/10) automatically

To use legacy parsing scripts if needed:
```yaml
rhel_cis_use_native_parsing: false
```

## Role Variables

### Core Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `rhel_cis_benchmark_profile` | `cis_server_l1` | Benchmark profile (`cis_server_l1`, `cis_server_l2`, `cis_workstation_l1`, `cis_workstation_l2`) |
| `rhel_cis_apply_remediations` | `true` | Apply remediations after baseline audit |
| `rhel_cis_enable_auto_backup` | `true` | Create pre-hardening snapshot |
| `rhel_cis_auto_rollback_on_failure` | `true` | Auto-rollback if operability checks fail |

### Tailoring Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `rhel_cis_tailoring_profile` | `""` | Tailoring directory name under `tailoring/` (empty = defaults) |

### Selective Audit Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `rhel_cis_rules_to_apply` | `[]` | List of rule IDs to audit/remediate (empty = full profile) |

### Paths

| Variable | Default | Description |
|----------|---------|-------------|
| `rhel_cis_backup_root` | `/opt/cis_backups` | System backup location |
| `rhel_cis_audit_dir` | `/opt/cis_audit` | Audit HTML and XML reports directory |
| `rhel_cis_work_dir` | `/opt/cis_work` | Remediation script and log workspace |
| `rhel_cis_controller_backup_dir` | `/var/lib/awx/projects/cis_backups` | Controller backup archive destination |

See `roles/rhel_cis_with_rollback/defaults/main.yml` for all available variables.

## Requirements

- Target OS: Red Hat Enterprise Linux 7, 8, 9, or 10
- Packages:
  - `openscap-scanner`
  - `scap-security-guide`
  - `python-lxml` (RHEL 7) or `python3-lxml` (RHEL 8/9/10)
- Ansible 2.9+
- Ansible Collections:
  - `community.general` >= 8.0.0

Install collections:
```bash
ansible-galaxy collection install -r requirements.yml
```

The role automatically installs missing SCAP packages and the appropriate XML parsing dependencies based on detected OS facts.

## Common Operations

### Audit Current Compliance Only

```bash
ansible-playbook -i inventory playbooks/rhel_cis_audit_only.yml
```

### View Detailed Post-Hardening Report

Open the generated HTML report locally or in your browser:
```bash
firefox /opt/cis_audit/post_hardening_<timestamp>.html
```

### Perform Manual Rollback

```bash
ansible-playbook -i inventory playbooks/perform_rollback.yml \
  -e rhel_cis_backup_name=pre_cis_hostname_20261102T120000
```

### Emergency Restore from Controller Backup

```bash
ansible-playbook -i inventory playbooks/emergency_restore_from_controller.yml \
  -e target_host=hostname \
  -e backup_name=pre_cis_hostname_20261102T120000
```

## Project Structure

```
.
├── ansible.cfg                   # Ansible configuration
├── inventory                     # Default inventory
├── tailoring/                    # Organization-specific CIS variable overrides
│   ├── README.md                 # Tailoring documentation
│   ├── example_tailoring.yml     # Example tailoring template
│   └── <org_name>/               # Organization profile folder
│       ├── common.yml            # Overrides common to all RHEL releases
│       └── rhel{7,8,9,10}.yml    # Release-specific overrides
├── playbooks/                    # Orchestration playbooks
│   ├── rhel_cis_with_rollback.yml              # Main playbook (AAP compatible)
│   ├── rhel_cis_with_rollback_interactive.yml  # Interactive CLI
│   ├── perform_rollback.yml                    # Rollback execution
│   ├── rhel_cis_audit_only.yml                # Audit only execution
│   └── verify_system_operational.yml           # Standalone system health verification
├── roles/
│   └── rhel_cis_with_rollback/   # Main CIS Benchmark role
│       ├── tasks/
│       │   ├── load_tailoring.yml          # Loads tailoring variables
│       │   ├── phase1_validation.yml       # Host validation & SCAP verification
│       │   ├── phase2_backup.yml           # Tarball snapshot of configs & services
│       │   ├── phase3_baseline_audit.yml   # Initial CIS audit & compliance score
│       │   ├── phase4_remediation.yml      # Fix script generation & execution
│       │   ├── phase5_verification.yml     # Operational check & post-audit
│       │   ├── phase6_summary.yml          # Final summary & rollback guidance
│       │   ├── parse_oscap_score.yml       # Native XML score extractor
│       │   └── rollback.yml                # Automated recovery logic
│       ├── vars/
│       │   ├── RedHat-7.yml                # RHEL 7 package definitions
│       │   ├── RedHat-8.yml                # RHEL 8 package definitions
│       │   ├── RedHat-9.yml                # RHEL 9 package definitions
│       │   └── RedHat-10.yml               # RHEL 10 package definitions
│       └── defaults/main.yml               # Default role configurations
└── docs/                         # Documentation guides
```

## Workflow Phases

1. **Phase 1: Validation** - Assert RHEL 7–10 version, load OS-specific variable definitions, verify SCAP datastreams, and install packages.
2. **Phase 2: Backup** - Archive critical configurations (`/etc/pam.d`, `sshd_config`, `sudoers`, etc.) and record running service states.
3. **Phase 3: Baseline Audit** - Execute baseline OSCAP CIS audit and calculate pre-hardening compliance score.
4. **Phase 4: Remediation** - Generate and execute SCAP remediation scripts (full profile or filtered rules).
5. **Phase 5: Verification** - Verify host operability (SSH connectivity, PAM integrity, filesystem state, critical daemons) and run post-scan.
6. **Phase 6: Summary** - Calculate compliance improvement delta, print final summary report, and output rollback instructions.

## License

MIT