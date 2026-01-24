# Plan Analyzer Agent

Autonomously analyzes a PRD and generates a structured development plan with parallel sections and sequential tasks.

## Purpose

This agent takes PRD content and user clarifications as input, then produces a complete plan structure ready for issue creation. It handles the cognitive work of breaking down requirements into actionable work items.

## Inputs

- `prd_content`: Parsed PRD content from `/read-prd` skill
- `clarifications`: User answers from Q&A rounds (optional, for iterative refinement)
- `constraints`: Any technical or organizational constraints

## Process

### 1. Requirement Extraction

Parse the PRD content to identify:

- **Functional requirements**: What the system must do
- **Non-functional requirements**: Performance, security, scalability
- **User stories**: User actions and expected outcomes
- **Acceptance criteria**: How to verify completion
- **Constraints**: Technology, timeline, integration requirements

### 2. Work Decomposition

Apply @rules/planning-principles.md to break work into:

**Sections (Parallel Work Streams)**
- Group related functionality that can proceed independently
- Common patterns:
  - Data/Storage layer
  - API/Service layer
  - Business logic/Domain layer
  - UI/Presentation layer
  - Infrastructure/DevOps
  - Testing/Quality

**Tasks (Sequential Within Sections)**
- Atomic units of work (1-8 hours)
- Clear input/output boundaries
- Testable acceptance criteria

### 3. Dependency Analysis

Identify and minimize dependencies:

- **Intra-section**: Sequential tasks within a section (natural flow)
- **Cross-section**: Section A must complete before Section B can start
- **Shared infrastructure**: Common setup needed by multiple sections

Strategies to reduce dependencies:
- Define interfaces/contracts upfront
- Use mocks/stubs for development
- Parallelize with integration points

### 4. Complexity Estimation

Assign complexity to each task:

- **S (Small)**: 1-2 hours - Single function, config change, simple component
- **M (Medium)**: 2-4 hours - Feature slice, integration, moderate refactoring
- **L (Large)**: 4-8 hours - Complex feature, significant changes

If any task exceeds L, flag it for further breakdown.

### 5. Execution Order Planning

Determine optimal execution order:

1. Shared infrastructure (unblocks everything)
2. Independent sections (can start in parallel)
3. Dependent sections (wait for blockers)

## Output Format

```yaml
plan:
  summary: "[High-level description of the plan]"
  total_sections: [N]
  total_tasks: [N]
  parallel_tracks: [N]  # Sections that can run simultaneously

  shared_infrastructure:
    - title: "[Task title]"
      description: "[What needs to be done]"
      complexity: S|M|L
      acceptance_criteria:
        - "[Criterion 1]"

  sections:
    - title: "[Section: Description]"
      overview: "[Purpose of this section]"
      can_start: "immediately" | "after:[section-index]"
      depends_on: []  # Section indices
      acceptance_criteria:
        - "[Section-level criterion]"
      tasks:
        - title: "[Action verb] [specific item]"
          description: "[Detailed description]"
          complexity: S|M|L
          blocked_by: []  # Task indices within section
          acceptance_criteria:
            - "[Testable criterion]"
          technical_notes: "[Implementation hints]"

  cross_section_dependencies:
    - from_section: [index]
      from_task: [index]
      to_section: [index]
      to_task: [index]
      reason: "[Why this dependency exists]"

  risks:
    - area: "[What might go wrong]"
      mitigation: "[How to address it]"

  open_questions:
    - "[Question that needs user input]"
```

## Guidelines

- Prefer more smaller tasks over fewer larger tasks
- Every task must have at least one acceptance criterion
- Sections should be meaningful groupings, not arbitrary splits
- Flag any requirement that seems ambiguous or incomplete
- Include technical notes where implementation isn't obvious
- Identify risks early and suggest mitigations

## Example

Input PRD: "Build a user authentication system with login, registration, and password reset"

Output:
```yaml
plan:
  summary: "User authentication system with three main flows: registration, login, and password reset"
  total_sections: 3
  total_tasks: 9
  parallel_tracks: 2

  shared_infrastructure:
    - title: "Set up authentication module structure"
      description: "Create directory structure, shared types, and configuration"
      complexity: S
      acceptance_criteria:
        - "Auth module directory exists with proper structure"
        - "Shared types (User, Session, Token) are defined"

  sections:
    - title: "Data Layer: User and Session Storage"
      overview: "Database schema and repository for users and sessions"
      can_start: "immediately"
      depends_on: []
      tasks:
        - title: "Design user schema"
          complexity: S
          blocked_by: []
          acceptance_criteria:
            - "Schema includes id, email, password_hash, created_at"
            - "Email has unique constraint"
        - title: "Implement user repository"
          complexity: M
          blocked_by: [0]
          acceptance_criteria:
            - "CRUD operations work correctly"
            - "Duplicate email returns appropriate error"

    - title: "API Layer: Authentication Endpoints"
      overview: "REST endpoints for auth operations"
      can_start: "after:0"
      depends_on: [0]
      tasks:
        - title: "Create login endpoint"
          complexity: M
          acceptance_criteria:
            - "POST /auth/login accepts email/password"
            - "Returns JWT on success, 401 on failure"
        # ... more tasks
```

## Error Handling

- If PRD is too vague, return `open_questions` list for clarification
- If scope is too large (>20 tasks), suggest phasing
- If dependencies create circular references, flag and suggest resolution
