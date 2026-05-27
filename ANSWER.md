# Issue Management Automation Workflow

I have successfully created and deployed an intelligent Issue Management Automation workflow for this Node.js project.

## Workflow Overview
The workflow is defined in `.github/workflows/issue-automation.yml` and triggers on issue `opened` and `labeled` events. It consists of three jobs:

1. **issue-triage**:
   - Auto-assigns category labels (`bug`, `epic`, `maintenance`) based on title keywords (case-insensitive).
   - Auto-assigns priority labels (`priority-critical`, `priority-high`, `priority-medium`, `priority-low`) based on case-insensitive title or body keywords (highest priority keyword wins, defaults to `priority-medium` if none found).
   - Dynamically ensures all required category, priority, and status labels exist in the repository on every run.
   - Adds the status label `needs-triage` initially.

2. **task-breakdown**:
   - For issue titles containing the word `Epic` (and not already broken down), it automatically creates exactly 4 sub-issues for each project phase:
     1. Requirements Analysis
     2. Design and Architecture
     3. Implementation
     4. Testing and Documentation
   - Sets the sub-issue title pattern to: `[SUBTASK] [Original Title] - Task N: [Task Name]`.
   - Populates each sub-issue body with `Related to #[parent-number]` to link back to the parent epic.
   - Updates the parent epic issue body with a Markdown checklist `## Epic Tasks` linking directly to the created sub-issues.
   - Applies `enhancement` and `needs-review` labels to all sub-issues.

3. **auto-response**:
   - Checks if the author has created previous issues in this repository. If they are a first-time contributor in this repo, it adds the label `first-time-contributor` and posts a welcome message.
   - Comments with category-specific guidelines:
     - Bug issues get `Bug Report Guidelines` reference.
     - Epic issues get `Feature Request Process` reference.
     - Maintenance issues get `Maintenance Guidelines` reference.
   - Assigns the issue to milestone `v1.0.0` (dynamically creating it if missing) for `priority-high` and `priority-critical` issues.
   - Transitions the status label from `needs-triage` to `needs-review` after response.

## Issue Templates Created
Three issue templates have been added to `.github/ISSUE_TEMPLATE/`:
- `bug_report.md` (Bug report template)
- `feature_request.md` (Feature request/Epic template)
- `maintenance_report.md` (Maintenance task template)

## Test Results
All three target test scenarios have run successfully with the workflow runs reporting `success`:
1. **Bug Issue**: Automatically triaged to `bug` and `priority-critical` (due to "blocking" keyword), transitioned to `needs-review`, added to milestone `v1.0.0`, and commented on with "Bug Report Guidelines".
2. **Epic Issue**: Automatically triaged to `epic` and `priority-high`, broken down into exactly 4 linked sub-issues with check-boxes, transitioned to `needs-review`, added to milestone `v1.0.0`, and commented on with "Feature Request Process".
3. **Maintenance Issue**: Automatically triaged to `maintenance` and `priority-medium`, transitioned to `needs-review`, and commented on with "Maintenance Guidelines".
