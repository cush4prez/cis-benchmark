# Playbook Usage Guide

## Recommended Playbooks (Native Parsing Enabled)

All recommended playbooks invoke the `rhel_cis_with_rollback` role, which uses native Ansible XML parsing (`community.general.xml`) by default.

### ✅ Production Playbooks (Use These)

| Playbook | Use Case | Native Parsing |
|----------|----------|----------------|
| `site.yml` | Role invocation (Recommended for AAP) | ✅ Yes |
| `rhel_cis_with_rollback_interactive.yml` | Guided CLI deployment with terminal prompts | ✅ Yes |

### ⚠️ Legacy vs. Modern Workflow

Avoid standalone shell-script parsing or legacy wrapper playbooks with duplicate `post_tasks`. Always invoke `playbooks/site.yml` so that pre-flight checks, backup, evaluation, and verification run through unified role tasks.

---

## Quick Start

### For AAP / AWX Deployment

Configure your Job Template:
- **Playbook**: `playbooks/site.yml`
- **Execution Environment**: `RHEL CIS EE` (with `community.general >= 8.0.0`)
- **Survey**: Configured with `cis_audit_mode`, `rhel_cis_benchmark_profile`, etc.

### For CLI Deployment

```bash
# Interactive guided mode
ansible-playbook -i inventory playbooks/rhel_cis_with_rollback_interactive.yml

# Automated mode with custom tailoring
ansible-playbook -i inventory playbooks/site.yml \
  -e rhel_cis_benchmark_profile=cis_server_l1 \
  -e rhel_cis_tailoring_profile=myorg
```

---

## Verifying Native Parsing

Confirm native parsing is enabled in role defaults:

```bash
grep "use_native_parsing" roles/rhel_cis_with_rollback/defaults/main.yml
# Output:
# rhel_cis_use_native_parsing: true
```

---

## Playbook Comparison

### `site.yml` (Recommended for Automation)
- **Role Invocation**: Direct role execution
- **Code Footprint**: Clean, minimal playbook wrapper
- **AAP Compatible**: Fully compatible with surveys and workflows

### `rhel_cis_with_rollback_interactive.yml` (Recommended for Manual CLI)
- **User Interface**: Formatted CLI prompts for profile, rules, and rollback
- **Target Audience**: Systems administrators executing manual, ad-hoc runs
- **AAP Compatible**: No (requires standard terminal interaction)

---

## Playbook Selection Matrix

| Objective | Playbook to Run |
|-----------|-----------------|
| Harden system with CIS Benchmark | `site.yml` or `rhel_cis_with_rollback_interactive.yml` |
| Check baseline compliance without changes | `rhel_cis_audit_only.yml` |
| Generate executive compliance report | `final_audit_report.yml` |
| Roll back to pre-hardening snapshot | `perform_rollback.yml` |
| Emergency restore from controller backup | `emergency_restore_from_controller.yml` |
| Validate host health & accessibility | `verify_system_operational.yml` |
| Validate organizational tailoring files | `test_tailoring.yml` |