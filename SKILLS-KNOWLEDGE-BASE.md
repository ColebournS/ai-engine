# AI Agent Skills: Research & Best Practices

Comprehensive research on building effective AI agent skills using the SKILL.md open standard. Covers the specification, best practices, anti-patterns, security considerations, ecosystem landscape, integration patterns, and practical guidance.

*Last updated: February 2026*

---

## Table of Contents

- [1. What Are Agent Skills](#1-what-are-agent-skills)
- [2. The Open Standard (SKILL.md)](#2-the-open-standard-skillmd)
- [3. Specification Deep Dive](#3-specification-deep-dive)
- [4. Best Practices](#4-best-practices)
- [5. Anti-Patterns](#5-anti-patterns)
- [6. Design Patterns](#6-design-patterns)
- [7. Skills + MCP Integration](#7-skills--mcp-integration)
- [8. Security Considerations](#8-security-considerations)
- [9. Repository Architecture](#9-repository-architecture)
- [10. Validation & Tooling](#10-validation--tooling)
- [11. Evaluation & Iteration](#11-evaluation--iteration)
- [12. Skills vs Rules vs MCP vs Subagents](#12-skills-vs-rules-vs-mcp-vs-subagents)
- [13. Ecosystem Landscape](#13-ecosystem-landscape)
- [14. Sources & References](#14-sources--references)

---

## 1. What Are Agent Skills

Agent Skills are a lightweight, open format for extending AI agent capabilities with specialized knowledge and workflows. At their core, a skill is a folder containing a `SKILL.md` file with metadata and instructions that teach agents how to perform specific tasks.

Introduced by Anthropic on October 16, 2025 and formalized as an open standard on December 18, 2025, skills are now supported by **27+ AI agents** including Cursor, Claude Code, Codex, Gemini CLI, VS Code, GitHub Copilot, and others. The official Anthropic skills repository has amassed 75,000+ stars on GitHub.

### Key Properties

- **Modular**: Each skill is self-contained in its own directory
- **Portable**: Skills work across 27+ AI agents via the open standard at [agentskills.io](https://agentskills.io)
- **Token-Efficient**: Uses progressive disclosure — agents only load what's relevant
- **Version-Controllable**: Plain files that live in git repos
- **Composable**: Skills can reference other skills, chain workflows, and orchestrate MCP tools

### How Skills Work (Lifecycle)

Skills use a three-phase progressive disclosure model:

1. **Discovery** (~50-100 tokens): At startup, agents load only the `name` and `description` of each available skill — just enough to know when it might be relevant
2. **Activation** (<5000 tokens recommended): When a task matches a skill's description, the agent reads the full `SKILL.md` instructions into context
3. **Execution**: The agent follows instructions, optionally loading referenced files or executing bundled scripts as needed

This means large reference files, scripts, and assets consume zero context tokens until they're actually needed. A developer can have 20 or 50 skills installed, but until one is triggered, each consumes only a sliver of the available token budget. Anthropic's playbook reports that well-designed skills can reduce token consumption from ~12,000 to ~6,000 for complex tasks and cut back-and-forth messages from 15 to 2.

### The Problem Skills Solve

Every LLM interaction starts from a blank slate. Skills make domain expertise persistent and reusable across sessions. Rather than re-explaining requirements and workflows every conversation, you teach the agent once and it applies that knowledge automatically when relevant.

**Declarative vs Procedural Knowledge**: Rules and context files (like `CLAUDE.md`) capture *declarative* knowledge — what and why (background context the agent should just know). Skills capture *procedural* knowledge — how (multi-step workflows with defined steps). Think of rules as an employee handbook, skills as training modules.

---

## 2. The Open Standard (SKILL.md)

The SKILL.md format is an open standard created by Anthropic (October 2025), standardized December 2025, and maintained at [agentskills.io](https://agentskills.io). It is supported by all major AI coding agents and maintained by 30+ contributors under the Apache 2.0 license.

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

### Description Field

The description is the single most critical field — it determines whether the agent discovers and activates your skill from potentially hundreds of available skills.

Good example:
```yaml
description: Extracts text and tables from PDF files, fills PDF forms, and merges multiple PDFs. Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.
```

Poor example:
```yaml
description: Helps with PDFs.
```

### Compatibility Field

Only needed if your skill has specific environment requirements:

```yaml
compatibility: Designed for Claude Code (or similar products)
```

```yaml
compatibility: Requires git, docker, jq, and access to the internet
```

### Allowed-Tools Field

Experimental. Pre-approves specific tools:

```yaml
allowed-tools: Bash(git:*) Bash(jq:*) Read
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

### Validation

Use the [skills-ref](https://github.com/agentskills/agentskills/tree/main/skills-ref) reference library to validate your skills:

```bash
skills-ref validate ./my-skill
```

This checks that your `SKILL.md` frontmatter is valid and follows all naming conventions. See [Section 10](#10-validation--tooling) for additional validation tools.

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

**Rules:**
1. Write in **third person** (the description is injected into the system prompt)
2. Be specific and include **trigger terms** — the phrases a user would actually say
3. Include both **WHAT** (capabilities) and **WHEN** (trigger scenarios)

**Good examples:**
```yaml
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.

description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.

description: Manages Linear project workflows including sprint planning, task creation, and status tracking. Use when user mentions 'sprint', 'Linear tasks', or 'project planning'.
```

**Bad examples:**
```yaml
description: Helps with documents          # too vague — will never trigger
description: Processes data                # too generic — matches everything
description: I can help you process files  # wrong point of view
```

### 4.3 Naming Conventions

Use consistent, descriptive naming. The folder name must match the `name` field and use kebab-case.

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

### 4.6 Start With Use Cases, Not Documentation

Anthropic's official playbook recommends defining 2-3 concrete use cases before writing any skill content. Each use case should define:
- **Trigger**: What the user says
- **Steps**: The sequence of actions
- **Result**: The expected output

This use-case-first approach prevents the common failure mode of building a skill that's technically complete but never triggers on real user queries.

### 4.7 Include Validation Steps

Mandate programmatic validation over vague instructions. Include validation scripts in the `scripts/` folder and instruct the agent to run them before finalizing outputs:

```markdown
## Validation

After generating output, run: `python scripts/validate.py output/`
Do NOT consider the task complete until validation passes.
```

### 4.8 Use Consistent Terminology

Pick one term and use it throughout the entire skill:

- **Good**: Always "extract" (not mixing "pull", "get", "retrieve")
- **Good**: Always "field" (not mixing "box", "element", "control")
- **Good**: Always "API endpoint" (not mixing "URL", "route", "path")

### 4.9 Avoid Time-Sensitive Information

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

### 4.10 Reference Project Paths Explicitly

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
| No trigger terms in description | Skill never activates on real queries | Include the phrases users actually say |

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
| Putting procedural knowledge in rules | Bloats always-on context | Move workflows to skills (loaded on demand) |
| Using skills for declarative context | Unnecessary activation overhead | Use rules/CLAUDE.md for background context |

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

### 6.10 Sequential Workflow Orchestration

Multi-step processes with explicit ordering, dependency management, and rollback instructions:

```markdown
## Customer onboarding workflow

Execute steps in order. Each step must validate before proceeding.

Step 1: Create account
→ Call accounts API
→ Validate: account ID returned

Step 2: Setup payment
→ Call payments API with account ID from Step 1
→ Validate: payment method confirmed

Step 3: Create subscription
→ Call subscriptions API
→ Validate: subscription active

**Rollback instructions:**
If Step 3 fails → Cancel payment method from Step 2
If Step 2 fails → Deactivate account from Step 1
```

### 6.11 Multi-MCP Coordination

Cross-service workflows with phase separation markers:

```markdown
## Design-to-development handoff

--- Phase 1: Asset Export ---
Export designs from Figma (via Figma MCP)
Validate: All assets exported successfully

--- Phase 2: Storage ---
Upload assets to Google Drive (via Drive MCP)
Validate: All files accessible via share links

--- Phase 3: Task Creation ---
Create implementation tasks in Linear (via Linear MCP)
Attach asset links to each task
Validate: All tasks created with correct labels

--- Phase 4: Notification ---
Post summary to #engineering in Slack (via Slack MCP)
```

### 6.12 Iterative Refinement Pattern

For high-stakes outputs using stopping conditions:

```markdown
## Report generation

1. Generate first draft
2. Run validation: `python scripts/validate_report.py`
3. Address each issue identified
4. Re-validate

**Stopping conditions:**
- All validation checks pass, OR
- 3 iteration cycles completed (escalate to user if still failing)
```

### 6.13 Context-Aware Tool Selection

Dynamic decisions via decision matrices:

```markdown
## File storage routing

| File type | Size | Destination |
|-----------|------|-------------|
| Code | Any | GitHub repository |
| Documents | <10MB | Notion page |
| Documents | >10MB | Google Drive |
| Media | Any | S3 bucket |
| Collaborative | Any | Notion |

Check file type and size, then route to the appropriate destination.
```

---

## 7. Skills + MCP Integration

Skills and MCP (Model Context Protocol) are complementary technologies that work together. Understanding their relationship is critical for building production-grade agent systems.

### The Mental Model

**MCP provides tools. Skills provide orchestration.**

- **MCP** = The professional kitchen (tools, ingredients, equipment)
- **Skills** = The recipes (step-by-step instructions for creating something valuable)

Or more concisely: **MCPs are verbs. Skills are playbooks.**

Without a skill, users connect an MCP but then face a blank slate. With a skill layered on top, the agent knows the team's specific methodology, the proper sequence of API calls, what validation to perform at each step, and how to handle common errors.

### Architecture

```
┌──────────────────────────────────────────────┐
│                   SKILL                       │
│            (orchestration layer)              │
│                                               │
│  ┌──────────────────────────────────────────┐ │
│  │ Bundled: scripts/ │ references/ │ assets/ │ │
│  └──────────────────────────────────────────┘ │
│                                               │
│    ┌───────┐  ┌───────┐  ┌───────┐           │
│    │  MCP  │  │ Bash  │  │ File  │           │
│    │(tools)│  │(exec) │  │ (I/O) │           │
│    └───────┘  └───────┘  └───────┘           │
└──────────────────────────────────────────────┘
```

The skill sits on top and orchestrates everything — calling MCP tools for external data, running scripts for processing, writing files for output.

### Why This Matters: Context Window Economics

MCPs consume context constantly just by being available — every tool schema is loaded into context on every message. A session can have 24%+ of the context window consumed by MCP tool definitions before the conversation even starts.

Skills use progressive disclosure, so they consume zero context until activated. This makes skills the more context-efficient choice when you don't strictly need external integrations.

### When to Use Each

| Scenario | Use |
|----------|-----|
| Agent needs capability anytime, in any context | MCP |
| Repeatable workflow with defined steps | Skill |
| External system access (Linear, Sentry, DB) | MCP |
| Orchestrating multiple tools in sequence | Skill |
| Domain expertise on top of tool access | Skill wrapping MCP |
| Background context about the project | Rules / CLAUDE.md |

### The Two Decision Questions

1. **Should the agent be able to call this capability anytime, across any context?** If yes → MCP. If only during a specific workflow → Skill with scripts.
2. **Is this a repeatable workflow with defined steps?** If yes → Skill. If it's a one-off → Just ask the agent directly.

### Four Skill Flavors

Skills tend to fall into four categories:

1. **Specialized workflows**: Multi-step procedures (TDD workflow, PR review, deployment checklist)
2. **Tool integrations**: Instructions for specific file formats or APIs (PDF processing, BigQuery)
3. **Domain expertise**: Company-specific knowledge (data model, naming conventions, rollback procedures)
4. **Knowledge retrieval**: Reference documentation loaded on demand (API specs, style guides, ADRs)

---

## 8. Security Considerations

### 8.1 The Threat Landscape

The agent skills ecosystem faces significant security challenges as of early 2026:

- **Up to 80% attack success rate** against frontier LLM agents through skill-based prompt injection (SKILL-INJECT benchmark, February 2026)
- **36.82% of 3,984 publicly available community skills** contain security flaws (Snyk ToxicSkills study, February 2026)
- **76 confirmed malicious payloads** found in community skills, with 13.4% rated as critical severity
- **26.1% of analyzed skills** contain vulnerabilities across prompt injection, data exfiltration, privilege escalation, and supply chain categories
- **Skills with executable scripts** are 2.12x more likely to contain vulnerabilities than instruction-only skills
- The **median quality score** across 11,393 tools in the ecosystem is just 44.7 out of 100
- Documented threat actors distributing backdoored skills targeting SSH keys and AWS credentials

### 8.2 Attack Vectors

- **Prompt injection via skill files**: Malicious instructions hidden in otherwise legitimate skill content
- **Contextual injection**: Attacks that appear normal in isolation but become harmful in specific contexts
- **Memory poisoning**: Malicious instructions that persist across agent sessions
- **Supply chain attacks**: Backdoored skills from community repositories
- **Trivially simple injections**: Hiding malicious instructions in long skill files and referenced scripts, or using benign approvals that carry over to harmful related actions

### 8.3 The ClawHavoc Incident (January 2026)

The largest documented agent skills supply chain attack:

- **341 malicious skills** uploaded to ClawHub marketplace between January 27-29, 2026
- **9,000+ installations compromised** within 72 hours
- Distributed **AMOS (Atomic Stealer)** malware targeting macOS and Windows
- Skills mimicked legitimate tools with professional documentation
- Included fake "Prerequisites" sections instructing users to run terminal commands that installed malware
- Harvested crypto wallet keys, browser passwords, SSH credentials, API keys, and session tokens
- Operated through at least 14 compromised or throwaway accounts
- Related vulnerability (CVE-2026-25253): Critical 1-click RCE affecting 17,500+ instances
- Post-incident analysis showed **47% of ClawHub skills had security concerns**

The incident mirrors historical supply chain attacks on NPM (2018), PyPI (2022), and VS Code Extensions (2023).

### 8.4 Defense Best Practices

1. **Audit community skills before use**: Read the full SKILL.md and all scripts before installing
2. **Never blindly trust community skills**: The 36% vulnerability rate means roughly 1 in 3 have issues
3. **Review scripts thoroughly**: Executable code is the highest-risk component (2.12x more likely to contain vulnerabilities)
4. **Prefer skills from trusted sources**: Anthropic's official repo, verified publishers
5. **Use capability-based permissions**: Leverage `allowed-tools` to limit what skills can do
6. **Pin skill versions**: Don't auto-update skills from external sources without review
7. **Validate skill provenance**: Know where skills came from and who maintains them
8. **Use validation tools**: Run security scanners like SkillCheck or skill-check before installing community skills

### 8.5 When Authoring Skills

- Never include secrets, API keys, or credentials in skill files
- Don't create skills that request elevated permissions beyond what's needed
- Include clear descriptions of what the skill will do (transparency)
- Use `allowed-tools` to declare and limit tool usage
- Write scripts that handle errors safely without exposing sensitive information

### 8.6 Emerging Governance Frameworks

**Skill Trust and Lifecycle Governance Framework**: A proposed four-tier, gate-based permission model that maps skill provenance to graduated deployment capabilities:

- **Design controls**: Risk identification and skill specification requirements
- **Runtime controls**: Dynamic authorization, anomaly detection, and interruptibility
- **Audit controls**: Cryptographic tracing and organizational accountability

**Three-tier certification model** (proposed): Scanned → Verified → Certified, with escalating rigor to establish minimum safety bars and create market pressure for secure skill authorship.

Researchers conclude that model scaling and simple input filtering won't solve agent skills security. Robust security requires context-aware authorization frameworks and capability-based permission systems with mandatory security vetting.

---

## 9. Repository Architecture

### 9.1 Storage Locations

| Type | Path | Scope |
|------|------|-------|
| Personal (global) | `~/.cursor/skills/` | Available across all your projects |
| Project | `.cursor/skills/` | Shared with the repository/team |
| Compatibility (Claude Code) | `.claude/skills/` | Cross-platform with Claude Code |
| Reserved | `~/.cursor/skills-cursor/` | **DO NOT USE** — Cursor's internal built-in skills |

Note: Cursor also reads from `.claude/skills/` folders for cross-agent compatibility.

### 9.2 Recommended Repository Structure

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

### 9.3 Category Naming Conventions

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

### 9.4 Installation Methods

Skills can be consumed from a repository in several ways:

1. **CLI installer** (recommended): Universal installers that auto-detect installed agents:
   ```bash
   npx agent-skills install vercel-labs/agent-skills/web-design-guidelines
   ```
2. **Cursor Settings UI** (Cursor 2.4+): Add Rule → Remote Rule (GitHub) — simplest for Cursor
3. **Manual copy**: Clone repo, copy skill folders to `.cursor/skills/` or `~/.cursor/skills/`
4. **Git submodules**: Reference the skills repo from project repos
5. **API** (Claude): `POST /v1/skills` endpoint for programmatic management; include skills in Messages API via `container.skills` parameter

---

## 10. Validation & Tooling

### 10.1 Official Validation

**skills-ref** (official reference library from agentskills.io):
```bash
skills-ref validate ./my-skill
```
Checks frontmatter validity, naming conventions, and spec conformance.

### 10.2 CLI Tools

**agent-skills-cli** (Rust-based):
```bash
agent-skills validate ./my-skill     # Validate against spec
agent-skills list                    # Discover skills
agent-skills read-properties         # Extract metadata as JSON/YAML/TOML
agent-skills to-prompt               # Generate XML prompt blocks
```
Available via npm, cargo, or shell installer. Supports CI/scripting with exit codes.

**skill-check** (TypeScript-based linter):
```bash
npx skill-check validate ./my-skill  # Validate structure
npx skill-check validate --fix       # Auto-fix issues
npx skill-check scaffold             # Generate new skill
```
Features: Quality scoring (0-100), multiple output formats (text, JSON, SARIF, HTML, GitHub annotations), integrated security scanning.

### 10.3 Quality Assurance Platforms

**SkillCheck** (web service): Validates skills against the open standard with tiered offerings:
- Free: YAML structure, naming conventions, semantic consistency
- Pro ($79): Security scanning, anti-slop detection, WCAG validation, token budget analysis, enterprise readiness scoring

### 10.4 Agent Evaluation Frameworks

**agent-eval**: Python-based CI evaluation harness for multi-agent environments. Detects behavioral regressions in agent instruction changes. Supports task/config matrices, Docker isolation, and code-based + LLM-based graders.

**CLIWatch**: Benchmarks CLIs against real LLM agents in CI. Posts GitHub PR comments with pass rates across models and detects agent-readiness regressions.

### 10.5 Testing Dimensions

Anthropic's playbook recommends testing skills across three dimensions:

1. **Triggering tests**: Does the skill activate for relevant queries and stay silent for unrelated ones?
2. **Functional tests**: Does the skill produce correct outputs?
3. **Performance comparisons**: How much does the skill improve over baseline (no skill)?

---

## 11. Evaluation & Iteration

### 11.1 Evaluation-Driven Development

Build evaluations BEFORE writing extensive documentation:

1. **Identify gaps**: Run the agent on representative tasks without a skill. Document specific failures
2. **Create evaluations**: Build 3+ scenarios that test these gaps
3. **Establish baseline**: Measure the agent's performance without the skill
4. **Write minimal instructions**: Just enough to address the gaps
5. **Iterate**: Execute evaluations, compare against baseline, refine

### 11.2 The Two-Claude Method

Work with one agent instance ("Claude A") to create skills tested by another instance ("Claude B"):

1. Complete a task with Claude A using normal prompting
2. Identify what context you repeatedly provided
3. Ask Claude A to create a skill capturing that pattern
4. Test the skill with Claude B on related tasks
5. Observe failures, return to Claude A to refine
6. Repeat the observe-refine-test cycle

### 11.3 Use-Case-First Development

Anthropic's recommended approach:

1. Define 2-3 concrete use cases (trigger, steps, expected result)
2. Use the built-in `skill-creator` meta-skill to generate a first draft
3. Test triggering — does it activate on the right queries?
4. Test function — does it produce correct outputs?
5. Measure improvement — token savings, fewer messages, higher quality
6. Iterate based on failures

Developers with clear use cases can build and test a functional skill in 15-30 minutes.

### 11.4 What to Watch For

- **Ignored content**: If the agent never accesses a bundled file, it might be unnecessary
- **Overreliance on sections**: If the agent repeatedly reads the same file, that content might belong in the main SKILL.md
- **Missed connections**: If the agent fails to follow references, links need to be more prominent
- **Unexpected navigation**: If the agent reads files in unexpected order, the structure may need rethinking

### 11.5 Testing Across Models

Skills act as additions to models, so effectiveness depends on the underlying model:

- **Smaller/faster models** (e.g., Haiku): May need more detailed guidance
- **Balanced models** (e.g., Sonnet): Aim for clarity and efficiency
- **Powerful models** (e.g., Opus): Avoid over-explaining; trust the model

If targeting multiple models, aim for instructions that work across all of them.

### 11.6 Team Feedback Loop

1. Share skills with teammates
2. Observe their usage patterns
3. Ask: Does the skill activate when expected? Are instructions clear? What's missing?
4. Incorporate feedback to address blind spots

---

## 12. Skills vs Rules vs MCP vs Subagents

Understanding when to use each mechanism:

| Feature | Rules | Skills | MCPs | Subagents |
|---------|-------|--------|------|-----------|
| **Purpose** | Project-wide conventions | Task-specific capabilities | External system access | Isolated specialized agents |
| **Activation** | Always-on or file-pattern match | Dynamic — agent decides based on task | Always loaded (or dynamic via MCP Tool Search) | Delegated by parent agent |
| **Scope** | Passive guidelines | Active instructions | Tools and data access | Independent execution context |
| **Context Cost** | Fixed (always loaded) | Variable (loaded on demand) | Constant per tool schema (even when unused) | Separate context window |
| **Format** | `.mdc` / `AGENTS.md` | `SKILL.md` with YAML frontmatter | Server protocol | `.md` with YAML frontmatter |
| **Best For** | Coding standards, naming conventions | Complex workflows, domain expertise | External integrations, real-time data | Parallel execution, deep research |

### The Simple Mental Model

**Rules guide. Skills do. MCPs tool. Subagents parallelize.**

### When to Use Each

**Use Rules when:**
- Standardizing coding conventions and style
- Teaching project architecture patterns
- Enforcing tone and communication preferences
- You want passive guidance across all tasks
- The information is declarative ("what" and "why")

**Use Skills when:**
- Teaching repeatable, domain-specific procedures
- Sharing team workflows (code review checklists, brand guidelines)
- Creating multi-step workflows with reference materials
- Orchestrating multiple tools in a defined sequence
- The information is procedural ("how")
- You find yourself re-explaining the same multi-step process to the agent

**Use MCPs when:**
- You need read/write access to external systems (Linear, Sentry, etc.)
- Connecting to APIs or services the agent couldn't access before
- You need real-time data from external sources
- The capability should be available anytime, in any context

**Use Subagents when:**
- Deep research across multiple codebase sections
- Breaking complex plans into parallel work streams
- Tasks that benefit from isolated context windows
- Running specialized analysis alongside other work

### Common Combinations

- **Skill + MCP**: Skill orchestrates a workflow that calls MCP tools at specific steps
- **Rule + Skill**: Rule enforces conventions; skill implements the workflow that follows them
- **Skill + Subagent**: Skill defines the workflow; subagent executes a parallelizable portion

---

## 13. Ecosystem Landscape

### 13.1 Scale (as of February 2026)

The AI agent skills ecosystem has experienced explosive growth:

| Ecosystem | Count |
|-----------|-------|
| MCP Servers | 4,133 |
| OpenClaw Skills | 2,471 |
| GPT Actions | 1,818 |
| IDE Plugins | 1,760 |
| Claude Skills | 1,211 |
| **Total tracked (SkillsIndex)** | **11,393** |

Additional registries: SkillsMP (96,000+ skills), MCP.so (17,000+ MCP servers), LobeHub.

### 13.2 Quality

Despite rapid growth, average tool quality remains low. The median quality score across all 11,393 tracked tools is **44.7 out of 100**, evaluated across security, utility, maintenance, and uniqueness dimensions.

### 13.3 Key Platforms

- **MCP (Model Context Protocol)**: De facto interoperability standard after convergence by OpenAI, Google, and Microsoft. 97 million monthly SDK downloads (970x increase in 12 months). Governance transferred to the Agentic AI Foundation under the Linux Foundation in early 2026.
- **ClawHub Skills Marketplace**: 3,000+ published skills, 15,000+ daily installations. Now requires VirusTotal scanning post-ClawHavoc.
- **Skills Registry** (skills.re): CLI tooling, semantic versioning, immutable signing, automated QA pipelines, reputation-based governance.
- **Anthropic's Official Repo** (github.com/anthropics/skills): 75,000+ stars. Production document skills plus open-source examples.

### 13.4 Key Milestones

| Date | Event |
|------|-------|
| October 16, 2025 | Anthropic introduces Agent Skills |
| December 18, 2025 | SKILL.md published as open standard; organization-wide deployment shipped |
| January 2026 | Anthropic publishes 32-page skills playbook; MCP Tool Search ships |
| January 27-29, 2026 | ClawHavoc supply chain attack (341 malicious skills, 9,000+ installs) |
| February 2026 | SKILL-INJECT benchmark published; Snyk ToxicSkills study; 27+ agents support the standard |
| Early 2026 | MCP governance transfers to Linux Foundation's Agentic AI Foundation |

---

## 14. Sources & References

### Official Sources
- [Agent Skills Specification](https://agentskills.io/specification) — The formal SKILL.md specification
- [What Are Skills?](https://agentskills.io/what-are-skills) — Conceptual overview
- [Anthropic Skills Repository](https://github.com/anthropics/skills) — Official skill examples and reference implementations (75K+ stars)
- [Anthropic Best Practices Guide](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) — Official authoring best practices
- [The Complete Guide to Building Skills for Claude](https://docs.anthropic.com) — Anthropic's 32-page playbook (January 29, 2026)
- [Agent Skills in the SDK](https://console.anthropic.com/docs/en/agent-sdk/skills) — API integration documentation
- [Best Practices for Claude Code](https://docs.anthropic.com/en/docs/claude-code/best-practices) — Claude Code agent best practices

### Community Resources
- [Supercharging Cursor: The Ultimate Guide to Adding Agent Skills](https://skilllm.com/blog/custom-agent-skills-cursor) — SkillLM guide
- [Claude Skills Guide](https://design.dev/guides/claude-skills/) — design.dev comprehensive guide
- [MCPs vs Agent Skills: Understanding the Difference](https://damiangalarza.com/posts/2026-02-05-mcps-vs-agent-skills/) — Damian Galarza's definitive comparison
- [Skills vs MCP -- When to Use Which](https://developertoolkit.ai/en/shared-workflows/skills-ecosystem/skills-vs-mcp/) — Developer Toolkit comparison
- [Cursor Rules, Skills, and Commands, Oh My!](https://www.ibuildwith.ai/blog/cursor-rules-skills-and-commands-oh-my-when-to-use-each/) — iBuildWith.ai comprehensive comparison
- [Complete Guide to Skills.md in 2026](https://www.flex.com.ph/articles/complete-guide-to-skillsmd-in-2026) — Flex overview
- [SKILL.md: The Game-Changer Giving AI Agents Procedural Memory](https://medium.com/@abhinav.dobhal/skill-md-the-game-changer-giving-ai-agents-procedural-memory-035facf1e481) — Real-world adoption analysis
- [The Complete Guide to Building Skills for Claude (Summary)](https://medium.com/@006amanraj/the-complete-guide-to-building-skills-for-claude-without-reading-33-pages-5f646ccd56b1) — Amanraj's playbook breakdown
- [Anthropic Playbook Breakdown](https://medium.com/@AdithyaGiridharan/anthropic-just-released-a-32-page-playbook-for-building-claude-skills-heres-what-you-need-to-b86fe0b123ae) — Adithya Giridharan's analysis

### Security Research
- [SKILL-INJECT: Measuring Agent Vulnerability to Skill File Attacks](https://www.skill-inject.com/) — February 2026 benchmark showing up to 80% attack success rate ([arXiv](https://arxiv.org/abs/2602.20156))
- [Agent Skills Enable a New Class of Realistic and Trivially Simple Prompt Injections](https://arxiv.org/abs/2510.26328) — October 2025 research on injection simplicity
- [Agent Skills for LLMs: Architecture, Acquisition, Security, and the Path Forward](https://arxiv.org/abs/2602.12430) — February 2026 comprehensive survey
- [Snyk ToxicSkills Study](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/) — 36.82% of community skills contain vulnerabilities (February 2026)
- [ClawHavoc: The AI Agent Supply Chain Attack](https://www.dugganusa.com/post/clawhavoc-the-ai-agent-supply-chain-attack-you-need-to-know-about) — Analysis of the January 2026 attack
- [Secure Skill Factory Standard](https://spec-weave.com/docs/skills/verified/secure-skill-factory-standard/) — Draft RFC for skill certification

### Tooling
- [skills-ref](https://github.com/agentskills/agentskills/tree/main/skills-ref) — Official reference validation library
- [skill-check](https://github.com/thedaviddias/skill-check) — TypeScript linter with quality scoring
- [agent-skills-cli](https://lib.rs/crates/agent-skills-cli) — Rust CLI for validation, listing, and prompt generation
- [SkillCheck](https://www.getskillcheck.com/) — Web-based validation with security scanning
- [agent-eval](https://github.com/fakoli/agent-eval) — CI evaluation harness for multi-agent environments
- [CLIWatch](https://cliwatch.com/) — Agent-readiness testing for CLIs

### Ecosystem
- [SkillsIndex](https://www.skillsindex.dev/blog/state-of-ai-agent-tools-february-2026/) — State of AI Agent Tools (February 2026)
- [Skills Registry](https://skills.re/) — Registry with CLI tooling and semantic versioning
- [Cursor Changelog 2.4](https://cursor.com/changelog/2-4) — Subagents, Skills, and Image Generation release
- [Cursor Agent Best Practices](https://cursor.com/blog/agent-best-practices) — Official Cursor guidance

---

## Quick Reference: Skill Creation Checklist

### Core Quality
- [ ] `name` is lowercase, hyphens, max 64 chars, matches directory name
- [ ] `description` is specific, includes trigger terms, written in third person
- [ ] `description` includes both WHAT (capabilities) and WHEN (triggers)
- [ ] `description` includes phrases users would actually say
- [ ] SKILL.md body is under 500 lines
- [ ] Consistent terminology throughout
- [ ] Examples are concrete, not abstract
- [ ] No time-sensitive information
- [ ] 2-3 concrete use cases defined before writing

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
- [ ] Validation scripts included and referenced in instructions

### Security
- [ ] No secrets, API keys, or credentials in skill files
- [ ] Scripts reviewed for security issues
- [ ] `allowed-tools` declared if using specific tools
- [ ] Community skills audited before installation
- [ ] Validation tool run against skill (skill-check, skills-ref, or SkillCheck)

### Testing
- [ ] Triggering tests: skill activates for relevant queries, stays silent for unrelated ones
- [ ] Functional tests: skill produces correct outputs
- [ ] Performance comparison: measured improvement over baseline
- [ ] Tested with multiple AI models (if targeting multiple)
- [ ] At least 3 evaluation scenarios created
- [ ] Team feedback incorporated (if applicable)
