# Planning Principles

## Core Philosophy

Development plans should maximize parallel work while maintaining clear dependencies. The goal is to enable multiple developers to work simultaneously without blocking each other.

## Parallel Sections

Break work into independent sections (work streams) that can proceed in parallel:

- **Data Layer**: Database schemas, models, repositories
- **API Layer**: Endpoints, request/response handling, validation
- **Business Logic**: Core domain logic, services
- **UI Components**: Frontend components, views, interactions
- **Infrastructure**: CI/CD, deployment, monitoring

Not all projects need all sections. Choose sections based on the actual work required.

## Dependency Management

### Minimize Cross-Section Dependencies

- Identify shared interfaces early and define them first
- Use dependency inversion: depend on abstractions, not implementations
- If Section B depends on Section A, consider:
  1. Can they share an interface defined upfront?
  2. Can Section B start with mocks/stubs?
  3. Is the dependency truly necessary?

### Sequential Dependencies Within Sections

Tasks within a section often have natural dependencies:
- Schema design before repository implementation
- Repository before service layer
- Service before API endpoints

Mark these explicitly with `blocked-by` relationships.

## Task Sizing

Keep tasks small and atomic to enable effective code review:

### Size Guidelines

- **Small (S)**: 1-2 hours - Single function, simple component, config change (~50-150 lines)
- **Medium (M)**: 2-4 hours - Feature slice, integration, refactoring (~150-300 lines)
- **Large (L)**: 4-8 hours - Complex feature, significant refactoring (~300-400 lines)

If a task exceeds 8 hours or 400 lines of changes, break it down further.

### PR Size for Effective Review

Aim for **200-400 lines of changed code** per PR. Beyond 400 lines, review quality drops significantly—studies show reviewers miss more bugs as size increases.

**Each task should result in a PR that:**
- Contains **one logical change** — a single feature, bug fix, or refactor
- Takes **under 30 minutes to review** — if longer, the task is too large
- Is **self-contained** — builds and passes tests independently

### When Larger PRs Are Acceptable

- Automated refactors (renames, formatting)
- Generated code or dependency updates
- Initial project scaffolding

### Signs a Task Is Too Large

- Difficult to write a concise commit message
- Multiple unrelated files changed
- The diff requires context-switching between features
- Reviewer needs significant time to understand the changes

## Acceptance Criteria

Every task must have clear acceptance criteria:

- Specific, measurable outcomes
- Test cases or verification steps
- Definition of "done"

Bad: "Implement user authentication"
Good: "Create login endpoint that accepts email/password, validates credentials, returns JWT token. Returns 401 for invalid credentials, 400 for malformed requests."

## Shared Infrastructure First

Identify and prioritize foundational work:

1. **Project setup**: Repository, CI/CD, development environment
2. **Shared types/interfaces**: Data models, API contracts
3. **Common utilities**: Error handling, logging, validation helpers
4. **Testing infrastructure**: Test utilities, fixtures, mocks

These unblock all other sections and should be completed first.

## Risk Mitigation

- Identify technical risks early
- Create spike/prototype tasks for uncertain areas
- Front-load risky work to discover issues early
- Have fallback approaches for high-risk items
