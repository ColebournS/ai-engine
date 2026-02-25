# Agent Skills

A collection of reusable [SKILL.md](https://agentskills.io) agent skills for AI coding assistants. These skills work across Cursor, Claude Code, Gemini CLI, GitHub Copilot, and other compatible agents.

## Available Skills

| Skill | Description |
|-------|-------------|
| [code-review](skills/code-review/) | Reviews code for bugs, security vulnerabilities, performance issues, edge cases, and consistency with codebase patterns. |
| [deep-research](skills/deep-research/) | Performs comprehensive multi-source research on any topic, producing well-sourced reports with inline citations. |
| [skill-forge](skills/skill-forge/) | Creates new agent skills and refines existing ones following the SKILL.md open standard. |

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

Use the `skill-forge` skill to create new skills interactively, or refer to [SKILLS-KNOWLEDGE-BASE.md](SKILLS-KNOWLEDGE-BASE.md) for the full specification and best practices.

## License

MIT — see [LICENSE](LICENSE) for details.
