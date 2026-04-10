---
name: tech-spec
description: Turn a PRD into a technical implementation spec. Bridges the "what and why" of a PRD and the "how" an agent needs to execute. Use when the user says "tech-spec", "write a technical spec", "turn this PRD into a plan", or after /write-prd when the next question is "how do we actually build this?". STATUS — v0.1 skeleton, structurally complete but content needs filling in during next authoring session.
---

# Tech Spec

Turn a PRD (the "what and why") into a technical implementation spec (the "how"). A tech-spec answers: which modules do we build, in what order, with what interfaces, and what can go wrong along the way.

**This is a Level 3 skeleton.** Structure is locked; phase content needs deep authoring. TODO markers call out specific content decisions to make when you return.

Inspired by Ouroboros's Double Diamond execution decomposition (Discover → Define → Design → Deliver) and the classic "design doc" form used at big engineering orgs.

---

## Contract

**Input:** a PRD (as a file path, conversation content, or caller-injected context) + the codebase.

**Output:** a technical spec as markdown. Default destination: `docs/tech-specs/<slug>.md`. Callers can override.

**What this skill does NOT do:** re-litigate the PRD. If the PRD is unclear, invoke `max:grill-me` on it, or push the user back to `max:write-prd`. Tech specs build on PRDs; they don't replace them.

**Composition chain:** `caller` → `write-prd` (outputs PRD) → `tech-spec` (outputs implementation plan) → `grill-me` (stress-tests the plan) → return to caller.

---

## Phases

### Phase 1: Load the PRD

Read the PRD from wherever the caller points at — file path, conversation, or pre-loaded context. Verify every PRD section has content (Goals / Requirements / Non-Goals / Constraints / Approach / Open Questions).

If the PRD is missing sections or any section is vague, STOP. Tell the user: "This PRD needs sharpening before I can tech-spec it. Want me to invoke grill-me on the PRD first, or do you want to revise it yourself?"

<!-- TODO: decide — do we hard-stop on PRD quality, or proceed with a warning? My vote: hard-stop for Requirements/Non-Goals gaps, warn for Approach/Open Questions gaps. -->

### Phase 2: Discover (Double Diamond's first diamond — explore the problem space)

Before designing the solution, understand the constraints the PRD's Approach section gestured at.

- Read the files the Approach mentions
- Identify the blast radius: which files, modules, and cross-cutting concerns will this touch?
- Find related abstractions, shared utilities, existing patterns
- Check git log for recent work in the affected areas (prior attempts, merged work, active WIP)

<!-- TODO: write a concrete "what to look for" checklist here. Modeled on /flow's codebase exploration phase but generalized for non-Bugbook use. -->

Output of this phase: a mental model rich enough to decompose the work.

### Phase 3: Define (commit to a decomposition)

Break the work into modules with clear interfaces. This is where deep-module thinking applies.

- **Identify deep modules** — small interface, deep implementation. A good deep module hides a lot of complexity behind a simple contract.
- **Sketch the module boundaries** — what's in each module, what's the public interface, what are the invariants?
- **Identify seams** — where will tests live? A good tech spec includes the testability decision alongside the decomposition.
- **Call out existing abstractions to reuse or extend** — don't invent new patterns when the codebase already has one.

<!-- TODO: reference deep-module thinking. Matt's improve-codebase skill + John Ousterhout's "A Philosophy of Software Design". Include concrete examples of a deep vs. shallow module from a typical web or macOS codebase. -->

Present the decomposition to the user in chunks. Get approval on each module before proceeding.

### Phase 4: Design (per-module specifics)

For each module in the decomposition, spec:

- **Public interface** — functions/classes/types the rest of the code will use
- **Invariants** — what must always be true inside this module
- **Failure modes** — what can go wrong, how is it handled
- **Tests** — what behaviors should be verified (hand off to `max:tdd` for the test-writing phase)

<!-- TODO: decide on the output format per module. Options:
     (a) Markdown with interface signatures in code blocks
     (b) IDL-like prose
     (c) Just prose. 
     My vote: (a). Concrete signatures are more honest than prose. -->

### Phase 5: Deliver (sequence the work)

Order the modules for execution. Principles:

1. **Foundations first** — modules with no dependencies go first
2. **Tracer bullet through the whole stack** — prove the architecture works end-to-end before polishing
3. **Risky modules early** — if something's likely to need rework, find out early
4. **Leaf modules last** — polish and edge-cases come after the core path is proven

<!-- TODO: write a concrete example of sequencing a multi-module feature. Use a made-up example or pull one from Max's Bugbook history (e.g., "wire transcription service" as a multi-module sequence). -->

### Phase 6: Grill gate

Invoke `max:grill-me` in spec mode against the tech spec. The grill's rubric dimensions map naturally:

- **Goals** — is the module decomposition obviously correct, or are there unresolved decisions?
- **Acceptance** — does each module have concrete "done when" criteria?
- **Boundaries** — are Non-Goals preserved from the PRD? Any scope creep introduced?
- **Alternatives** — which decompositions did we consider and rule out?
- **Assumptions** — which constraints are verified vs. guessed?

Use the spec-mode threshold (0.2). If above, iterate; if below, exit.

<!-- TODO: decide if tech-spec should use a tighter or looser threshold than PRDs. My instinct: same threshold (0.2) because tech specs are what agents actually execute against. -->

### Phase 7: Output

Write the tech spec to `docs/tech-specs/<slug>.md` (or caller-specified destination). Print the path and ambiguity report.

---

## Output format

```markdown
# <Feature Name> — Technical Spec

## Context
<Link to the source PRD. Summarize in 2-3 sentences.>

## Decomposition
<List of modules with brief descriptions.>

## Modules

### <Module 1 Name>
**Responsibility:** <what this module does in one sentence>
**Public interface:**
```swift|ts|py
<signatures>
```
**Invariants:** <what must always be true>
**Failure modes:** <what can go wrong>
**Tests:** <what behaviors to verify>

### <Module 2 Name>
...

## Sequence
<Ordered list of modules to implement, with rationale for the order.>

## Open Questions
<Anything unresolved from the tech spec phase that needs user input.>

## Ambiguity Report
<Appended by grill-me at exit.>
```

<!-- TODO: refine this template. Matt's write-a-prd template has sections like "User Stories" that don't map — tech specs are for implementers, not PMs. -->

---

## Anti-patterns

- **Restating the PRD as a tech spec.** If your tech spec just paraphrases the PRD in different words, you haven't added value. Tech specs answer "how", not "what".
- **Over-decomposing into shallow modules.** 20 tiny modules with bloated interfaces is worse than 5 deep modules with clean interfaces. When in doubt, merge modules.
- **Skipping the sequencing phase.** A correct decomposition in the wrong order wastes as much time as a wrong decomposition. The sequence is load-bearing.
- **Designing without reading the code.** Tech specs that don't reference existing patterns are aspirational fiction. Read first, design second.

<!-- TODO: add 2-3 more anti-patterns specific to your experience. What goes wrong in tech specs you've seen fail? -->
