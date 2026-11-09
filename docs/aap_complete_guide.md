# Ansible Automation Platform (AAP) 2.7 - Complete Setup Guide

Complete guide for deploying RHEL CIS Benchmark hardening using Ansible Automation Platform 2.7.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Step 1: Create Execution Environment](#step-1-create-execution-environment)
- [Step 2: Create Project](#step-2-create-project)
- [Step 3: Create Inventory](#step-3-create-inventory)
- [Step 4: Create Job Templates](#step-4-create-job-templates)
- [Step 5: Configure Surveys](#step-5-configure-surveys)
- [Running Jobs](#running-jobs)
- [Monitoring and Results](#monitoring-and-results)
- [Emergency Recovery](#emergency-recovery)
- [Troubleshooting](#troubleshooting)

---

## Overview

This role is fully compatible with AAP 2.7 and provides:

- ✅ Survey-based GUI configuration
- ✅ Selective or full CIS Benchmark auditing
- ✅ Support for CIS Server Level 1/2 and Workstation Level 1/2 profiles
- ✅ Automatic backup with disaster recovery
- ✅ Intelligent auto-rollback on operational failures
- ✅ RHEL 7, 8, 9, and 10 support
- ✅ Native Ansible parsing (no Python scripts required)

---

## Prerequisites

### Required

- **AAP 2.6 or later** installed and operational
- **Git repository** containing this project
- **RHEL hosts** to harden (RHEL 7, 8, 9, or 10)
- **Credentials** with sudo/root access to target hosts
- **Target host prerequisites**: `python-lxml` (RHEL 7) or `python3-lxml` (RHEL 8/9/10)

### Recommended

- **Red Hat Satellite** or **RHEL subscription** for SCAP content
- **Backup storage** on AAP controller: `/var/lib/awx/projects/cis_backups/`

---

## Step 1: Create Execution Environment

The role requires the `community.general` collection version 8.0.0+ and `ansible.posix` >= 1.3.0.

### Option A: Use Existing EE (if it has community.general 8.0+)

1. Go to **Administration → Execution Environments**
2. Verify an environment contains `community.general >= 8.0.0`
3. Note the name for template configuration

### Option B: Create Custom EE

#### Step B.1: Create EE Definition Files

```bash
mkdir -p ~/rhel-cis-ee
cd ~/rhel-cis-ee
```

Create `execution-environment.yml`:

```yaml
---
version: 3

images:
  base_image:
    name: registry.redhat.io/ansible-automation-platform-24/ee-minimal-rhel9:latest

dependencies:
  galaxy: requirements.yml
  python:
    - python3-lxml

additional_build_steps:
  append_final:
    - RUN microdnf install -y python3-lxml && microdnf clean all
```

Create `requirements.yml`:

```yaml
---
collections:
  - name: community.general
    version: ">=8.0.0"
  - name: ansible.posix
    version: ">=1.3.0"
```

#### Step B.2: Build and Push the EE

```bash
# Build
ansible-builder build --tag rhel-cis-ee:1.0.0 --container-runtime podman

# Push to your registry (e.g., Quay.io)
podman login quay.io
podman tag rhel-cis-ee:1.0.0 quay.io/your-org/rhel-cis-ee:latest
podman push quay.io/your-org/rhel-cis-ee:latest
```

#### Step B.3: Add EE to AAP

1. **Administration → Execution Environments → Add**
2. Name: `RHEL CIS EE`
3. Image: `quay.io/your-org/rhel-cis-ee:latest`
4. Pull: `Always` (or `If not present`)
5. Save

---

## Step 2: Create Project

1. **Resources → Credentials → Add** (if repository is private):
   - Type: `Source Control`
   - Provide your Git PAT or SSH private key
2. **Resources → Projects → Add**:
   - **Name**: `RHEL CIS Benchmark`
   - **Organization**: Your organization
   - **Execution Environment**: `RHEL CIS EE`
   - **Source Control Type**: `Git`
   - **Source Control URL**: `https://github.com/yourorg/rhel-cis-benchmark.git`
   - **Source Control Branch**: `main`
   - **Options**: Clean, Delete on Update, Update Revision on Launch
3. Save and click **Sync**

---

## Step 3: Create Inventory

### Option A: Manual Inventory
Create inventory named `RHEL Servers` and add target RHEL 7–10 hostnames and IP addresses under **Hosts**.

### Option B: Sourced from Project
1. Create `inventories/rhel_servers.yml` in your Git repo.
2. In AAP: **Resources → Inventories → Add → Inventory** (`RHEL Servers (Sourced)`).
3. Under **Sources**, add a source pointing to `inventories/rhel_servers.yml` with **Overwrite** and **Update on project update** checked.

---

## Step 4: Create Job Templates

### Template 1: CIS Hardening (Primary - with Survey)

1. **Resources → Templates → Add → Add job template**
2. Configure **Details**:
   - **Name**: `CIS - Full Hardening`
   - **Job Type**: `Run`
   - **Inventory**: `RHEL Servers`
   - **Project**: `RHEL CIS Benchmark`
   - **Execution Environment**: `RHEL CIS EE`
   - **Playbook**: `playbooks/site.yml` ⭐
   - **Credentials**: Machine credential with root/sudo escalation
   - **Variables**:
     ```yaml
     ---
     rhel_cis_controller_backup_dir: /var/lib/awx/projects/cis_backups
     rhel_cis_fetch_to_controller: true
     rhel_cis_use_native_parsing: true
     rhel_cis_auto_install_deps: true
     rhel_cis_auto_detect_python: true
     ```
   - **Options**: ☑ Privilege Escalation, ☑ Enable Fact Storage
3. **Save**

### Template 2: Audit Only (No Changes)

- **Name**: `CIS - Audit Only`
- **Playbook**: `playbooks/rhel_cis_audit_only.yml`
- **Options**: Privilege Escalation

### Template 3: Rollback

- **Name**: `CIS - Rollback`
- **Playbook**: `playbooks/perform_rollback.yml`
- **Options**: Privilege Escalation, Prompt on launch (Variables)

### Template 4: Final Audit Report

- **Name**: `CIS - Final Audit Report`
- **Playbook**: `playbooks/final_audit_report.yml`
- **Options**: Privilege Escalation

---

## Step 5: Configure Surveys

Create the survey questions manually on `CIS - Full Hardening`:

#### Question 1: CIS Audit Mode
- **Variable Name**: `cis_audit_mode`
- **Question**: `CIS Audit Mode`
- **Type**: `Multiple Choice (single select)`
- **Choices**:
  ```
  full
  selective
  ```
- **Default**: `full` (Required: Yes)

#### Question 2: CIS Profile
- **Variable Name**: `rhel_cis_benchmark_profile`
- **Question**: `CIS Benchmark Profile`
- **Type**: `Multiple Choice (single select)`
- **Choices**:
  ```
  cis_server_l1
  cis_server_l2
  cis_workstation_l1
  cis_workstation_l2
  ```
- **Default**: `cis_server_l1` (Required: Yes)

#### Question 3: CIS Rules to Audit (Selective Mode Only)
- **Variable Name**: `cis_rules_input`
- **Question**: `CIS Rules to Audit (comma-separated)`
- **Type**: `Textarea`
- **Default**: *(blank)* (Required: No)

#### Question 4: Apply Remediations
- **Variable Name**: `rhel_cis_apply_remediations`
- **Question**: `Apply Remediations?`
- **Type**: `Multiple Choice (single select)`
- **Choices**: `true`, `false`
- **Default**: `true` (Required: Yes)

#### Question 5: Auto Rollback on Failure
- **Variable Name**: `rhel_cis_auto_rollback_on_failure`
- **Question**: `Auto Rollback on Failure?`
- **Type**: `Multiple Choice (single select)`
- **Choices**: `true`, `false`
- **Default**: `true` (Required: Yes)

#### Question 6: Organization Tailoring Profile
- **Variable Name**: `rhel_cis_tailoring_profile`
- **Question**: `Organization Tailoring Profile (optional)`
- **Type**: `Text`
- **Default**: *(blank)* (Required: No)

Toggle **Survey Enabled** to ON and save.

---

## Monitoring and Results

### Job Execution Phases
1. **Phase 1: Validation** - Validates RHEL 7–10, loads OS variables (`RedHat-*.yml`), installs SCAP datastreams.
2. **Phase 2: Backup** - Archives critical config files (`/etc/pam.d`, `sshd_config`, `sudoers`, audit rules) and copies archive to controller.
3. **Phase 3: Baseline Audit** - Runs `oscap xccdf eval` against target CIS profile, parsing score natively.
4. **Phase 4: Remediation** - Generates and applies CIS fixes.
5. **Phase 5: Verification** - Checks SSH accessibility, PAM integrity, filesystem state, and services.
6. **Phase 6: Summary** - Compares pre- and post-hardening compliance scores and prints rollback guidelines.

---

## Emergency Recovery

### Console Recovery
If SSH is blocked after hardening:
1. Access machine via VM or IPMI console.
2. Extract the local snapshot:
   ```bash
   cd /
   tar -xzf /opt/cis_backups/pre_cis_<hostname>_<timestamp>.tar.gz
   systemctl restart sshd
   ```

### Restore from AAP Controller
Run the `CIS - Rollback` template specifying:
```yaml
---
rhel_cis_backup_name: pre_cis_hostname_20261102T120000
```