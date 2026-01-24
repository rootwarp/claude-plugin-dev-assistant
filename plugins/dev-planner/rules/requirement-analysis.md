# Requirement Analysis

## Reading PRD Content

When analyzing a PRD, extract and categorize:

### Functional Requirements
- What the system must do
- User actions and system responses
- Data inputs and outputs
- Business rules and logic

### Non-Functional Requirements
- Performance (response times, throughput)
- Security (authentication, authorization, data protection)
- Scalability (expected load, growth)
- Reliability (uptime, error handling)
- Usability (accessibility, UX standards)

### User Stories

Identify explicit or implicit user stories:
```
As a [role]
I want to [action]
So that [benefit]
```

Extract acceptance criteria for each story.

### Constraints

- Technology stack requirements
- Integration with existing systems
- Timeline or milestone constraints
- Budget or resource limitations
- Regulatory or compliance requirements

## Clarifying Questions

Always ask clarifying questions to reduce ambiguity. Focus on:

### Scope Boundaries
- "What is explicitly out of scope?"
- "Are there related features we should NOT implement?"
- "What's the MVP vs. future enhancements?"

### Priority
- "Which features are must-have vs. nice-to-have?"
- "If we had to cut scope, what would go first?"
- "What's the critical path to launch?"

### Technical Constraints
- "Are there existing systems this must integrate with?"
- "Are there technology choices already decided?"
- "What are the performance requirements?"

### User Expectations
- "Who are the primary users?"
- "What's their technical proficiency?"
- "Are there existing workflows this replaces?"

### Edge Cases
- "What happens when [unusual scenario]?"
- "How should errors be handled?"
- "What are the data limits (max items, size, etc.)?"

## Detecting Ambiguity

Watch for these ambiguous patterns:

### Vague Quantifiers
- "fast", "responsive", "scalable" → Ask for specific numbers
- "many", "few", "some" → Ask for ranges or limits
- "easy to use" → Ask for specific UX requirements

### Undefined Terms
- Domain-specific terms without definitions
- Acronyms without expansion
- References to external systems without details

### Implicit Requirements
- "Users can log in" → What authentication method? Password rules? Session handling?
- "Display a list" → Pagination? Sorting? Filtering? Search?
- "Save the data" → Where? Validation? Audit trail?

### Missing Error Handling
- What happens on invalid input?
- What happens on system failure?
- How are users notified of problems?

### Assumed Knowledge
- References to "the usual way"
- "Similar to [other product]"
- Industry standards not explicitly stated

## Question Rounds

Conduct 3-5 rounds of clarification:

### Round 1: High-Level Understanding
- Confirm overall goals and scope
- Identify major features and user flows
- Understand success criteria

### Round 2: Feature Details
- Dive into specific feature requirements
- Clarify acceptance criteria
- Identify dependencies and integrations

### Round 3: Technical Specifics
- Confirm technology choices
- Understand performance requirements
- Identify security and compliance needs

### Round 4: Edge Cases
- Explore error scenarios
- Discuss boundary conditions
- Confirm data validation rules

### Round 5: Final Confirmation
- Summarize understanding
- Confirm priorities
- Identify any remaining unknowns

## Output Format

After analysis, produce a structured summary:

```markdown
## Requirements Summary

### Goals
[High-level objectives]

### Functional Requirements
1. [Requirement 1]
2. [Requirement 2]

### Non-Functional Requirements
- Performance: [specifics]
- Security: [specifics]

### User Stories
1. [Story 1 with acceptance criteria]
2. [Story 2 with acceptance criteria]

### Constraints
- [Constraint 1]
- [Constraint 2]

### Open Questions
- [Question 1]
- [Question 2]

### Assumptions
- [Assumption 1]
- [Assumption 2]
```
