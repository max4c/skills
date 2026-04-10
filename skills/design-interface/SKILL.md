---
name: design-interface
description: Explore multiple radically different interface designs for a feature in parallel before committing to one. Uses parallel sub-agents to generate distinct approaches so you can compare tradeoffs rather than anchoring on the first idea. For UI/UX interface work specifically — not API interface design. Use when the user says "design-interface", "let's explore some designs for X", "what are the options for this screen", or before building any non-trivial UI.
---

# Design Interface

Generate multiple radically different design explorations for a UI feature in parallel, then compare them. The point is to *avoid* anchoring on the first idea — most designs are worse than they could be because the first sketch becomes the spec.

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

Before exploring solutions, nail down the problem. Gather just enough to brief parallel agents — not a full PRD.

**Intake checklist** (ask one at a time, skip what's already answered):

- **What's the feature?** "A new sidebar for X" or "A way to Y"
- **Who's the primary user?** Power user vs novice, enterprise vs consumer, one-time vs daily
- **What's the context of use?** Desktop, mobile, one-handed, quick glance, deep focus, collaborative
- **What's the user trying to accomplish?** What should they be able to do in 10 seconds? In 10 minutes?
- **What's the platform constraint?** macOS, iOS, web, cross-platform. Existing design system (HIG, Material, custom)?
- **What's the accessibility requirement?** Keyboard-only support? Screen reader? Dynamic type?
- **What's out of scope?** What aren't we designing right now?

If the user already has a PRD, read it instead of asking. Extract the intake answers from it.

### Phase 2: Define the design dimensions

Before spawning explorations, identify the axes along which designs can vary. **This is the most important step** — without explicit dimensions, parallel agents converge on the average and produce 5 variations of the same idea.

**Dimension library** (pick 2-3 where variation matters most for this feature):

**Navigation & structure**
- **Spatial arrangement:** left rail / collapsible drawer / inline tabs / floating palette / bottom bar / top bar
- **Hierarchy:** flat list / tree / hub-and-spoke / breadcrumb-driven / search-first
- **Depth:** everything visible at once / progressive disclosure / drill-down

**Density & pacing**
- **Visual density:** compact (many items, small targets) / spacious (few items, large targets)
- **Information density:** dashboard (lots of data visible) / focused (one thing at a time)
- **Interaction pacing:** rapid (hover, keyboard-heavy) / deliberate (click-confirm)

**Interaction style**
- **Primary input:** click / hover / keyboard / drag / touch / gesture
- **Modality:** modal (pauses the flow) / inline (non-blocking) / ambient (background)
- **Feedback:** immediate (optimistic) / confirmed (wait for server) / batched (queue then apply)

**Visual weight**
- **Loudness:** prominent (colorful, shadows, gradients) / quiet (muted, minimal chrome) / hidden (reveals on hover)
- **Whitespace:** generous (airy, calm) / tight (efficient, utilitarian)
- **Affordance strength:** obvious (buttons look like buttons) / subtle (discoverable but calm)

**Orientation**
- **Primary axis:** horizontal (left-to-right reading) / vertical (top-to-bottom scrolling) / radial (centered hub)
- **Grid:** rigid grid / free-form / magnetic snapping

**Temporal**
- **State transitions:** animated (transitions are communicative) / instant (no animation)
- **Persistence:** session-only / persistent across launches / cloud-synced

Pick 2-3 dimensions where variation matters most for THIS feature. Each parallel exploration will occupy a different corner of that dimension space.

**Example** — for "a new sidebar to browse workspaces":
- Dimension 1: Spatial arrangement (left rail vs. top-bar dropdown vs. command-palette-first)
- Dimension 2: Density (compact-with-icons vs. spacious-with-descriptions)
- Dimension 3: Hierarchy (flat list vs. tree with folders)

That gives up to 3×2×2 = 12 possible corners. Pick 3-5 corners that are maximally different.

### Phase 3: Spawn parallel explorations

Dispatch 3-5 parallel sub-agents, each with a distinct brief. **Concurrent dispatch prevents anchoring** — if the explorations run sequentially, later ones anchor on earlier ones.

**Dispatch mechanism:** use the Agent tool with multiple concurrent calls in a single message. Each call is a separate agent instance with a distinct brief. If concurrent Agent tool calls aren't available in the environment, fall back to a single agent with explicit instructions: "Generate N distinct designs, treating each as if it were produced by a different designer who hasn't seen the others."

**Brief template for each parallel exploration:**

```
You are designing a <feature type> for <user type> on <platform>.

Context: <1-2 sentences from Phase 1 intake>

Your assigned design corner:
- <Dimension 1>: <value — be specific>
- <Dimension 2>: <value>
- <Dimension 3>: <value>

Constraints:
- <platform>
- <accessibility>
- <any other hard constraints>

Be opinionated. Don't hedge. Propose something that MAXIMIZES your corner's
strengths and accepts its weaknesses. Do not produce a balanced or compromise
design — that's what averaging produces, and averaging is what we're trying
to avoid.

Describe the design in enough detail to visualize it:
- Overall layout (positions, proportions, key regions)
- Interaction flow (how the user accomplishes the primary task)
- Visual treatment (typography, color use, chrome, affordances)
- Key moments (first-run, empty state, error state, success state)

Do NOT:
- Write code or implementation details
- Reference specific Figma components or libraries
- Hedge with "this could also..."
- Propose multiple alternatives within your design

Output: a markdown description of ONE design, 300-600 words, structured as:
- Name (captures the key design choice)
- Overview (2-3 sentences)
- Layout
- Interaction
- Visual treatment
- Strengths (2-3 specific benefits)
- Weaknesses (2-3 specific costs — be honest)
- Best for (the user or use case this excels at)
```

The goal is that the 3-5 designs are **distinguishably different** — not 5 variations of the same idea.

### Phase 4: Present comparisons

Collect the outputs and present them to the user in a comparison format that forces explicit tradeoff visibility:

```markdown
# Design Explorations: <Feature Name>

## Brief recap
<2-3 sentences of what's being designed, for whom, on what platform.>

## Dimensions explored
- <Dimension 1>: <values we explored>
- <Dimension 2>: <values>
- <Dimension 3>: <values>

## Option A: <Name>
**Corner:** <Dimension 1 value, Dimension 2 value, ...>

**Overview:** <2-3 sentences>

**Layout:** <description>

**Interaction:** <description>

**Visual treatment:** <description>

**Strengths:**
- <specific benefit>
- <specific benefit>

**Weaknesses:**
- <specific cost>
- <specific cost>

**Best for:** <the kind of user or use case this excels at>

---

## Option B: <Name>
...

---

## Comparison matrix

| Attribute | A | B | C | D | E |
|---|---|---|---|---|---|
| Speed to learn | 3 | 5 | 2 | 4 | 3 |
| Density (1=sparse, 5=dense) | 5 | 2 | 4 | 3 | 3 |
| Flexibility | 2 | 4 | 5 | 3 | 4 |
| Accessibility | 4 | 3 | 3 | 5 | 4 |
| Feels Bugbook-y | 5 | 2 | 3 | 4 | 3 |
| Implementation effort | Low | High | Medium | Low | Medium |

(Customize the rows to match what matters for this feature. Don't use generic rubrics — pick attributes that reveal the tradeoffs.)
```

**Don't rank the designs for the user.** Present the tradeoffs; let the user decide which tradeoffs matter.

### Phase 5: Pick (or iterate)

Ask the user: **"Which option resonates, or do you want to explore further along a specific dimension?"**

Possible user responses:

- **Pick one** — done. Return the selected option as the output.
- **Combine parts** — synthesize a new design from pieces of two or more options. Present the synthesis ("Option F: take A's layout + C's interaction model") and confirm before finalizing.
- **Explore further** — pick a dimension and spawn more parallel agents along that axis. "I want to see more options with spacious density but different hierarchies."
- **Start over** — the briefs weren't right. Go back to Phase 1 and re-intake.

### Phase 6: Output

Write the selected (or synthesized) design to `docs/designs/<slug>.md`. Include all the explorations that were considered so future-you can see the tradeoff space:

```markdown
# <Feature Name> — Design

## Selected design: Option C (or synthesis of A + C)
<full description>

## Rationale
<why this one won over the others>

## Explorations considered
<summaries of A, B, C, D, E with links to full descriptions in an appendix>

## Appendix: full exploration writeups
<all 5 explorations in full>
```

Return the path to the caller.

---

## Anti-patterns

- **Generating 5 variations of the same design.** If the options differ only in color or padding, you're not exploring — you're polishing. Push the dimension variation harder. Each agent's brief should make a DIFFERENT commitment.
- **Hedging.** "This design could work, but also..." is useless. Each option should be a confident commitment, even if the overall process explores multiple commitments. Instruct agents to commit, not hedge.
- **Skipping the dimensions phase.** Without explicit dimension targets, parallel agents converge on the average. Force them apart with specific corner assignments.
- **Presenting winners and losers.** Don't rank the designs for the user. Present the tradeoffs; let them decide which tradeoffs matter.
- **Treating design as decoration.** Design is about how the user accomplishes their goal. If your options vary in aesthetics but not in workflow, you're solving the wrong problem.
- **Over-specifying the brief.** If you tell each agent exactly what to design, you're not exploring, you're prescribing. The brief should set the corner, not the solution.
- **Spawning too many agents.** 3-5 is the sweet spot. More than 5 and the comparison becomes overwhelming; fewer than 3 and you might as well just sketch one idea yourself.
- **Forgetting accessibility as a dimension.** Keyboard, screen reader, and dynamic type are dimensions too. Include at least one exploration that prioritizes accessibility over the other dimensions.

---

## Further reading

- Matt Pocock's `design-an-interface` skill — the parallel-agent framing
- Edward Tufte, *The Visual Display of Quantitative Information* — for dense-data feature design
- Don Norman, *The Design of Everyday Things* — for affordance and interaction clarity
- Apple Human Interface Guidelines / Material Design / Fluent UI — for platform-native dimension vocabulary
