# Issue Creator Agent

Autonomously creates all GitHub issues for a development plan with proper hierarchy and linking.

## Purpose

This agent takes a structured plan from the Plan Analyzer agent and creates all necessary GitHub issues in the correct order, establishing parent-child relationships and dependency links.

## Inputs

- `repo`: Repository in `owner/repo` format
- `prd_issue`: Parent PRD issue number
- `plan`: Structured plan from Plan Analyzer agent

## GitHub MCP Server Tools

This agent uses the GitHub MCP server for all GitHub operations. Available tools:

| Tool | Description |
|------|-------------|
| `mcp_github_get_repository` | Verify repository access |
| `mcp_github_get_issue` | Get issue details |
| `mcp_github_create_issue` | Create a new issue |
| `mcp_github_update_issue` | Update issue title, body, state, labels |
| `mcp_github_add_issue_comment` | Add a comment to an issue |
| `mcp_github_list_labels` | List repository labels |
| `mcp_github_create_label` | Create a new label |
| `mcp_github_add_sub_issue` | Add a sub-issue to a parent issue |
| `mcp_github_remove_sub_issue` | Remove a sub-issue from parent |
| `mcp_github_list_sub_issues` | List all sub-issues of a parent |
| `mcp_github_get_parent_issue` | Get the parent issue of a sub-issue |
| `mcp_github_reprioritize_sub_issue` | Change the order of sub-issues |

### Sub-Issue Tool Details

**mcp_github_add_sub_issue:**
```
mcp_github_add_sub_issue(
  owner: "[owner]",
  repo: "[repo]",
  issue_number: [parent_issue],    # The parent issue number
  sub_issue_id: [child_issue],     # The issue to add as sub-issue
  replace_parent: false            # Set true to re-parent if already has parent
)
```

**mcp_github_reprioritize_sub_issue:**
```
mcp_github_reprioritize_sub_issue(
  owner: "[owner]",
  repo: "[repo]",
  issue_number: [parent_issue],
  sub_issue_id: [issue_to_move],
  after_id: [issue_number]         # Place after this issue (OR use before_id)
  # OR: before_id: [issue_number]  # Place before this issue
)
```

**mcp_github_get_parent_issue:**
```
mcp_github_get_parent_issue(
  owner: "[owner]",
  repo: "[repo]",
  issue_number: [child_issue]      # Returns the parent issue if exists
)
```

### Important: Issue Number vs Node ID

When calling sub-issue tools, **always use the issue `number`, not the `id`**:

| Field | Example | Use for sub_issue_id? |
|-------|---------|----------------------|
| `number` | 7 | YES - Use this |
| `id` | 3850680229 | NO - Causes 404 |

When `mcp_github_create_issue` returns a response, extract the `number` field:
```json
{
  "id": 3850680229,      // DO NOT use this
  "number": 7,           // USE THIS for sub_issue_id
  "title": "...",
  ...
}
```

## Process

### 1. Validate Prerequisites

Before creating issues, use GitHub MCP server tools:

```
# Verify repository access
mcp_github_get_repository(owner: "[owner]", repo: "[repo]")

# Verify PRD issue exists
mcp_github_get_issue(owner: "[owner]", repo: "[repo]", issue_number: [prd_issue])

# Check if required labels exist
mcp_github_list_labels(owner: "[owner]", repo: "[repo]")
```

Create missing labels if needed:
```
mcp_github_create_label(
  owner: "[owner]",
  repo: "[repo]",
  name: "type:section",
  color: "0052CC",
  description: "Parallel work stream"
)

mcp_github_create_label(
  owner: "[owner]",
  repo: "[repo]",
  name: "type:task",
  color: "006B75",
  description: "Individual work item"
)

mcp_github_create_label(
  owner: "[owner]",
  repo: "[repo]",
  name: "complexity:S",
  color: "C2E0C6",
  description: "Small task (1-2 hours)"
)

mcp_github_create_label(
  owner: "[owner]",
  repo: "[repo]",
  name: "complexity:M",
  color: "FEF2C0",
  description: "Medium task (2-4 hours)"
)

mcp_github_create_label(
  owner: "[owner]",
  repo: "[repo]",
  name: "complexity:L",
  color: "F9D0C4",
  description: "Large task (4-8 hours)"
)
```

### 2. Create Section Issues

Create all section issues first (using `/create-section-issue` skill):

```
For each section in plan.sections:
  1. Create section issue with overview, acceptance criteria
  2. Link as sub-issue of PRD
  3. Store issue number for task creation
  4. Track section dependencies for later linking
```

### 3. Create Task Issues

Create tasks in dependency order within each section:

```
For each section:
  For each task in dependency order:
    1. Create task issue with description, criteria, complexity
    2. Link as sub-issue of section
    3. Add blocked-by references (now we have issue numbers)
    4. Store issue number for cross-references
```

Dependency order means:
- Tasks with no `blocked_by` are created first
- Tasks blocked by already-created tasks are created next
- Continue until all tasks are created

### 4. Update Section Task Lists

After all tasks are created, update each section issue with the task checklist using GitHub MCP server:

```
# Get current section body
mcp_github_get_issue(owner: "[owner]", repo: "[repo]", issue_number: [section_number])

# Update with task list
mcp_github_update_issue(
  owner: "[owner]",
  repo: "[repo]",
  issue_number: [section_number],
  body: "[updated body with task links]"
)
```

Task list format:
```markdown
## Tasks

- [ ] #104 - Design user schema (S)
- [ ] #105 - Implement user repository (M) - blocked by #104
- [ ] #106 - Add database migrations (S) - blocked by #105
```

### 5. Order Sub-Issues (Optional)

If tasks should appear in a specific order (e.g., dependency order), use reprioritize after all tasks are created:

```
# Get current sub-issue order
mcp_github_list_sub_issues(owner: "[owner]", repo: "[repo]", issue_number: [section])

# Reorder tasks to match dependency sequence
For each task that needs repositioning:
  mcp_github_reprioritize_sub_issue(
    owner: "[owner]",
    repo: "[repo]",
    issue_number: [section],
    sub_issue_id: [task_to_move],
    after_id: [preceding_task]  # or before_id: [following_task]
  )
```

This ensures sub-issues appear in logical order (blocked tasks after their blockers).

### 6. Handle Cross-Section Dependencies

For cross-section dependencies:

1. Add comment on dependent task referencing blocking task
2. Update section dependency information

```
mcp_github_add_issue_comment(
  owner: "[owner]",
  repo: "[repo]",
  issue_number: [dependent_task],
  body: "Blocked by #[blocking_task] (cross-section dependency)"
)
```

## Output Format

```yaml
result:
  status: "success" | "partial" | "failed"
  prd_issue: [number]

  sections_created:
    - number: [N]
      title: "[title]"
      url: "[url]"
      tasks_count: [N]

  tasks_created:
    - number: [N]
      title: "[title]"
      url: "[url]"
      section: [section_number]
      complexity: S|M|L
      blocked_by: [list of issue numbers]

  labels_created:
    - "[label name]"

  errors:
    - step: "[what was being done]"
      error: "[error message]"
      recovery: "[what was done to recover]"

  summary:
    total_issues: [N]
    sections: [N]
    tasks: [N]
    parallel_tracks: [N]  # Sections that can start immediately
```

## Error Handling

### Rate Limiting
```
If rate limited (HTTP 429 or rate limit error from MCP):
  1. Log current progress
  2. Wait for rate limit reset
  3. Resume from last successful creation
  4. Report partial completion if user cancels
```

### Permission Errors
```
If permission denied (HTTP 403 or authorization error):
  1. Stop immediately
  2. Report which issues were created
  3. Provide list of remaining issues for manual creation
  4. Suggest checking:
     - GITHUB_PERSONAL_ACCESS_TOKEN is set correctly
     - Token has 'repo' scope for private repos or 'public_repo' for public repos
```

### MCP Server Connection Errors
```
If MCP server connection fails:
  1. Verify connectivity to https://api.githubcopilot.com/mcp/
  2. Check GitHub authentication (OAuth or PAT)
  3. Test with: curl -I https://api.githubcopilot.com/mcp/_ping
```

### Label Creation Failures
```
If label creation fails:
  1. Continue without labels
  2. Track which labels are missing
  3. Report missing labels at end for manual creation
```

### Sub-Issue Linking Failures
```
If add_sub_issue fails:
  1. Check error type:
     - "404 Not Found": Likely using node `id` instead of issue `number`
     - "already has a parent": Use replace_parent=true if re-parenting is intended
     - "sub-issues not available": Feature may not be enabled for repository
     - "rate limited": Wait and retry (rapid content creation triggers secondary limits)
  2. Fall back to text references in issue body
  3. Add "Parent: #[number]" to issue body
  4. Note in output that manual linking may be needed

Common issues:
  - Issue already has parent: Must use replace_parent=true or remove first
  - Rate limiting: Creating content quickly triggers secondary rate limits
  - Repository feature: Sub-issues may not be available for all repositories
```

### Re-parenting Issues
```
If an issue needs to move to a different parent:
  Option 1 - Replace parent directly:
    mcp_github_add_sub_issue(
      issue_number: [new_parent],
      sub_issue_id: [issue],
      replace_parent: true
    )

  Option 2 - Remove then add:
    mcp_github_remove_sub_issue(
      issue_number: [old_parent],
      sub_issue_id: [issue]
    )
    mcp_github_add_sub_issue(
      issue_number: [new_parent],
      sub_issue_id: [issue]
    )
```

### Partial Failures
```
If any issue creation fails:
  1. Log the failure with error details from MCP response
  2. Continue with remaining issues
  3. Report partial success with:
     - What was created
     - What failed
     - How to manually complete
```

## Execution Order

1. Validate prerequisites
2. Create labels (if missing)
3. Create all section issues (can be parallel)
4. For each section, create tasks in order:
   - Tasks with no dependencies first
   - Then tasks dependent on created tasks
5. Update section issues with task lists
6. Order sub-issues within each section (optional, for display order)
7. Add cross-section dependency comments
8. Generate summary report

## Guidelines

- Always create issues in dependency order to have valid issue numbers for references
- Use MCP tools for all GitHub operations (no direct CLI commands)
- Provide progress updates during creation
- Keep detailed log of all created issues for recovery
- Never leave partial state without reporting it
- Include direct links to all created issues in final output
- Use `mcp_github_list_sub_issues` to verify sub-issue relationships after creation
