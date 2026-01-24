# /read-prd

Read and parse a PRD (Product Requirements Document) from a GitHub issue.

## Arguments

- `issue`: GitHub issue URL or number (required)
- `repo`: Repository in `owner/repo` format (required if issue is just a number)

## Instructions

### Step 1: Parse Input

Extract issue number and repository from input:

- If URL: `https://github.com/owner/repo/issues/123` → repo=`owner/repo`, issue=`123`
- If number with repo: issue=`123`, repo from argument

### Step 2: Fetch Issue Content

Use the GitHub MCP server `get_issue` tool to fetch the issue:

```
mcp_github_get_issue(
  owner: "[owner]",
  repo: "[repo]",
  issue_number: [issue-number]
)
```

This returns issue details including title, body, labels, state, and URL.

### Step 3: Parse Issue Content

Extract from the issue body:

1. **Title**: Issue title as the PRD name
2. **Overview/Summary**: Opening section describing the feature
3. **Requirements**: Bulleted lists, numbered lists, or sections with requirements
4. **User Stories**: Any "As a user..." patterns or user journey descriptions
5. **Acceptance Criteria**: Checkboxes or criteria sections
6. **Technical Notes**: Implementation hints, architecture notes
7. **Constraints**: Limitations, dependencies, deadlines mentioned
8. **Out of Scope**: Explicitly excluded items

### Step 4: Structure Output

Return structured data:

```markdown
## PRD: [Title]

**Source:** [Issue URL]
**Status:** [open/closed]
**Labels:** [label1, label2]

### Overview
[Summary extracted from issue]

### Requirements
1. [Requirement 1]
2. [Requirement 2]
...

### User Stories
- As a [role], I want to [action] so that [benefit]
...

### Acceptance Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]
...

### Technical Notes
[Any technical guidance from the issue]

### Constraints
- [Constraint 1]
- [Constraint 2]

### Out of Scope
- [Excluded item 1]

### Raw Content
<details>
<summary>Original issue body</summary>

[Full issue body for reference]

</details>
```

## Error Handling

- **Issue not found**: Return error message asking user to verify issue number and repository
- **Access denied**: Inform user that the repository may be private and suggest checking GITHUB_PERSONAL_ACCESS_TOKEN has appropriate scope
- **Empty body**: Return warning that PRD has no content, ask user to provide requirements directly
- **MCP server error**: Check connectivity to https://api.githubcopilot.com/mcp/ and GitHub authentication

## Example

Input:
```
/read-prd https://github.com/acme/project/issues/42
```

Output:
```markdown
## PRD: User Authentication System

**Source:** https://github.com/acme/project/issues/42
**Status:** open
**Labels:** enhancement, prd

### Overview
Implement a complete user authentication system with login, registration, and password reset functionality.

### Requirements
1. Users can register with email and password
2. Users can log in with credentials
3. Users can reset forgotten passwords via email
4. Sessions expire after 24 hours of inactivity

### User Stories
- As a new user, I want to create an account so that I can access the application
- As a returning user, I want to log in so that I can continue my work
- As a forgetful user, I want to reset my password so that I can regain access

### Acceptance Criteria
- [ ] Registration validates email format and password strength
- [ ] Login returns JWT token on success
- [ ] Password reset sends email within 30 seconds
- [ ] All endpoints return appropriate error messages

### Technical Notes
- Use bcrypt for password hashing
- JWT tokens should include user ID and role
- Consider rate limiting for security

### Constraints
- Must integrate with existing user database
- Email service is already configured (use existing mailer)

### Out of Scope
- Social login (OAuth) - planned for v2
- Two-factor authentication - planned for v2
```
