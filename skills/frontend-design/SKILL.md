---
name: frontend-design
description: Creates distinctive, production-grade frontend interfaces with high design quality. Use when the user asks to build, design, style, or lay out web components, pages, UI elements, or applications. Generates creative, polished code that avoids generic AI aesthetics.
---

# Frontend Design

Create distinctive, production-grade interfaces that avoid generic "AI slop" aesthetics. Implement real working code with exceptional attention to aesthetic details and creative choices.

## Instructions

### Step 1: Understand the Context

Before coding, gather requirements from the user's request:

1. **Purpose** — What problem does this interface solve? Who uses it?
2. **Constraints** — Technical requirements (framework, performance, accessibility).
3. **Tone** — If not specified, choose a bold aesthetic direction from options like: brutally minimal, maximalist chaos, retro-futuristic, organic/natural, luxury/refined, playful/toy-like, editorial/magazine, brutalist/raw, art deco/geometric, soft/pastel, industrial/utilitarian. Use these for inspiration but design one true to the chosen direction.
4. **Differentiation** — What makes this interface unforgettable? Identify the one thing someone will remember.

### Step 2: Commit to an Aesthetic Direction

Choose a clear conceptual direction and execute it with precision. Bold maximalism and refined minimalism both work — the key is intentionality, not intensity.

Vary between light and dark themes, different fonts, different aesthetics across generations. Never converge on the same choices repeatedly.

### Step 3: Implement Working Code

Produce code (HTML/CSS/JS, React, Vue, etc.) that is:

1. Production-grade and functional
2. Visually striking and memorable
3. Cohesive with a clear aesthetic point-of-view
4. Meticulously refined in every detail

Match implementation complexity to the aesthetic vision. Maximalist designs need elaborate code with extensive animations and effects. Minimalist designs need restraint, precision, and careful attention to spacing, typography, and subtle details.

### Step 4: Apply Aesthetic Guidelines

Focus on these areas:

1. **Typography** — Choose fonts that are beautiful, unique, and interesting. Avoid generic fonts (Arial, Inter, Roboto, system fonts). Pair a distinctive display font with a refined body font. Opt for unexpected, characterful choices.
2. **Color & Theme** — Commit to a cohesive palette. Use CSS variables for consistency. Dominant colors with sharp accents outperform timid, evenly-distributed palettes.
3. **Motion** — Use animations for effects and micro-interactions. Prioritize CSS-only solutions for HTML; use Motion library for React when available. One well-orchestrated page load with staggered reveals (`animation-delay`) creates more delight than scattered micro-interactions. Use scroll-triggering and hover states that surprise.
4. **Spatial Composition** — Unexpected layouts. Asymmetry. Overlap. Diagonal flow. Grid-breaking elements. Generous negative space OR controlled density.
5. **Backgrounds & Visual Details** — Create atmosphere and depth rather than defaulting to solid colors. Apply creative forms: gradient meshes, noise textures, geometric patterns, layered transparencies, dramatic shadows, decorative borders, custom cursors, grain overlays.

### Step 5: Avoid Generic Patterns

Never produce interfaces with:
- Overused font families (Inter, Roboto, Arial, system fonts, Space Grotesk)
- Cliched color schemes (particularly purple gradients on white backgrounds)
- Predictable layouts and cookie-cutter component patterns
- Design that lacks context-specific character

Interpret creatively and make unexpected choices that feel genuinely designed for the context.

## Validation

After implementation, verify the interface against these criteria:

1. **Distinctiveness** — Could this be mistaken for a generic template? If yes, push the design further.
2. **Cohesion** — Do typography, color, motion, and layout all support the same aesthetic direction?
3. **Functionality** — Does the interface work correctly? Are interactions responsive and smooth?
4. **Detail** — Are spacing, alignment, and transitions polished? Check for rough edges.
5. **Memorability** — Is there at least one element that makes the interface unforgettable?

If any criterion fails, iterate on the design before presenting to the user.

## Error Handling

If the user doesn't specify a framework:
→ Default to HTML/CSS/JS unless the project context indicates otherwise

If the user requests a specific framework not available in the project:
→ Ask whether to add the dependency or use an alternative

If external fonts fail to load:
→ Define fallback font stacks that preserve the aesthetic intent

If animations cause performance issues:
→ Prefer CSS transforms and opacity over layout-triggering properties; use `will-change` sparingly and `prefers-reduced-motion` media queries for accessibility

If the user's requirements conflict with strong design (e.g., "make it look like every other dashboard"):
→ Propose a version that meets functional requirements while still introducing distinctive touches; explain the design rationale
