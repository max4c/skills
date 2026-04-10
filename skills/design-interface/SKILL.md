---
name: design-interface
description: Explore multiple radically different interface designs for a feature in parallel before committing to one. Uses parallel sub-agents to generate distinct approaches so you can compare tradeoffs rather than anchoring on the first idea. For UI/UX interface work specifically — not API interface design. Use when the user says "design-interface", "let's explore some designs for X", "what are the options for this screen", or before building any non-trivial UI. STATUS — v0.1 skeleton, structure complete but parallel agent orchestration and design prompt library need deep authoring.
---

# Design Interface

Generate multiple radically different design explorations for a UI feature in parallel, then compare them. The point is to *avoid* anchoring on the first idea — most designs are worse than they could be because the first sketch becomes the spec.

**This is a Level 3 skeleton.** The orchestration pattern is locked; the parallel agent prompts and the design vocabulary library need deep authoring.

Inspired by Matt Pocock's `design-an-interface`. Adapted for visual/UX design exploration rather than code API design.

---

## Contract

**Input:** a description of what you need to design + (optional) constraints (platform, style guide, accessibility requirements).

**Output:** 3-5 distinct design explorations as markdown + sketch descriptions, presented side-by-side so the user can compare. No code, no mockups — descriptions detailed enough that a designer or developer could build any of them.

**What this skill does NOT do:** write implementation code, build actual mockups in Figma, or commit to a design. It produces *explorations* for the user to evaluate. The follow-up (implementation, prototyping) is a separate workflow.

**This is for visual/UX design.** For API or type interface design, use `max:tech-spec` instead.

---

## Phases

### Phase 1: Understand the design brief

Before exploring solutions, nail down the problem.

Ask the user (one at a time, skip what's already answered):

- **What's the feature?** "A new sidebar for X" or "A way to Y"
- **Who's the user?** Power user, novice, enterprise, casual — different audiences demand different designs
- **What's the context of use?** Desktop, mobile, one-handed, quick glance, deep focus
- **What's the goal of the feature?** What should the user be able to do in 10 seconds?
- **What's the constraint?** Platform (macOS, iOS, web), existing design system, accessibility
- **What's out of scope?** What aren't we designing right now?

<!-- TODO: write a condensed intake checklist. The goal is to gather enough context to brief parallel agents, not to write a full PRD. -->

If the user already has a PRD, read it instead of asking these questions.

### Phase 2: Define the design dimensions

Before spawning explorations, identify the axes along which designs can vary. This prevents the parallel agents from generating 5 near-identical designs.

Example dimensions for a "sidebar" feature:

- **Spatial:** left rail vs. collapsible drawer vs. inline tabs vs. floating palette
- **Density:** compact (many items visible) vs. spacious (few items, large targets)
- **Navigation pattern:** hierarchical tree vs. flat list vs. search-first vs. hub-and-spoke
- **Interaction style:** click vs. hover vs. keyboard-only vs. drag
- **Visual weight:** loud (colorful, prominent) vs. quiet (muted, recedes)

Pick 2-3 dimensions where variation matters most for this feature. Each parallel exploration will occupy a different corner of that dimension space.

<!-- TODO: build a library of design dimensions for common feature types: navigation, forms, lists, editors, dashboards, modals. Reference existing design systems (Apple HIG, Material, Fluent) for vocabulary. -->

### Phase 3: Spawn parallel explorations

Dispatch 3-5 parallel sub-agents, each with a distinct brief. The dispatch happens concurrently so no exploration anchors on another.

<!-- TODO: the exact dispatch mechanism is an execution detail. Options:
     (a) Use the Agent tool with multiple concurrent calls
     (b) Write a sequential series of drafts with explicit "ignore the previous one" framing
     (c) Use a single agent with explicit instructions to generate N distinct options
     My vote: (a) if possible, (c) as fallback. Concurrent Agent calls best prevent anchoring. -->

Each brief should include:
- The feature context (from Phase 1)
- A SPECIFIC corner of the dimension space (e.g., "you're designing for the left-rail, compact, hierarchical corner — lean into those choices hard")
- Instructions: "Be opinionated. Don't hedge. Propose something that maximizes your corner's strengths and accepts its weaknesses."

The goal is that the 5 designs are *distinguishably different* — not 5 variations of the same idea.

### Phase 4: Present comparisons

Collect the outputs and present them to the user in a comparison format:

```markdown
# Design Explorations: <Feature Name>

## Option A: <Name — captures the key design choice>

**Corner:** <e.g., "Left-rail, compact, hierarchical">

**Description:** <2-3 paragraphs describing the design in enough detail to visualize it>

**Strengths:**
- <specific benefit>
- <specific benefit>

**Weaknesses:**
- <specific cost>
- <specific cost>

**Best for:** <the kind of user or use case this design excels at>

## Option B: <Name>
...

## Comparison

| Dimension | A | B | C | D | E |
|---|---|---|---|---|---|
| Speed to learn | 3 | 5 | 2 | 4 | 3 |
| Density | 5 | 2 | 4 | 3 | 3 |
| Flexibility | 2 | 4 | 5 | 3 | 4 |
| ...
```

<!-- TODO: refine the presentation format. The comparison table is key — it forces explicit tradeoffs rather than hand-waving. Include visual sketches if the skill can support them (ASCII art or markdown diagrams). -->

### Phase 5: Pick (or iterate)

Ask the user: "Which option resonates, or do you want to explore further along a specific dimension?"

Possible user responses:
- **Pick one** — done. Return the selected option as the output.
- **Combine parts** — synthesize a new design from pieces of two or more options. Present the synthesis and confirm.
- **Explore further** — pick a dimension and spawn more parallel agents along that axis.
- **Start over** — the briefs weren't right. Go back to Phase 1.

### Phase 6: Output

Write the selected (or synthesized) design to `docs/designs/<slug>.md`. Include all the explorations that were considered so future-you can see the tradeoff space.

---

## Anti-patterns

- **Generating 5 variations of the same design.** If the options differ only in color or padding, you're not exploring — you're polishing. Push the dimension variation harder.
- **Hedging.** "This design could work, but also..." is useless. Each option should be a confident commitment, even if the overall process explores multiple commitments.
- **Skipping the dimensions phase.** Without explicit dimension targets, parallel agents converge on the average. Force them apart.
- **Presenting winners and losers.** Don't rank the designs for the user. Present the tradeoffs; let the user decide which tradeoffs matter.
- **Treating design as decoration.** Design is about how the user accomplishes their goal. If your options vary in aesthetics but not in workflow, you're solving the wrong problem.

<!-- TODO: add more anti-patterns specific to design exploration. What goes wrong in design reviews? -->
