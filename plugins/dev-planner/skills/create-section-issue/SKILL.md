# /create-section-issue

Create a section issue representing a parallel work stream in the development plan.

## Arguments

- `repo`: Repository in `owner/repo` format (required)
- `prd_issue`: Parent PRD issue number (required)
- `title`: Section title (required)
- `overview`: Description of the section's purpose (required)
- `tasks`: List of task titles that will be created under this section (optional)
- `depends_on`: List of section issue numbers this depends on (optional)
- `acceptance_criteria`: List of acceptance criteria (optional)
- `notes`: Additional technical notes (optional)

## Instructions

### Step 1: Prepare Issue Content

Format the issue body following @rules/issue-structure.md:

```markdown
## Overview

[overview argument]

## Tasks

[Will be populated with task issue links after tasks are created]

## Dependencies

**Depends on:** [List depends_on issues or "None - can start immediately"]
**Blocks:** [To be determined]

## Acceptance Criteria

- [ ] [criterion 1]
- [ ] [criterion 2]

## Notes

[notes argument or "None"]
```

### Step 2: Create the Issue

Use the GitHub MCP server `create_issue` tool to create the issue:

```
mcp_github_create_issue(
  owner: "[owner]",
  repo: "[repo]",
  title: "[title]",
  body: "[formatted body]",
  labels: ["type:section"]
)
```

If the `type:section` label doesn't exist, create without the label and note this for the user.

### Step 3: Link to PRD as Sub-Issue

Use the GitHub MCP server `add_sub_issue` tool to link the section as a sub-issue of the PRD:

```
mcp_github_add_sub_issue(
  owner: "[owner]",
  repo: "[repo]",
  issue_number: [prd_issue],
  sub_issue_id: [new-issue-number]
)
```

Note: If sub-issue linking fails, add a reference in the issue body instead:
"Parent PRD: #[prd_issue]"

### Step 4: Return Result

Output the created issue details:

```markdown
## Section Issue Created

**Issue:** #[number]
**Title:** [title]
**URL:** [url]
**Parent PRD:** #[prd_issue]
**Dependencies:** [depends_on or "None"]

Ready for task creation.
```

## Error Handling

- **Label doesn't exist**: Create issue without label, inform user to create `type:section` label
- **Permission denied**: Inform user to check GITHUB_PERSONAL_ACCESS_TOKEN has 'repo' scope
- **Sub-issue linking fails**: Fall back to text reference in issue body ("Parent PRD: #[prd_issue]")
- **Rate limited**: Wait and retry, inform user of delay
- **MCP server error**: Check connectivity to https://api.githubcopilot.com/mcp/ and GitHub authentication

## Example

Input:
```
/create-section-issue
  repo: acme/project
  prd_issue: 100
  title: "Data Layer: User and Authentication Models"
  overview: "Implement database schema and repository layer for user authentication. Includes user model, session management, and password reset tokens."
  depends_on: []
  acceptance_criteria:
    - "User model supports email, hashed password, and roles"
    - "Repository provides CRUD operations with proper error handling"
    - "All database operations are tested"
  notes: "Use PostgreSQL with existing connection pool. Follow repository pattern from existing codebase."
```

Output:
```markdown
## Section Issue Created

**Issue:** #101
**Title:** Data Layer: User and Authentication Models
**URL:** https://github.com/acme/project/issues/101
**Parent PRD:** #100
**Dependencies:** None

Ready for task creation.
```

Created issue body:
```markdown
## Overview

Implement database schema and repository layer for user authentication. Includes user model, session management, and password reset tokens.

## Tasks

[Tasks will be linked here after creation]

## Dependencies

**Depends on:** None - can start immediately
**Blocks:** TBD

## Acceptance Criteria

- [ ] User model supports email, hashed password, and roles
- [ ] Repository provides CRUD operations with proper error handling
- [ ] All database operations are tested

## Notes

Use PostgreSQL with existing connection pool. Follow repository pattern from existing codebase.
```
