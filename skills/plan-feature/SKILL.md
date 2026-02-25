---
name: plan-feature
description: Creates implementation plans for features, tickets, and tasks by analyzing requirements, gathering codebase context, and producing structured plans with risks, dependencies, and testing strategies. Use when the user wants to plan, scope, break down, or design an approach for a feature, ticket, task, story, or piece of work before implementing it.
---

# Plan Feature

Creates structured implementation plans by analyzing requirements and codebase context. Adapts depth based on task complexity.

## Instructions

### Step 1: Understand the Request

Read the user's feature or ticket description. Extract:

- **Goal**: What should be achieved?
- **Constraints**: Any explicit requirements, deadlines, or limitations?
- **Ambiguities**: What's unclear or underspecified?

If critical information is missing (the goal is vague or the scope is unbounded), ask the user targeted questions before proceeding. Don't guess at requirements.

### Step 2: Assess Complexity

Classify the task to determine planning depth:

| Signal | Small | Medium | Large |
|--------|-------|--------|-------|
| Files changed | 1-3 | 4-10 | 10+ |
| New concepts introduced | 0 | 1-2 | 3+ |
| Cross-service impact | None | Minor | Significant |
| Data model changes | None | Additive | Breaking |
| Risk of regression | Low | Moderate | High |

- **Small**: Skip to Step 4, produce a lightweight plan (steps + key decisions only)
- **Medium/Large**: Complete all steps, produce a full plan

### Step 3: Gather Codebase Context

Before planning, understand the relevant parts of the codebase:

1. **Find related code**: Search for files, modules, and patterns related to the feature
2. **Identify conventions**: How does the codebase handle similar features? Look for:
   - Directory structure and file organization patterns
   - Existing abstractions that can be reused or extended
   - Testing patterns used for similar functionality
   - Deployment and configuration patterns
3. **Map dependencies**: What existing code will the feature interact with?
4. **Check for prior art**: Has something similar been built before? Can it be extended?

Use the explore subagent or search tools to do this efficiently. Focus on understanding patterns, not reading every file.

### Step 4: Switch to Plan Mode

Call the `SwitchMode` tool to switch to Plan mode. This presents the plan interactively for user review and approval before any implementation begins.

### Step 5: Present the Plan

Use the appropriate template based on complexity:

**Small tasks** — use the lightweight template:

```markdown
## Plan: [Feature Name]

### Approach
[1-3 sentences describing the approach]

### Steps
1. [Step with file/location]
2. [Step with file/location]
...

### Key Decisions
- [Decision point and recommended choice]
```

**Medium/Large tasks** — use the full template:

```markdown
## Plan: [Feature Name]

### Context
[Brief summary of what exists today and why this change is needed]

### Approach
[Describe the chosen approach and why it was selected over alternatives]

### Alternatives Considered
- **[Alternative A]**: [Why not chosen]
- **[Alternative B]**: [Why not chosen]

### Implementation Steps
1. **[Step title]**
   - What: [What to do]
   - Where: [Files/modules affected]
   - Why: [Rationale if non-obvious]
2. ...

### Dependencies & Ordering
- [Step X] must complete before [Step Y] because [reason]
- [External dependency] is needed for [step]

### Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| [Risk] | [H/M/L] | [H/M/L] | [How to mitigate] |

### Data Model Changes
[Describe schema changes, migrations, backward compatibility considerations — omit if none]

### Testing Strategy
- **Unit tests**: [What to unit test and key scenarios]
- **Integration tests**: [What integration points to test]
- **Manual verification**: [How to verify the feature works end-to-end]

### Rollout Considerations
[Feature flags, gradual rollout, rollback plan, monitoring — omit for internal/low-risk changes]
```

### Step 6: Iterate

After presenting the plan, wait for user feedback. Common adjustments:

- **Scope change**: Re-assess complexity, update affected sections
- **Approach disagreement**: Present the alternative in more detail, discuss trade-offs
- **Missing context**: Gather additional codebase context as needed
- **Approved**: Offer to begin implementation or create a task list

## Adaptive Depth Guidelines

Not every section is needed for every plan. Apply judgment:

| Section | Include When |
|---------|-------------|
| Alternatives Considered | Multiple viable approaches exist |
| Dependencies & Ordering | Steps have sequencing constraints |
| Risks & Mitigations | Non-trivial chance of regression or failure |
| Data Model Changes | Schema or persistent state is affected |
| Rollout Considerations | User-facing change or production risk |

For small tasks, collapsing the plan into 3-5 bullet points is perfectly fine. The goal is to surface the right information, not to fill a template.

## Error Handling

If the feature description is too vague to plan:
→ Ask the user 2-3 specific questions to narrow scope. Don't produce a plan based on assumptions.

If codebase exploration reveals the feature already exists:
→ Tell the user what you found and ask whether they want to extend, modify, or replace it.

If the feature requires tools or services you can't verify:
→ Flag the assumption in the Risks section and proceed with the plan.

If the codebase has conflicting patterns for the same concern:
→ Note the inconsistency, recommend one pattern, and explain why.

If the user rejects the plan:
→ Ask what specifically needs to change. Don't regenerate from scratch — make targeted adjustments.
