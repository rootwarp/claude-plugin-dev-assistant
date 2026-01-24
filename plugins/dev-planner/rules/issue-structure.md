# GitHub Issue Structure

## Issue Hierarchy

```
PRD Issue (type:prd)
├── Section Issue (type:section) - parallel work stream
│   ├── Task Issue (type:task) - sequential within section
│   ├── Task Issue (type:task)
│   └── Task Issue (type:task)
├── Section Issue (type:section) - can run in parallel
│   ├── Task Issue (type:task)
│   └── Task Issue (type:task)
└── Section Issue (type:section)
    └── ...
```

## Label Conventions

### Type Labels (required)
- `type:prd` - Product Requirements Document (top-level)
- `type:section` - Parallel work stream grouping
- `type:task` - Individual work item

### Complexity Labels (for tasks)
- `complexity:S` - Small (1-2 hours)
- `complexity:M` - Medium (2-4 hours)
- `complexity:L` - Large (4-8 hours)

### Optional Labels
- `priority:high` - Should be worked on first
- `priority:low` - Can be deferred
- `blocked` - Currently blocked by dependencies

## Section Issue Template

```markdown
## Overview

[Brief description of this work stream and its purpose]

## Tasks

- [ ] #[task-number] - [Task title]
- [ ] #[task-number] - [Task title]
- [ ] #[task-number] - [Task title]

## Dependencies

**Depends on:** [List section issues this depends on, if any]
**Blocks:** [List section issues blocked by this, if any]

## Acceptance Criteria

- [ ] [Criterion 1]
- [ ] [Criterion 2]

## Notes

[Any additional context, technical decisions, or constraints]
```

## Task Issue Template

```markdown
## Description

[Clear description of what needs to be done]

## Acceptance Criteria

- [ ] [Specific, testable criterion]
- [ ] [Specific, testable criterion]

## Technical Notes

[Implementation hints, API references, related code locations]

## Dependencies

**Blocked by:** #[issue-number] - [reason]
```

## Linking Strategy

### Parent-Child Relationships

Use GitHub's sub-issue feature to establish hierarchy via the REST API or MCP tools:

**MCP Server Tools:**
```
# Link section as sub-issue of PRD
mcp_github_add_sub_issue(
  owner: "[owner]",
  repo: "[repo]",
  issue_number: [prd-number],      # parent issue
  sub_issue_id: [section-number]   # child issue to add
)

# Link task as sub-issue of section
mcp_github_add_sub_issue(
  owner: "[owner]",
  repo: "[repo]",
  issue_number: [section-number],  # parent issue
  sub_issue_id: [task-number]      # child issue to add
)
```

**REST API Endpoints:**
```
# Add sub-issue: POST /repos/{owner}/{repo}/issues/{issue_number}/sub_issues
# Body: { "sub_issue_id": [child-issue-number], "replace_parent": false }

# List sub-issues: GET /repos/{owner}/{repo}/issues/{issue_number}/sub_issues

# Get parent: GET /repos/{owner}/{repo}/issues/{issue_number}/parent

# Remove sub-issue: DELETE /repos/{owner}/{repo}/issues/{issue_number}/sub_issue
# Body: { "sub_issue_id": [child-issue-number] }

# Reprioritize: PATCH /repos/{owner}/{repo}/issues/{issue_number}/sub_issues/priority
# Body: { "sub_issue_id": [id], "after_id": [id] } or { "sub_issue_id": [id], "before_id": [id] }
```

**Critical:** The `sub_issue_id` parameter expects the issue `number` (e.g., 7), not the internal `id` (e.g., 3850680229). Using the wrong ID type causes 404 errors.

**Note:** If an issue already has a parent and you want to move it to a new parent, use `replace_parent: true` in the add sub-issue request.

### Sequential Dependencies

For tasks that must be completed in order within a section:

1. Add "Blocked by: #[number]" in the issue body
2. Create task list in parent section showing order
3. Use GitHub's sub-issue relationships where appropriate

### Cross-Section Dependencies

When a section depends on another section:

1. Document in section's Dependencies field
2. Ensure blocking section is created first
3. Consider if dependency can be reduced via interfaces

## Naming Conventions

### Section Titles
Format: `[Area]: [Brief Description]`

Examples:
- `Data Layer: User and Authentication Models`
- `API: REST Endpoints for User Management`
- `UI: Authentication Components`

### Task Titles
Format: `[Action verb] [specific item]`

Examples:
- `Design user schema with role support`
- `Implement JWT token generation`
- `Create login form component`
- `Add input validation to signup endpoint`
