# Skill Quality Checklist

Run through every item when creating or refining a skill. Items marked **(blocking)** must pass — the skill may not function correctly otherwise.

## Spec Compliance

- [ ] **(blocking)** `name` is lowercase, hyphens and numbers only, max 64 chars
- [ ] **(blocking)** `name` matches the parent directory name
- [ ] **(blocking)** `name` has no consecutive hyphens, doesn't start/end with hyphen
- [ ] **(blocking)** `name` does not contain "anthropic" or "claude"
- [ ] **(blocking)** `description` is non-empty, max 1024 characters
- [ ] **(blocking)** YAML frontmatter has both `name` and `description`

## Description Quality

- [ ] Written in third person ("Processes X" not "I help you process X")
- [ ] Includes WHAT — specific capabilities
- [ ] Includes WHEN — trigger terms and activation scenarios
- [ ] Contains the phrases users would naturally say (trigger terms)
- [ ] Specific enough to distinguish from other skills
- [ ] Does NOT overlap with other existing skills' trigger terms

## Content Quality

- [ ] Body is under 500 lines
- [ ] Only includes context the agent doesn't already have
- [ ] Uses one consistent term per concept (no synonym mixing)
- [ ] Examples are concrete with real input/output, not abstract
- [ ] No time-sensitive information (or uses current/deprecated pattern)
- [ ] Forward slashes in all file paths
- [ ] 2-3 concrete use cases were defined before writing

## Structure

- [ ] File references are one level deep from SKILL.md
- [ ] Reference files over 100 lines have a table of contents
- [ ] Multi-step workflows use numbered steps
- [ ] Decision points use conditional workflow pattern
- [ ] Progressive disclosure: detailed content in separate files, not inline

## Safety & Robustness

- [ ] Error handling section covers common failure modes
- [ ] Quality-critical tasks have a validation/feedback loop
- [ ] No secrets, API keys, or credentials in any skill file
- [ ] Scripts (if any) handle errors explicitly, not punting to agent
- [ ] Scripts (if any) have documented dependencies
- [ ] Scripts (if any) have no magic numbers — all constants justified
- [ ] Clear "Run" vs "See" intent for every script reference
- [ ] `allowed-tools` declared if using specific tools (experimental)

## Portability

- [ ] No hardcoded absolute paths that would break in other environments
- [ ] Relative file references work from the skill's directory
- [ ] Optional frontmatter fields (`license`, `metadata`, `compatibility`) included if skill will be shared

## Testing

- [ ] **Triggering**: Skill activates correctly on representative prompts
- [ ] **Triggering**: Skill does NOT activate on unrelated prompts
- [ ] **Functional**: Output matches expectations across different request phrasings
- [ ] **Performance**: Skill improves quality/speed over no-skill baseline
- [ ] All referenced files (references/, scripts/, assets/) actually exist
