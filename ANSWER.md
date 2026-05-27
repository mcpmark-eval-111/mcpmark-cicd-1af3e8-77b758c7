# Issue Management Automation Workflow

This repository has been configured with an intelligent Issue Management Automation workflow to automatically triage issues, break down Epic issues into concrete sub-tasks, and send tailored auto-responses.

## Workflow Overview

The workflow is defined in `.github/workflows/issue-automation.yml` and triggers on `issues` (`opened` and `labeled`) events.

### 1. Auto-Triage Job (`issue-triage`)
- **Category Assignment**: Scans the issue title (case-insensitive) for category keywords:
  - `"bug"` → Adds the `bug` label
  - `"epic"` → Adds the `epic` label
  - `"maintenance"` → Adds the `maintenance` label
- **Priority Assignment**: Scans both the issue title and body (case-insensitive) for priority keywords:
  - Critical priority keywords (`"critical"`, `"urgent"`, `"production"`, `"outage"`) → `priority-critical`
  - High priority keywords (`"important"`, `"high"`, `"blocking"`) → `priority-high`
  - Low priority keywords (`"low"`, `"nice-to-have"`, `"minor"`) → `priority-low`
  - Default / Medium priority keywords (`"medium"`, `"normal"`) or no matches → `priority-medium`
- **Initial Status**: All processed issues receive the `needs-triage` label initially.

### 2. Task Decomposition Job (`task-breakdown`)
- Executed only for issues containing `"Epic"` in the title.
- Automatically spawns exactly 4 sub-issues named in the pattern: `[SUBTASK] [Original Title] - Task N: [Task Name]`.
- Standardized task names are:
  1. `Requirements Analysis`
  2. `Design and Architecture`
  3. `Implementation`
  4. `Testing and Documentation`
- Links sub-issues to parent using `"Related to #[parent-number]"` in their bodies.
- Adds `enhancement` and `needs-review` labels to all sub-issues.
- Modifies the parent Epic issue with a `"## Epic Tasks"` checklist pointing to the sub-issue numbers.
- Contains advanced idempotency checks to prevent duplicate sub-task creation if the issue is re-labeled.

### 3. Auto-Response Job (`auto-response`)
- Checks if the user is a first-time contributor specifically in this repository.
- If they are, applies the `first-time-contributor` label and prefixes the comment with a friendly welcome message.
- Appends distinct guidelines depending on the category:
  - **bug**: Contains `"Bug Report Guidelines"`
  - **epic**: Contains `"Feature Request Process"`
  - **maintenance**: Contains `"Maintenance Guidelines"`
- Automates release tracking by assigning the `"v1.0.0"` milestone to any `priority-high` or `priority-critical` issues (auto-creating the milestone if it does not yet exist).
- Updates the status label from `needs-triage` to `needs-review` once completed.

---

## Supporting Files Created

We created and placed standard markdown issue templates inside the `.github/ISSUE_TEMPLATE` folder to guide developers when opening issues:

1. **Bug Report** (`.github/ISSUE_TEMPLATE/bug_report.md`): Gathers steps to reproduce, expected vs actual behavior, and context.
2. **Feature Request** (`.github/ISSUE_TEMPLATE/feature_request.md`): Prompts for feature description, proposed solution, and alternatives considered.
3. **Maintenance Report** (`.github/ISSUE_TEMPLATE/maintenance_report.md`): Inquires about task details, necessity, and implementation plans.

---

## Verification & Testing

The system has been successfully verified across multiple test issues:
- **Bug Issue**: Triggered triage, assigned `bug` and `priority-high`, linked milestone `"v1.0.0"`, commented with `"Bug Report Guidelines"`, and moved status from `needs-triage` to `needs-review`.
- **Epic Issue**: Triaged as `epic` and `priority-high`, generated 4 sub-issues labeled `enhancement`/`needs-review`, updated parent checklist, set milestone `"v1.0.0"`, commented with `"Feature Request Process"`, and moved status from `needs-triage` to `needs-review`.
- **Maintenance Issue**: Triaged as `maintenance` and `priority-medium`, commented with `"Maintenance Guidelines"`, and moved status from `needs-triage` to `needs-review`.
