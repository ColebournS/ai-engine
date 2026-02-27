# Agent Skills

A collection of reusable [SKILL.md](https://agentskills.io) agent skills for AI coding assistants. These skills work across Cursor, Claude Code, Gemini CLI, GitHub Copilot, and other compatible agents.

## Available Skills

| Skill | Description |
|-------|-------------|
| [code-review](skills/code-review/) | Reviews code for bugs, security vulnerabilities, performance issues, edge cases, and consistency with codebase patterns. |
| [deep-research](skills/deep-research/) | Performs comprehensive multi-source research on any topic, producing well-sourced reports with inline citations. |
| [frontend-design](skills/frontend-design/) | Creates distinctive, production-grade frontend interfaces with high design quality. |
| [git-workflow](skills/git-workflow/) | Manages git commits, branches, and PR preparation by discovering project conventions from repo history. |
| [plan-feature](skills/plan-feature/) | Creates implementation plans for features and tickets by analyzing requirements, gathering codebase context, and producing structured plans with risks, dependencies, and testing strategies. |
| [skill-maker](skills/skill-maker/) | Creates new agent skills and refines existing ones following the SKILL.md open standard. |
| [systematic-debugging](skills/systematic-debugging/) | Diagnoses bugs and test failures through structured root cause analysis using a four-phase process. |
| [terminal-command](skills/terminal-command/) | Formats terminal commands for fast, safe, copy-paste-friendly execution with no placeholders, auto-waiting, and environment-aware caution. |
| [test-writing](skills/test-writing/) | Writes tests using RED-GREEN-REFACTOR, discovering project test patterns and frameworks through code search. |

## Installation

### Cursor (2.4+)

**Option A — Remote Rule (recommended):**
Add Rule → Remote Rule → paste this repository's GitHub URL.

**Option B — Manual copy:**
Clone this repo and copy the skill folders into your project or global skills directory:

```bash
# Project-specific (shared with team via git)
cp -r skills/* /path/to/your/project/.cursor/skills/

# Personal (available across all projects)
cp -r skills/* ~/.cursor/skills/
```

### Claude Code

```bash
cp -r skills/* /path/to/your/project/.claude/skills/
```

## Creating New Skills

Use the `skill-maker` skill to create new skills interactively, or refer to [SKILLS-KNOWLEDGE-BASE.md](/skills/SKILLS-KNOWLEDGE-BASE.md) for the full specification and best practices.

## License

MIT — see [LICENSE](LICENSE) for details.
