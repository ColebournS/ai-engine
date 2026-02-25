# AI Agent Skills: Research & Best Practices

Comprehensive research on building effective AI agent skills using the SKILL.md open standard. Covers the specification, best practices, anti-patterns, security considerations, repository architecture, and practical patterns.

---

## Table of Contents

- [1. What Are Agent Skills](#1-what-are-agent-skills)
- [2. The Open Standard (SKILL.md)](#2-the-open-standard-skillmd)
- [3. Specification Deep Dive](#3-specification-deep-dive)
- [4. Best Practices](#4-best-practices)
- [5. Anti-Patterns](#5-anti-patterns)
- [6. Design Patterns](#6-design-patterns)
- [7. Security Considerations](#7-security-considerations)
- [8. Repository Architecture](#8-repository-architecture)
- [9. Evaluation & Iteration](#9-evaluation--iteration)
- [10. Skills vs Rules vs Subagents](#10-skills-vs-rules-vs-subagents)
- [11. Sources & References](#11-sources--references)

---

## 1. What Are Agent Skills

Agent Skills are a lightweight, open format for extending AI agent capabilities with specialized knowledge and workflows. At their core, a skill is a folder containing a `SKILL.md` file with metadata and instructions that teach agents how to perform specific tasks.

### Key Properties

- **Modular**: Each skill is self-contained in its own directory
- **Portable**: Skills work across 27+ AI agents (Cursor, Claude Code, Gemini CLI, GitHub Copilot, etc.)
- **Token-Efficient**: Uses progressive disclosure — agents only load what's relevant
- **Version-Controllable**: Plain files that live in git repos
- **Composable**: Skills can reference other skills and chain workflows

### How Skills Work (Lifecycle)

Skills use a three-phase progressive disclosure model:

1. **Discovery** (~100 tokens): At startup, agents load only the `name` and `description` of each available skill — just enough to know when it might be relevant
2. **Activation** (<5000 tokens recommended): When a task matches a skill's description, the agent reads the full `SKILL.md` instructions into context
3. **Execution**: The agent follows instructions, optionally loading referenced files or executing bundled scripts as needed

This means large reference files, scripts, and assets consume zero context tokens until they're actually needed.

---

## 2. The Open Standard (SKILL.md)

The SKILL.md format is an open standard created by Anthropic (December 2025), maintained at [agentskills.io](https://agentskills.io). It is supported by all major AI coding agents.

### Minimum Viable Skill

```
my-skill/
└── SKILL.md
```

```yaml
---
name: my-skill
description: What this skill does and when to use it.
---

# My Skill

Instructions for the agent go here.
```

That's it. Two frontmatter fields and markdown instructions.

### Full Directory Structure

```
skill-name/
├── SKILL.md              # Required — main instructions
├── scripts/              # Optional — executable code
│   ├── validate.py
│   └── helper.sh
├── references/           # Optional — documentation
│   ├── api-docs.md
│   └── examples.md
└── assets/               # Optional — static resources
    ├── schema.json
    └── template.html
```

---

## 3. Specification Deep Dive

### Required Frontmatter Fields

| Field | Required | Constraints |
|-------|----------|-------------|
| `name` | Yes | Max 64 chars. Lowercase letters, numbers, hyphens only. Must match parent directory name. No consecutive hyphens. Cannot start/end with hyphen. |
| `description` | Yes | Max 1024 chars. Non-empty. Should describe both WHAT and WHEN. |

### Optional Frontmatter Fields

| Field | Constraints | Purpose |
|-------|-------------|---------|
| `license` | Short string | License name or reference to bundled LICENSE file |
| `compatibility` | Max 500 chars | Environment requirements (target product, system packages, network access) |
| `metadata` | Key-value map | Arbitrary metadata (author, version, etc.) |
| `allowed-tools` | Space-delimited list | Pre-approved tools the skill may use (experimental) |

### Name Field Rules

Valid:
```yaml
name: pdf-processing
name: data-analysis
name: code-review
```

Invalid:
```yaml
name: PDF-Processing      # uppercase not allowed
name: -pdf                 # cannot start with hyphen
name: pdf--processing      # consecutive hyphens not allowed
name: anthropic-helper     # reserved word "anthropic"
name: claude-tools         # reserved word "claude"
```

### Body Content

The markdown body after frontmatter has no structural restrictions. Write whatever helps agents perform the task effectively. Recommended sections include:

- Step-by-step instructions
- Examples of inputs and outputs
- Common edge cases
- Templates and code scaffolds

### Progressive Disclosure Rules

- Keep `SKILL.md` body under **500 lines** for optimal performance
- Use **one level deep** file references — link directly from SKILL.md to reference files
- Avoid deeply nested reference chains (agents may only partially read nested files)
- For reference files over 100 lines, include a table of contents at the top

---

## 4. Best Practices

### 4.1 Conciseness Is Critical

The context window is a shared resource. Every token in your skill competes with conversation history, other skills' metadata, and the user's request.

**Default assumption: The agent is already very smart.** Only add context the agent doesn't already have.

Challenge each piece of information:
- "Does this paragraph justify its token cost?"
- "Can I assume the agent knows this?"
- "Does the agent really need this explanation?"

**Good** (~50 tokens):
````markdown
## Extract PDF text

Use pdfplumber for text extraction:

```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

**Bad** (~150 tokens):
````markdown
## Extract PDF text

PDF (Portable Document Format) files are a common file format that contains
text, images, and other content. To extract text from a PDF, you'll need to
use a library. There are many libraries available for PDF processing, but
pdfplumber is recommended because it's easy to use...
````

### 4.2 Write Effective Descriptions

The description is the single most critical field. It determines whether the agent discovers and activates your skill from potentially 100+ available skills.

**Rules:**
1. Write in **third person** (the description is injected into the system prompt)
2. Be specific and include **trigger terms**
3. Include both **WHAT** (capabilities) and **WHEN** (trigger scenarios)

**Good examples:**
```yaml
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.

description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.

description: Review code for quality, security, and maintainability following team standards. Use when reviewing pull requests, code changes, or when the user asks for a code review.
```

**Bad examples:**
```yaml
description: Helps with documents          # too vague
description: Processes data                # too generic
description: I can help you process files  # wrong point of view
```

### 4.3 Naming Conventions

Use consistent, descriptive naming. Consider gerund form (verb + -ing) for clarity.

**Recommended patterns:**
- Gerund form: `processing-pdfs`, `analyzing-spreadsheets`, `managing-databases`
- Noun phrases: `pdf-processing`, `spreadsheet-analysis`
- Action-oriented: `process-pdfs`, `analyze-spreadsheets`
- Category-prefixed: `db-postgres`, `ui-design-system`, `infra-docker`

**Avoid:**
- Vague names: `helper`, `utils`, `tools`
- Overly generic: `documents`, `data`, `files`

### 4.4 Set Appropriate Degrees of Freedom

Match specificity to the task's fragility:

| Freedom Level | When to Use | Example |
|---------------|-------------|---------|
| **High** (text instructions) | Multiple valid approaches, context-dependent | Code review guidelines |
| **Medium** (pseudocode/templates) | Preferred pattern with acceptable variation | Report generation |
| **Low** (specific scripts) | Fragile operations, consistency critical | Database migrations |

**Analogy:** Think of the agent as exploring a path:
- **Narrow bridge with cliffs**: One safe way forward — provide exact instructions (low freedom)
- **Open field with no hazards**: Many paths to success — give general direction (high freedom)

### 4.5 Include Error Handling

Tell the agent what to do when things go wrong:

```markdown
## Error Handling

If the target directory doesn't exist:
→ Create it and any parent directories

If a file with the same name already exists:
→ Ask the user before overwriting

If required dependencies are missing:
→ Install them with the project's package manager

If the TypeScript compiler reports errors:
→ Fix the type errors before proceeding
```

### 4.6 Use Consistent Terminology

Pick one term and use it throughout the entire skill:

- **Good**: Always "extract" (not mixing "pull", "get", "retrieve")
- **Good**: Always "field" (not mixing "box", "element", "control")
- **Good**: Always "API endpoint" (not mixing "URL", "route", "path")

### 4.7 Avoid Time-Sensitive Information

Don't include information that will become outdated:

**Bad:**
```markdown
If you're doing this before August 2025, use the old API.
```

**Good:**
```markdown
## Current method
Use the v2 API endpoint.

## Old patterns (deprecated)
<details>
<summary>Legacy v1 API</summary>
The v1 API used a different endpoint. This is no longer supported.
</details>
```

### 4.8 Reference Project Paths Explicitly

Always tell the agent exactly where to find and place files:

```markdown
## File Locations

1. Components go in `src/components/{Category}/`
2. Hooks go in `src/hooks/`
3. Utilities go in `src/lib/utils/`
4. Types go in `src/types/`
5. Tests are co-located with source files
```

---

## 5. Anti-Patterns

### 5.1 Structural Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Monolithic SKILL.md (>500 lines) | Consumes too much context | Split into referenced files |
| Deeply nested references | Agent may only partially read | Keep references one level deep |
| Inconsistent formatting | Confuses the agent | Use consistent markdown structure |
| Missing frontmatter | Skill won't be discovered | Always include `name` and `description` |
| Windows-style paths (`scripts\helper.py`) | Breaks on Unix systems | Use forward slashes (`scripts/helper.py`) |

### 5.2 Content Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Vague descriptions ("Helps with documents") | Agent can't decide when to activate | Be specific with trigger terms |
| Too many options presented | Agent gets confused choosing | Provide a default with escape hatch |
| Explaining things the agent already knows | Wastes tokens | Assume intelligence, add only unique context |
| No verification criteria | Agent can't validate its output | Include validation steps |
| Ignoring edge cases | Agent fails on unusual inputs | Document common edge cases |
| Mixed terminology | Confusing instructions | Choose one term per concept |
| Time-sensitive content | Becomes incorrect | Use "old patterns" sections |
| First/second person descriptions | Discovery problems with system prompt injection | Write in third person |

### 5.3 Code/Script Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Punting errors to the agent | Unreliable behavior | Handle errors explicitly in scripts |
| Magic numbers/voodoo constants | Agent can't determine correct values | Document why each value was chosen |
| Assuming packages are installed | Script failures | List dependencies explicitly |
| No feedback loops | Errors not caught early | Add validate → fix → repeat cycles |
| Read intent vs execute intent ambiguity | Agent doesn't know what to do with scripts | Explicitly state "Run" vs "See" |

### 5.4 Architecture Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| One mega-skill that does everything | Token waste, poor discovery | Split into focused, single-purpose skills |
| Skills that conflict/overlap | Multiple skills trigger for same request | Make trigger descriptions non-overlapping |
| Duplicated instructions across skills | Maintenance burden | Extract shared content into referenced files |
| No validation or testing | Unknown effectiveness | Build evaluations before writing docs |

---

## 6. Design Patterns

### 6.1 Template Pattern

Provide output format templates with appropriate strictness:

````markdown
## Report structure

ALWAYS use this exact template:

```markdown
# [Analysis Title]

## Executive summary
[One-paragraph overview of key findings]

## Key findings
- Finding 1 with supporting data
- Finding 2 with supporting data

## Recommendations
1. Specific actionable recommendation
2. Specific actionable recommendation
```
````

### 6.2 Examples Pattern

Provide input/output pairs when output quality depends on seeing examples:

```markdown
## Commit message format

**Example 1:**
Input: Added user authentication with JWT tokens
Output:
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware

**Example 2:**
Input: Fixed bug where dates displayed incorrectly
Output:
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
```

### 6.3 Workflow Pattern

Break complex operations into sequential steps with progress tracking:

```markdown
## Migration workflow

Copy this checklist and track progress:

Task Progress:
- [ ] Step 1: Analyze current schema
- [ ] Step 2: Create migration file
- [ ] Step 3: Validate migration
- [ ] Step 4: Execute migration
- [ ] Step 5: Verify results

**Step 1: Analyze current schema**
Run: `python scripts/analyze_schema.py`
...
```

### 6.4 Conditional Workflow Pattern

Guide through decision points:

```markdown
## Document modification workflow

1. Determine the modification type:

   **Creating new content?** → Follow "Creation workflow" below
   **Editing existing content?** → Follow "Editing workflow" below

2. Creation workflow:
   - Use docx-js library
   - Build document from scratch

3. Editing workflow:
   - Unpack existing document
   - Modify XML directly
   - Validate after each change
```

### 6.5 Feedback Loop Pattern

Implement validation loops for quality-critical tasks:

```markdown
## Editing process

1. Make your edits
2. **Validate immediately**: `python scripts/validate.py output/`
3. If validation fails:
   - Review the error message
   - Fix the issues
   - Run validation again
4. **Only proceed when validation passes**
```

### 6.6 Dynamic Context Pattern

Instruct the agent to gather context before acting:

```markdown
## Instructions

Before generating any code:

1. Read `package.json` to determine:
   - Framework version
   - Available dependencies
   - Script commands

2. Scan `src/components/` for existing patterns:
   - How are props interfaces named?
   - Are default or named exports used?
   - What styling approach is used?

3. Now generate code matching these discovered patterns.
```

### 6.7 Multi-File Skill Pattern

Reference companion files for larger templates:

```
create-feature/
├── SKILL.md
├── component.template
├── test.template
└── styles.template
```

```markdown
## Instructions

1. Read the component template from `component.template`
2. Replace all placeholders:
   - `{{NAME}}` → component name (PascalCase)
   - `{{PATH}}` → file path relative to src/
3. Write the processed template to the target path
```

### 6.8 Chained Skills Pattern

Reference other skills for multi-step workflows:

```markdown
## Instructions

1. Use the "Create React Component" skill to scaffold the UI
2. Use the "Create API Route" skill to scaffold the backend
3. Use the "Write Tests" skill to generate tests for both
4. Wire up the component to call the API route
```

### 6.9 Plan-Validate-Execute Pattern

For complex or destructive operations, create a verifiable intermediate plan:

```markdown
## Workflow

1. Analyze inputs → create `changes.json` (plan)
2. Validate plan: `python scripts/validate_plan.py changes.json`
3. If validation fails, iterate on the plan
4. Execute plan: `python scripts/apply_changes.py changes.json`
5. Verify results: `python scripts/verify_output.py`
```

This catches errors early and makes complex operations reversible during the planning phase.

---

## 7. Security Considerations

### 7.1 The Threat Landscape

Recent research (SKILL-INJECT, February 2026) reveals critical vulnerabilities in the agent skills ecosystem:

- **Up to 80% attack success rate** against frontier LLM agents through skill-based prompt injection
- Agents have been shown to execute harmful instructions including data exfiltration, destructive actions, and ransomware-like behavior
- **36.82% of 3,984 publicly available community skills** contain security flaws (Snyk ToxicSkills study, February 2026)
- 76 confirmed malicious payloads found in community skills
- 13.4% rated as critical severity
- Documented threat actors distributing backdoored skills targeting SSH keys and AWS credentials

### 7.2 Attack Vectors

- **Prompt injection via skill files**: Malicious instructions hidden in otherwise legitimate skill content
- **Contextual injection**: Attacks that appear normal in isolation but become harmful in specific contexts
- **Memory poisoning**: Malicious instructions that persist across agent sessions
- **Supply chain attacks**: Backdoored skills from community repositories

### 7.3 Defense Best Practices

1. **Audit community skills before use**: Read the full SKILL.md and all scripts before installing
2. **Never blindly trust community skills**: The 36% vulnerability rate means roughly 1 in 3 have issues
3. **Review scripts thoroughly**: Executable code is the highest-risk component
4. **Prefer skills from trusted sources**: Anthropic's official repo, verified publishers
5. **Use capability-based permissions**: Leverage `allowed-tools` to limit what skills can do
6. **Pin skill versions**: Don't auto-update skills from external sources without review
7. **Validate skill provenance**: Know where skills came from and who maintains them

### 7.4 When Authoring Skills

- Never include secrets, API keys, or credentials in skill files
- Don't create skills that request elevated permissions beyond what's needed
- Include clear descriptions of what the skill will do (transparency)
- Use `allowed-tools` to declare and limit tool usage
- Write scripts that handle errors safely without exposing sensitive information

---

## 8. Repository Architecture

### 8.1 Storage Locations

| Type | Path | Scope |
|------|------|-------|
| Personal (global) | `~/.cursor/skills/` | Available across all your projects |
| Project | `.cursor/skills/` | Shared with the repository/team |
| Compatibility | `.claude/skills/` | Cross-platform with Claude Code |
| Reserved | `~/.cursor/skills-cursor/` | **DO NOT USE** — Cursor's internal built-in skills |

### 8.2 Recommended Repository Structure

For a dedicated skills repository:

```
skills-repo/
├── README.md                    # Overview and installation
├── LICENSE
├── skills/                      # All skills organized by category
│   ├── core/                    # Core workflow skills
│   │   ├── feature-build/
│   │   │   └── SKILL.md
│   │   ├── code-review/
│   │   │   ├── SKILL.md
│   │   │   ├── STANDARDS.md
│   │   │   └── examples.md
│   │   └── documentation/
│   │       └── SKILL.md
│   ├── frontend/                # UI/frontend skills
│   │   ├── ui-design-system/
│   │   ├── react-patterns/
│   │   └── state-management/
│   ├── backend/                 # Backend skills
│   │   ├── db-postgres/
│   │   ├── api-rest/
│   │   └── authentication/
│   ├── infrastructure/          # Infra/DevOps skills
│   │   ├── infra-docker/
│   │   ├── infra-terraform/
│   │   └── infra-ci-cd/
│   └── tools/                   # Tool-specific skills
│       ├── tools-git/
│       └── tools-testing/
└── template/                    # Skill template for new skills
    └── SKILL.md
```

### 8.3 Category Naming Conventions

Use consistent prefixes to organize skills by domain:

| Prefix | Domain | Examples |
|--------|--------|----------|
| `db-` | Database | `db-postgres`, `db-sqlite` |
| `ui-` | Frontend/UI | `ui-design-system`, `ui-principles` |
| `api-` | API design | `api-rest`, `api-graphql` |
| `auth-` | Authentication | `auth-jwt`, `auth-oauth` |
| `infra-` | Infrastructure | `infra-docker`, `infra-terraform` |
| `tools-` | Integrations | `tools-git`, `tools-linear` |
| `state-` | State management | `state-zustand`, `state-effector` |
| (none) | Core workflow | `feature-build`, `code-review` |

### 8.4 Installation Methods

Skills can be consumed from a repository in several ways:

1. **Cursor Settings UI** (Cursor 2.4+): Add Rule → Remote Rule (GitHub) — simplest approach
2. **Manual copy**: Clone repo, copy skill folders to `.cursor/skills/`
3. **MCP Server**: For global skill access across all projects (advanced)
4. **Git submodules**: Reference the skills repo from project repos

---

## 9. Evaluation & Iteration

### 9.1 Evaluation-Driven Development

Build evaluations BEFORE writing extensive documentation:

1. **Identify gaps**: Run the agent on representative tasks without a skill. Document specific failures
2. **Create evaluations**: Build 3+ scenarios that test these gaps
3. **Establish baseline**: Measure the agent's performance without the skill
4. **Write minimal instructions**: Just enough to address the gaps
5. **Iterate**: Execute evaluations, compare against baseline, refine

### 9.2 The Two-Claude Method

Work with one agent instance ("Claude A") to create skills tested by another instance ("Claude B"):

1. Complete a task with Claude A using normal prompting
2. Identify what context you repeatedly provided
3. Ask Claude A to create a skill capturing that pattern
4. Test the skill with Claude B on related tasks
5. Observe failures, return to Claude A to refine
6. Repeat the observe-refine-test cycle

### 9.3 What to Watch For

- **Ignored content**: If the agent never accesses a bundled file, it might be unnecessary
- **Overreliance on sections**: If the agent repeatedly reads the same file, that content might belong in the main SKILL.md
- **Missed connections**: If the agent fails to follow references, links need to be more prominent
- **Unexpected navigation**: If the agent reads files in unexpected order, the structure may need rethinking

### 9.4 Testing Across Models

Skills act as additions to models, so effectiveness depends on the underlying model:

- **Smaller/faster models** (e.g., Haiku): May need more detailed guidance
- **Balanced models** (e.g., Sonnet): Aim for clarity and efficiency
- **Powerful models** (e.g., Opus): Avoid over-explaining; trust the model

If targeting multiple models, aim for instructions that work across all of them.

### 9.5 Team Feedback Loop

1. Share skills with teammates
2. Observe their usage patterns
3. Ask: Does the skill activate when expected? Are instructions clear? What's missing?
4. Incorporate feedback to address blind spots

---

## 10. Skills vs Rules vs Subagents

Understanding when to use each mechanism:

| Feature | Rules (`.cursor/rules/`) | Skills (`.cursor/skills/`) | Subagents (`.cursor/agents/`) |
|---------|--------------------------|---------------------------|-------------------------------|
| **Purpose** | Project-wide conventions | Task-specific capabilities | Isolated specialized agents |
| **Activation** | Always-on or file-pattern match | Dynamic — agent decides based on task | Delegated by parent agent |
| **Scope** | Passive guidelines | Active instructions | Independent execution context |
| **Context Cost** | Loaded at startup (if always-on) | Loaded on demand | Separate context window |
| **Format** | `.mdc` with YAML frontmatter | `SKILL.md` with YAML frontmatter | `.md` with YAML frontmatter |
| **Best For** | Coding standards, naming conventions, formatting | Complex workflows, scaffolding, domain expertise | Code review, debugging, data analysis |

**When to use each:**

- **Rules**: "Always use TypeScript strict mode" — project-wide convention
- **Skills**: "How to create a React component in THIS project" — specialized task knowledge
- **Subagents**: "Review this PR for security issues" — isolated, focused agent

---

## 11. Sources & References

### Official Sources
- [Agent Skills Specification](https://agentskills.io/specification) — The formal SKILL.md specification
- [What Are Skills?](https://agentskills.io/what-are-skills) — Conceptual overview
- [Anthropic Skills Repository](https://github.com/anthropics/skills) — Official skill examples and reference implementations
- [Anthropic Best Practices Guide](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) — Official authoring best practices

### Community Resources
- [aussiegingersnap/cursor-skills](https://github.com/aussiegingersnap/cursor-skills) — Curated collection with category-based organization
- [Supercharging Cursor: The Ultimate Guide to Adding Agent Skills](https://skilllm.com/blog/custom-agent-skills-cursor) — SkillLM guide
- [Claude Skills Guide](https://design.dev/guides/claude-skills/) — design.dev comprehensive guide
- [Cursor Rules Best Practices](https://www.lambdacurry.dev/blog/comprehensive-cursor-rules-best-practices-guide) — Lambda Curry (applicable to skills)

### Security Research
- [SKILL-INJECT: Measuring Agent Vulnerability to Skill File Attacks](https://www.skill-inject.com/) — February 2026 research showing up to 80% attack success rate
- [Secure Skill Factory Standard](https://spec-weave.com/docs/skills/verified/secure-skill-factory-standard/) — Draft RFC for skill certification
- Snyk ToxicSkills Study (February 2026) — 36.82% of community skills contain vulnerabilities

### Cursor-Specific
- Cursor's built-in `create-skill` skill (`~/.cursor/skills-cursor/create-skill/SKILL.md`) — Canonical reference for skill creation within Cursor
- [Cursor How: Agent Skills Hub](https://www.cursorhow.com/agent-skills-hub) — Community skill collection

---

## Quick Reference: Skill Creation Checklist

### Core Quality
- [ ] `name` is lowercase, hyphens, max 64 chars, matches directory name
- [ ] `description` is specific, includes trigger terms, written in third person
- [ ] `description` includes both WHAT (capabilities) and WHEN (triggers)
- [ ] SKILL.md body is under 500 lines
- [ ] Consistent terminology throughout
- [ ] Examples are concrete, not abstract
- [ ] No time-sensitive information

### Structure
- [ ] File references are one level deep from SKILL.md
- [ ] Progressive disclosure used appropriately (large content in separate files)
- [ ] Reference files over 100 lines have a table of contents
- [ ] Workflows have clear, numbered steps
- [ ] Forward slashes used in all file paths

### Scripts (if applicable)
- [ ] Scripts handle errors explicitly (don't punt to the agent)
- [ ] Required packages documented
- [ ] No magic numbers — all constants justified
- [ ] Clear distinction between "Run" and "See" for scripts
- [ ] Feedback loops for quality-critical tasks

### Security
- [ ] No secrets, API keys, or credentials in skill files
- [ ] Scripts reviewed for security issues
- [ ] `allowed-tools` declared if using specific tools
- [ ] Community skills audited before installation

### Testing
- [ ] Tested with real usage scenarios
- [ ] Tested with multiple AI models (if targeting multiple)
- [ ] At least 3 evaluation scenarios created
- [ ] Team feedback incorporated (if applicable)
