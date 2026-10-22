# Terraform State Recovery Lab

A hands-on infrastructure engineering lab exploring Terraform state, resource reconciliation, troubleshooting, and validation using the Terraform local provider.

## Project Overview

Terraform uses state to track the relationship between resource configurations and the resources it manages. When the configuration, state, and actual environment do not align, Terraform may propose changes that require careful investigation.

This lab explores a state discrepancy in a controlled local environment and documents the investigation, recovery workflow, and validation steps.

**Objectives**
- Understand how Terraform state relates to resource configuration.
- Investigate a discrepancy between current and backup state.
- Reconcile a resource using Terraform's normal plan-and-apply workflow.
- Validate the resulting state and confirm that the configuration is stable.
- Apply basic Git hygiene to prevent state files and saved plans from being published.

## Environment

- **OS:** Ubuntu 26.04.1 LTS (x86_64), running in VMware Fusion
- **Terraform:** v1.16.4
- **Version control:** Git
- **Provider:** `hashicorp/local`
- **Resource:** `local_file.hello`

The lab uses a local file resource rather than provisioning cloud infrastructure.

## Incident and Investigation

During the lab, the current Terraform state reported zero managed resources, while the backup state contained the expected `local_file.hello` resource.

The discrepancy was confirmed by inspecting the state and reviewing the backup. The backup provided useful evidence of the resource's prior recorded configuration, but it did not independently establish why the current state no longer listed the resource.

**Finding:** The state discrepancy was observed and investigated, but its original root cause was not conclusively determined.

This distinction matters in incident analysis: observed facts should be separated from hypotheses about what caused them.

## Recovery Workflow

The resource was reconciled through Terraform's normal workflow:

1. **Inspect the configuration and state.** Review the Terraform configuration, current resource list, and available backup evidence.
2. **Generate a plan.** Run `terraform plan` and inspect the proposed action before making changes.
3. **Apply the reviewed plan.** Apply the saved plan to create the resource represented by the configuration.
4. **Verify the resource.** Use `terraform state list` to confirm that `local_file.hello` is tracked.
5. **Validate convergence.** Run `terraform plan` again and confirm that Terraform reports no further changes.

### Validation Commands

```bash
terraform validate
terraform state list
terraform plan
git status --short --branch
```

Expected state-list output after reconciliation:

```text
local_file.hello
```

The subsequent Terraform plan reported no changes, indicating that the configuration and Terraform's recorded state were aligned for this lab at that point.

## Security and Version Control

Terraform state can contain sensitive infrastructure details and, depending on the resources involved, secret values. Saved plan files may also contain sensitive information.

The repository therefore excludes runtime and potentially sensitive artifacts, including:

- `terraform.tfstate` and state backups
- `.terraform/` working-directory contents
- Saved plan files such as `tfplan`
- Variable files that may contain credentials or other sensitive values

The provider lock file, Terraform configuration, README, and sample output file are tracked.

**Important:** `.gitignore` helps prevent accidental additions, but it does not remove files that are already tracked or erase information from Git history. Before publishing, inspect the tracked file list and review the changes being committed.

## Lessons Learned

- Terraform state is operationally important and should be protected and backed up appropriately.
- A backup can provide useful recovery evidence, but it should be evaluated rather than assumed to explain the root cause.
- A successful apply is not the end of validation; checking the resulting state and running another plan helps confirm the expected outcome.
- Git history, ignored files, and tracked files should be reviewed before publishing infrastructure projects.
- In a production incident, preserve evidence and follow the team's approved recovery procedure before modifying state.

## Scope and Limitations

This is a local learning lab, not a production infrastructure recovery exercise. It does not demonstrate recovery of a remote state backend, cloud resources, or a multi-user Terraform workflow.

The original cause of the state discrepancy remains undetermined. The lab documents the observed evidence and the successful reconciliation without claiming otherwise.

## Future Improvements

- Add repeatable checks to validate the configuration and expected resource state.
- Document safe state-backup and recovery procedures in a separate exercise.
- Explore remote state, state locking, and collaborative workflows in a disposable test environment.
- Add automated validation through a CI workflow.

## Repository Hygiene

Before each push, review:

```bash
git status --short
git diff --check
git diff
git ls-tree -r --name-only HEAD
```

Never publish real state files, credentials, or sensitive plan artifacts.
