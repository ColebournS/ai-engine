---
name: deep-research
description: Performs comprehensive deep research on any topic using web search, multiple sources, and perspective comparison. Produces thorough, well-sourced reports adapted to topic complexity. Use when the user wants to research, investigate, deep dive, learn about, explore, understand, or get up to speed on a topic.
---

# Deep Research

Conduct thorough, multi-source research on a topic and deliver a well-structured, cited report. This skill uses web search extensively, compares perspectives, and synthesizes findings into actionable knowledge.

**MANDATORY: Every report MUST end with a full Sources section listing every source consulted — title, URL, and a one-line relevance note. No exceptions, regardless of report format or length. Inline citations throughout the report are also required — never state a finding without attributing it.**

## Instructions

### Step 1: Clarify the Research Request

Before starting, determine:

1. **Topic**: What exactly does the user want to research? Restate it back to confirm understanding.
2. **Goal**: Are they trying to make a decision, learn a concept, evaluate options, or stay current?
3. **Constraints**: Any specific angles to focus on or exclude? Time period? Domain?

### Step 2: Ask Research Preferences

Use the AskQuestion tool (if available) or ask conversationally. Ask these two questions:

**Interaction mode:**
- **Autonomous** — Go deep, come back with a full report, minimal interruptions
- **Iterative** — Present initial findings, then let the user steer deeper into areas they care about

**Depth strategy:**
- **Breadth-first** — Cover many angles and perspectives on the topic
- **Depth-first** — Go very deep on the core topic, fewer tangents
- **Adaptive** — Start broad, then drill into the most important areas

### Step 3: Plan the Research

Decompose the topic into 4-8 research questions that collectively cover it. Think about:

- Core definition / "what is it?"
- How it works / technical details
- Current state / latest developments
- Trade-offs, pros/cons, or competing perspectives
- Practical applications / real-world usage
- Common pitfalls or misconceptions
- Comparison with alternatives (if applicable)

Share the research plan with the user before proceeding. Adjust if they want different emphasis.

### Step 4: Execute the Research

For each research question:

1. **Search broadly** — Use 2-3 varied search queries per question to avoid single-source bias. Include the current year for recency when relevant.
2. **Fetch key sources** — Read the most promising results. Prefer primary sources (official docs, papers, authoritative blogs) over aggregator content.
3. **Extract findings** — Note key facts, data points, expert opinions, and areas of consensus or disagreement.
4. **Track sources** — Record the URL and title of every source consulted.

**Search strategy tips:**
- Vary query phrasing to surface different perspectives
- Include terms like "vs", "comparison", "pros cons", "best practices" for evaluative topics
- Include the current year for topics where recency matters
- Search for counterarguments and criticisms, not just positive coverage
- If a topic has academic depth, search for papers, studies, or technical specifications

**If iterative mode:** After completing research on the first 2-3 questions, present interim findings and ask the user which areas to prioritize next.

### Step 5: Synthesize Findings

Combine research into a coherent narrative:

1. **Identify themes** — What patterns emerge across sources?
2. **Map consensus vs disagreement** — Where do sources agree? Where do they diverge, and why?
3. **Assess source quality** — Weight authoritative primary sources higher than opinion pieces
4. **Connect the dots** — Draw insights that individual sources don't make explicit
5. **Note gaps** — What couldn't be determined? What needs further investigation?

### Step 6: Deliver the Report

Adapt the format to topic complexity:

**For straightforward topics** (clear answer, limited controversy):

```markdown
# Research: [Topic]

## Summary
[2-3 sentence overview of key findings]

## Key Findings
- Finding 1 — [detail] ([source])
- Finding 2 — [detail] ([source])
- ...

## Sources
1. [Title](URL) — [one-line relevance note]
```

**For complex topics** (multiple perspectives, nuanced trade-offs):

```markdown
# Research: [Topic]

## Executive Summary
[One paragraph: what this is, why it matters, key takeaway]

## Background
[Context needed to understand the findings]

## Key Findings

### [Theme 1]
[Findings with citations]

### [Theme 2]
[Findings with citations]

## Perspectives & Trade-offs
[Where sources agree, where they disagree, and why]

## Practical Implications
[What this means in practice — actionable takeaways]

## Open Questions
[What remains unclear or needs further investigation]

## Sources
1. [Title](URL) — [one-line relevance note]
```

### Step 7: Offer to Go Deeper

After delivering the report, ask the user:
- Are there specific sections they want expanded?
- Any follow-up questions raised by the findings?
- Should any area be researched further with more targeted searches?

If the user wants to go deeper, repeat Steps 4-6 for the targeted area.

## Research Quality Standards

- **ALWAYS include a Sources section** at the end of every report — this is non-negotiable
- **Every source must include**: title, URL, and a one-line note on what it contributed
- **Inline citations required** — attribute every finding to its source within the body text, not just in the Sources section
- **Minimum 5 distinct sources** per report (more for complex topics)
- **No single-source claims** for important findings — corroborate across sources
- **Distinguish clearly** between established facts, expert opinions, and speculation
- **Acknowledge uncertainty** — say "unclear" or "contested" rather than guessing
- **Recency matters** — flag when information may be outdated and search for updates

## Error Handling

If web search returns no useful results for a query:
→ Reformulate the query with different terms and try again. If still nothing, note the gap explicitly in the report.

If sources contradict each other on a key point:
→ Present both perspectives, note the disagreement, and assess which source is more authoritative and why.

If the topic is too broad to cover meaningfully:
→ Ask the user to narrow the scope, or propose 2-3 focused sub-topics to choose from.

If the topic is too niche and sources are scarce:
→ Note the scarcity, present what's available, and suggest adjacent topics or communities where more information might exist.

If a fetched URL is paywalled or inaccessible:
→ Skip it, search for alternative sources covering the same content, and note if important sources were inaccessible.
