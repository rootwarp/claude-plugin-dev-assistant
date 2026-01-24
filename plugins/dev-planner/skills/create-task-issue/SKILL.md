# /create-task-issue

Create a task issue representing a sequential work item within a section.

## Arguments

- `repo`: Repository in `owner/repo` format (required)
- `section_issue`: Parent section issue number (required)
- `title`: Task title - should start with action verb (required)
- `description`: Detailed description of the task (required)
- `acceptance_criteria`: List of specific, testable criteria (required)
- `complexity`: Task size - S (1-2h), M (2-4h), or L (4-8h) (required)
- `blocked_by`: List of task issue numbers this is blocked by (optional)
- `technical_notes`: Implementation hints and references (optional)

## Instructions

### Step 1: Prepare Issue Content

Format the issue body following @rules/issue-structure.md:

```markdown
## Description

[description argument]

## Acceptance Criteria

- [ ] [criterion 1]
- [ ] [criterion 2]
- [ ] [criterion 3]

## Technical Notes

[technical_notes argument or "None"]

## Dependencies

**Blocked by:** [List blocked_by issues with links, or "None - can start immediately"]
**Part of:** #[section_issue]
```

### Step 2: Create the Issue

Use the GitHub MCP server `create_issue` tool to create the issue with appropriate labels:

```
mcp_github_create_issue(
  owner: "[owner]",
  repo: "[repo]",
  title: "[title]",
  body: "[formatted body]",
  labels: ["type:task", "complexity:[S|M|L]"]
)
```

If labels don't exist, create without them and note this for the user.

### Step 3: Link to Section as Sub-Issue

Use the GitHub MCP server `add_sub_issue` tool to link the task as a sub-issue of the section:

```
mcp_github_add_sub_issue(
  owner: "[owner]",
  repo: "[repo]",
  issue_number: [section_issue],
  sub_issue_id: [new-issue-number],
  replace_parent: false
)
```

**Parameters:**
- `issue_number`: The parent section issue number (from `number` field, NOT `id`)
- `sub_issue_id`: The newly created task issue number (from `number` field, NOT `id`)
- `replace_parent`: Set to `true` if moving an existing task to a different section

**Warning:** Use the `number` field from the create_issue response, not the `id` field. Using `id` (like 3850680229) will cause 404 errors.

**Fallback:** If sub-issue linking fails, the "Part of" reference in the body serves as the link.

### Step 3b: Reorder Tasks (Optional)

If tasks need to be displayed in a specific order within the section, use reprioritize:

```
mcp_github_reprioritize_sub_issue(
  owner: "[owner]",
  repo: "[repo]",
  issue_number: [section_issue],
  sub_issue_id: [task-to-move],
  after_id: [task-number]    # Place after this task
  # OR: before_id: [task-number]  # Place before this task
)
```

This is useful when tasks are created out of order but should appear in dependency sequence.

### Step 4: Return Result

Output the created issue details:

```markdown
## Task Issue Created

**Issue:** #[number]
**Title:** [title]
**URL:** [url]
**Section:** #[section_issue]
**Complexity:** [S|M|L]
**Blocked by:** [blocked_by list or "None"]
```

## Error Handling

- **Labels don't exist**: Create issue without labels, list missing labels for user
- **Permission denied**: Inform user to check GITHUB_PERSONAL_ACCESS_TOKEN has 'repo' scope
- **Invalid complexity**: Default to M (Medium) and warn user
- **Blocked-by issue doesn't exist**: Create issue but warn about invalid reference
- **Sub-issue linking fails**: Fall back to text reference in issue body ("Part of: #[section_issue]")
- **MCP server error**: Check connectivity to https://api.githubcopilot.com/mcp/ and GitHub authentication

## Example

Input:
```
/create-task-issue
  repo: acme/project
  section_issue: 101
  title: "Design user schema with role support"
  description: "Create the database schema for the users table including support for multiple roles (admin, user, guest). Schema should support soft deletes and audit timestamps."
  acceptance_criteria:
    - "Schema includes: id, email, password_hash, role, created_at, updated_at, deleted_at"
    - "Email has unique constraint"
    - "Role is an enum type with defined values"
    - "Migration file is created and tested"
  complexity: M
  blocked_by: []
  technical_notes: "Use PostgreSQL enum type for role. Reference existing migrations in db/migrations/ for style. Consider adding an index on email for login queries."
```

Output:
```markdown
## Task Issue Created

**Issue:** #104
**Title:** Design user schema with role support
**URL:** https://github.com/acme/project/issues/104
**Section:** #101
**Complexity:** M
**Blocked by:** None
```

Created issue body:
```markdown
## Description

Create the database schema for the users table including support for multiple roles (admin, user, guest). Schema should support soft deletes and audit timestamps.

## Acceptance Criteria

- [ ] Schema includes: id, email, password_hash, role, created_at, updated_at, deleted_at
- [ ] Email has unique constraint
- [ ] Role is an enum type with defined values
- [ ] Migration file is created and tested

## Technical Notes

Use PostgreSQL enum type for role. Reference existing migrations in db/migrations/ for style. Consider adding an index on email for login queries.

## Dependencies

**Blocked by:** None - can start immediately
**Part of:** #101
```

## Batch Creation

When creating multiple tasks, create them in dependency order:
1. First create tasks with no dependencies
2. Then create tasks blocked by the first batch (now you have issue numbers)
3. Continue until all tasks are created

This ensures blocked-by references have valid issue numbers.
