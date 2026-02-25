---
name: skill-forge
description: Creates new agent skills and refines existing ones following the SKILL.md open standard. Use when the user wants to create, generate, scaffold, write, improve, refine, review, or iterate on an agent skill or SKILL.md file.
---

# Skill Forge

## Determine the Mode

**Creating a new skill?** → Follow [Phase A: Create](#phase-a-create)
**Refining an existing skill?** → Follow [Phase B: Refine](#phase-b-refine)

---

## Phase A: Create

### Step 1: Gather Requirements

First, locate and read the skills knowledge base (`SKILLS-KNOWLEDGE-BASE.md` at the root of the skills repository) to ground yourself in the full best practices, anti-patterns, design patterns, and specification details. If you cannot find it, proceed using the guidance in this skill.

Then, before writing anything, determine what you can infer from the user's request and conversation context:

1. **Purpose**: What specific task or workflow should this skill handle?
2. **Scope**: Is this focused on a single task or a multi-step workflow?
3. **Target users**: Personal use or shared with a team?
4. **Trigger scenarios**: What will users say or do that should activate this skill?
5. **Domain knowledge**: What does the agent need that it wouldn't already know?
6. **Freedom level**: How prescriptive should the instructions be?
   - **High freedom** (guidelines) — multiple valid approaches exist
   - **Medium freedom** (templates/pseudocode) — preferred pattern with variation
   - **Low freedom** (exact scripts) — fragile operations, consistency critical

If you have conversation context (e.g., the user just completed a workflow manually), infer what you can from what was discussed — capture the patterns, decisions, and context they repeatedly provided.

### Step 2: Ask Clarifying Questions

After inferring what you can, ask the user targeted questions to fill in gaps. Use the AskQuestion tool (if available) or ask conversationally. This step is critical — assumptions lead to inaccurate skills.

**Always ask about these (unless already clear from context):**

1. **Scope** — What should the skill cover and what's out of scope? Offer concrete options based on what the skill could reasonably include (e.g., "Review only diffs" vs "Review full files" vs "Both").
2. **Priorities** — What aspects matter most? List the plausible focus areas for the skill's domain and let the user select. This shapes which concerns get prominent coverage vs brief mention.
3. **Placement** — Where should the skill live? (skills repo, global `~/.cursor/skills/`, or project-specific `.cursor/skills/`)

**Ask about these when relevant:**

4. **Output format** — If the skill produces output (reports, code, docs), does the user want a specific structure or is flexible format fine?
5. **Interaction style** — Should the skill be opinionated (strong defaults, push back) or neutral (present options, let the user decide)?
6. **Existing conventions** — Are there project-specific patterns, tools, or standards the skill should enforce or follow?

**Question design guidelines:**
- Tailor questions to the specific skill being created — use concrete options, not abstract choices
- Provide 2-4 options per question, phrased in terms the user would naturally understand
- Include an "all of the above" or "both" option where it makes sense
- Skip questions where the answer is obvious from context
- Batch all questions into a single prompt rather than asking one at a time

### Step 3: Design the Skill

1. **Name**: Derive a clear name following these rules:
   - Lowercase letters, numbers, hyphens only
   - Max 64 characters, must match directory name
   - No consecutive hyphens, cannot start/end with hyphen
   - Cannot contain "anthropic" or "claude"
   - Prefer descriptive names: `processing-pdfs`, `db-postgres`, `code-review`
   - Avoid vague names: `helper`, `utils`, `tools`

2. **Description**: Write a third-person description (max 1024 chars) that includes:
   - **WHAT**: Specific capabilities the skill provides
   - **WHEN**: Trigger terms and scenarios for activation
   - Example: `"Scaffolds new React components with TypeScript, tests, and styles. Use when the user wants to create, generate, or scaffold a new React component."`

3. **Structure**: Decide what files are needed:
   - `SKILL.md` — always required
   - `references/` — for detailed docs the agent reads on demand
   - `scripts/` — for executable utilities
   - `assets/` — for templates, schemas, static resources

### Step 4: Write the SKILL.md

**Select design patterns** that fit the skill's purpose. Choose from:

| Pattern | Use When |
|---------|----------|
| Template | Skill produces structured output that should follow a format |
| Examples (input/output pairs) | Output quality depends on seeing concrete demonstrations |
| Workflow (numbered steps + checklist) | Complex multi-step operations |
| Conditional workflow | Decision points branch into different paths |
| Feedback loop (validate → fix → repeat) | Quality-critical tasks that need verification |
| Dynamic context | Agent should inspect the project before acting |
| Multi-file | Templates or reference docs are too large for SKILL.md |
| Chained skills | Workflow composes multiple existing skills |

Most skills combine 2-3 patterns. Choose the minimal set that covers the skill's needs.

**Write the SKILL.md** using this structure as a starting point (include sections as relevant):

```markdown
---
name: skill-name
description: Third-person description with WHAT and WHEN.
---

# Skill Title

[Core instructions — what the agent should do when this skill activates]

## Instructions

[Numbered, sequential steps — the core workflow]

## Examples

[Concrete input/output pairs if output quality depends on demonstrations]

## Validation

[How to verify the output is correct — checklist or feedback loop]

## Error Handling

[What to do when things go wrong — cover common failure modes]
```

Not every section is needed for every skill. Omit sections that don't apply. The `## Instructions` and `## Error Handling` sections should appear in every skill.

**Consider optional frontmatter fields** when relevant:

| Field | When to Include |
|-------|-----------------|
| `license` | Skill will be shared publicly or across teams |
| `metadata` (author, version) | Skill is versioned or has multiple contributors |
| `compatibility` | Skill requires specific tools, packages, or environments |
| `allowed-tools` | Skill should be restricted to specific agent tools (experimental) |

**Authoring rules:**
- Body must stay under 500 lines — move detailed content to reference files
- Assume the agent is intelligent — only add context it doesn't already have
- Use one consistent term per concept throughout (don't mix synonyms)
- Use forward slashes in all file paths
- Provide concrete examples, not abstract descriptions
- Include numbered steps for any multi-step workflow
- Include a feedback/validation loop for quality-critical tasks
- Reference files must be one level deep from SKILL.md (no nested chains)
- For reference files over 100 lines, include a table of contents at the top

### Step 5: Create Reference Files (if needed)

Only create reference files when SKILL.md would exceed 500 lines or when content is only needed for specific subtasks.

Link from SKILL.md directly:
```markdown
## Advanced Features
- For API details, see [references/api-docs.md](references/api-docs.md)
- For examples, see [references/examples.md](references/examples.md)
```

### Step 6: Validate

Run through the quality checklist in [quality-checklist.md](quality-checklist.md). Fix any issues before considering the skill complete.

### Step 7: Place the Skill

Determine the target location based on the user's intent:

| Scope | Location |
|-------|----------|
| This skills repo | `skills/{skill-name}/SKILL.md` |
| Personal (global) | `~/.cursor/skills/{skill-name}/SKILL.md` |
| Project-specific | `{project}/.cursor/skills/{skill-name}/SKILL.md` |

If placing in this repo, use name prefixes to indicate domain:

| Prefix | Domain | Examples |
|--------|--------|----------|
| (none) | Core workflow | `skill-forge`, `code-review` |
| `ui-` | Frontend/UI | `ui-design-system` |
| `db-`, `api-`, `auth-` | Backend | `db-postgres`, `api-rest` |
| `infra-` | Infrastructure | `infra-docker`, `infra-terraform` |
| `tools-` | Integrations | `tools-git`, `tools-linear` |

Category subdirectories (e.g., `skills/core/`, `skills/backend/`) are optional — introduce them once the repo has enough skills to benefit from grouping.

### Step 8: Present Results

Summarize to the user what was created:
- Skill name and description
- File structure (which files were created and where)
- Design patterns applied
- Any decisions made or trade-offs worth noting

---

## Phase B: Refine

### Step 1: Read and Analyze

1. Locate and read the skills knowledge base (`SKILLS-KNOWLEDGE-BASE.md` at the root of the skills repository) to ground yourself in best practices and anti-patterns. If you cannot find it, proceed using the guidance in this skill.
2. Read the existing `SKILL.md` and all supporting files for the skill being refined
3. Read [quality-checklist.md](quality-checklist.md) for the full quality criteria
4. Identify issues in these categories:
   - **Spec compliance**: Does frontmatter meet all constraints?
   - **Description quality**: Is it third-person, specific, with trigger terms?
   - **Conciseness**: Are there tokens that don't justify their cost?
   - **Structure**: Is progressive disclosure used properly?
   - **Clarity**: Are instructions numbered, concrete, and unambiguous?
   - **Completeness**: Are error handling, edge cases, and validation covered?
   - **Anti-patterns**: Check against known anti-patterns (see below)

### Step 2: Check for Anti-Patterns

Flag any of these:

| Anti-Pattern | Fix |
|-------------|-----|
| Vague description ("Helps with X") | Rewrite with specific WHAT + WHEN + trigger terms |
| First/second person description | Rewrite in third person |
| SKILL.md over 500 lines | Split content into reference files |
| Deeply nested file references | Flatten to one level deep |
| Mixed terminology | Pick one term per concept |
| Too many options without a default | Provide one default with escape hatch |
| Explaining things the agent already knows | Remove — assume intelligence |
| No validation/verification steps | Add feedback loop |
| Time-sensitive content | Use "current" / "deprecated" pattern |
| Windows-style paths | Convert to forward slashes |
| Magic numbers in scripts | Add justification comments |
| Missing error handling | Add explicit error scenarios |

### Step 3: Apply Improvements

Make targeted edits. For each change, briefly explain the rationale to the user. Prioritize:

1. **Spec violations** (must fix — skill may not work otherwise)
2. **Discovery problems** (bad description = skill never activates)
3. **Token waste** (verbose content hurting context budget)
4. **Missing safety nets** (no validation, no error handling)
5. **Structural issues** (readability, organization)

### Step 4: Revalidate

After changes, run through [quality-checklist.md](quality-checklist.md) again. Confirm all items pass.

---

## Error Handling

If the target directory doesn't exist:
→ Create it and any parent directories

If a SKILL.md already exists at the target path (during create):
→ Ask the user before overwriting

If the user's desired name violates spec rules (uppercase, consecutive hyphens, reserved words, etc.):
→ Explain the constraint and propose a valid alternative

If the description exceeds 1024 characters:
→ Trim while preserving WHAT and WHEN components

If the SKILL.md body exceeds 500 lines:
→ Extract detailed content into reference files and link from SKILL.md

If the knowledge base file cannot be found:
→ Proceed using the guidance embedded in this skill — it covers the essential rules
