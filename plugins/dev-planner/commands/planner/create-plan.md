# /planner:create-plan

Create a structured development plan from a PRD GitHub issue, breaking it into parallel sections and sequential tasks.

## Instructions

### Step 1: Gather PRD Information

Ask the user for:

1. **PRD Issue**: GitHub issue URL or number containing the requirements
   - Example: `https://github.com/owner/repo/issues/123` or `#123`
2. **Repository**: If not clear from URL, ask for `owner/repo`

Use the `/read-prd` skill to fetch and parse the PRD content.

### Step 2: Analyze Requirements

Follow @rules/requirement-analysis.md to:

1. Extract functional and non-functional requirements
2. Identify user stories and acceptance criteria
3. List technical constraints
4. Note ambiguities and gaps

Present a summary to the user:

```markdown
## PRD Analysis

### Understanding
[Your interpretation of the requirements]

### Key Features
1. [Feature 1]
2. [Feature 2]

### Technical Constraints
- [Constraint 1]
- [Constraint 2]

### Ambiguities Found
- [Ambiguity 1]
- [Ambiguity 2]
```

### Step 3: Clarifying Questions (Iterative)

Conduct up to 5 rounds of Q&A to clarify requirements:

- Ask 3-5 focused questions per round
- Group questions by topic (scope, technical, UX, etc.)
- Summarize answers before asking more questions
- Stop when requirements are sufficiently clear

Example question format:
```markdown
## Clarifying Questions - Round 1

**Scope:**
1. [Question about boundaries]
2. [Question about MVP vs future]

**Technical:**
3. [Question about integration]
4. [Question about constraints]

**User Experience:**
5. [Question about user expectations]
```

### Step 4: Design Plan Structure

Use the **plan-analyzer** agent (@agents/plan-analyzer/AGENT.md) to autonomously:

1. **Identify Sections** (parallel work streams):
   - Group related work that can proceed independently
   - Minimize cross-section dependencies
   - Consider: Data, API, Business Logic, UI, Infrastructure

2. **Define Tasks** (sequential within sections):
   - Break each section into atomic tasks
   - Estimate complexity (S/M/L)
   - Define clear acceptance criteria
   - Identify blocking dependencies

3. **Determine Execution Order**:
   - Which sections can start immediately?
   - Which sections depend on others?
   - What shared infrastructure is needed first?

The agent follows @rules/planning-principles.md and @rules/issue-structure.md to produce a structured plan.

### Step 5: Present Plan for Approval

Show the proposed plan structure:

```markdown
## Proposed Development Plan

### Execution Overview
[Description of how work will flow]

### Shared Infrastructure (Do First)
- [ ] [Task]: [Description] (complexity)

### Section 1: [Name]
**Can start:** Immediately / After Section X
**Tasks:**
1. [ ] [Task title] (S/M/L) - [Brief description]
   - Acceptance: [Criteria]
2. [ ] [Task title] (S/M/L) - blocked by #1
   - Acceptance: [Criteria]

### Section 2: [Name]
**Can start:** Immediately (parallel with Section 1)
**Tasks:**
1. [ ] [Task title] (S/M/L)
   - Acceptance: [Criteria]

### Dependencies
- Section 2 Task 3 depends on Section 1 completion
- [Other cross-section dependencies]

### Estimated Total Tasks: [N]
### Parallel Tracks: [N sections that can run simultaneously]
```

Ask user to approve or request changes:
- "Does this plan structure look correct?"
- "Any sections or tasks missing?"
- "Should any tasks be combined or split further?"

### Step 6: Create GitHub Issues

After approval, use the **issue-creator** agent (@agents/issue-creator/AGENT.md) to autonomously create all issues:

1. **Validate Prerequisites**:
   - Verify repository access
   - Create missing labels (`type:section`, `type:task`, `complexity:*`)

2. **Create Section Issues** (using `/create-section-issue`):
   - Create all section issues first
   - Link each as sub-issue of the PRD
   - Apply `type:section` label

3. **Create Task Issues** (using `/create-task-issue`):
   - Create tasks within each section in dependency order
   - Link each as sub-issue of its section
   - Apply `type:task` and `complexity:*` labels
   - Add `blocked-by` references for sequential dependencies

4. **Update Section Task Lists**:
   - Edit each section issue to include task checklist with issue numbers

5. **Handle Cross-Section Dependencies**:
   - Add dependency comments where needed

### Step 7: Display Summary

Present the complete plan with created issues:

```markdown
## Development Plan Created

### PRD Issue
#[number] - [title]

### Sections
| Section | Issue | Status | Tasks |
|---------|-------|--------|-------|
| [Name]  | #[N]  | Ready  | [N]   |
| [Name]  | #[N]  | Blocked by #X | [N] |

### All Tasks
| Task | Section | Complexity | Blocked By |
|------|---------|------------|------------|
| #[N] [title] | [section] | S/M/L | - |
| #[N] [title] | [section] | S/M/L | #[N] |

### Execution Order
1. Start immediately: Sections [list]
2. After [section]: Sections [list]

### Quick Links
- [PRD Issue](url)
- [Section 1](url)
- [Section 2](url)
```

## Error Handling

- **Invalid issue URL**: Ask user to verify the URL format and repository access
- **Permission errors**: Inform user to check GITHUB_PERSONAL_ACCESS_TOKEN has `repo` scope (or `public_repo` for public repositories)
- **Label creation fails**: Create issues without labels and note which labels need manual creation
- **Rate limiting**: Pause and inform user, offer to resume
- **MCP server errors**: Verify connectivity to https://api.githubcopilot.com/mcp/ and GitHub authentication

## Rules

- Never skip the clarification step - ambiguous requirements lead to poor plans
- Always get user approval before creating issues
- Create issues in dependency order (sections before tasks)
- Include issue numbers in all cross-references after creation
- Keep task descriptions concise but include all acceptance criteria
