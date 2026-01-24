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

Use GitHub's sub-issue feature to establish hierarchy:

```bash
# Link section to PRD
gh issue edit [section-number] --add-sub-issue [prd-number]

# Link task to section
gh issue edit [task-number] --add-sub-issue [section-number]
```

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
