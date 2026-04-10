---
name: improve-codebase
description: Architectural review and improvement proposals. Reads the codebase for structural opportunities — shallow modules that should be deepened, leaky abstractions, misplaced complexity — and proposes specific changes with tradeoffs. Not a refactoring bot; it produces proposals, not commits. Use when the user says "improve-codebase", "architectural review", "where is this codebase weak", or before starting a big refactor. STATUS — v0.1 skeleton, structure complete but persona implementations need deep authoring.
---

# Improve Codebase

Architectural review that produces proposals, not commits. Read the codebase, identify structural improvement opportunities, and present them with tradeoffs for the user to decide. The output is a decision document, not a diff.

**This is a Level 3 skeleton.** The phase structure and persona framing are locked; the specific persona prompts and rubric content need deep authoring.

Inspired by Matt Pocock's `improve-codebase-architecture` and Ouroboros's "Nine Minds" persona system. Synthesizes shallow-module detection with multi-perspective review.

---

## Contract

**Input:** a codebase (or a directory the user points at) + optional focus area.

**Output:** a markdown proposal document with 2-5 specific architectural improvements, each with rationale, tradeoffs, and rough effort estimate. Default destination: `docs/architecture-reviews/<date>-<focus>.md`.

**What this skill does NOT do:** refactor code, open PRs, make commits, or "just clean things up". It produces a proposal document. The user decides which proposals to act on and by what mechanism.

---

## Core concept: deep vs shallow modules

From John Ousterhout's *A Philosophy of Software Design*. This is the single most load-bearing concept in this skill.

**Deep module:** small public interface, large internal implementation. A class that exposes 3 methods but internally handles 500 lines of complexity. Users of the module don't need to understand the internals. The interface is stable even as the implementation evolves.

**Shallow module:** wide public interface, thin internal implementation. A class that exposes 20 methods, each of which is a 2-line wrapper around something else. Callers have to understand the module's internals to use it correctly. Changes propagate to every caller.

**The goal of good architecture is to maximize depth.** Improve-codebase looks for shallow modules and proposes ways to deepen them, either by (a) absorbing caller logic into the module, (b) narrowing the interface, or (c) merging the module into a bigger one.

<!-- TODO: write 2-3 concrete before/after examples of deepening a shallow module. Pick examples from common frameworks (e.g., a wrapper class in a web framework that's really just a pass-through). -->

---

## Phases

### Phase 1: Orient

Before looking for problems, understand what the codebase does.

- Read the top-level entry points (main.swift, App.swift, index.ts, main.py, etc.)
- Map the module structure at a high level (what directories exist, what's in each)
- Read the README if it exists
- Ask the user: "What area should I focus on?" If they say "anywhere", pick the directory with the most churn from recent git log.

**Time budget:** 10-15 minutes. Don't read every file. Build a mental model sufficient to identify candidates.

<!-- TODO: write concrete orientation checklists for different codebase types (web app, macOS app, CLI tool, library). -->

### Phase 2: Identify candidates

Walk the codebase looking for structural weaknesses. Target categories:

**Shallow modules**
- Classes/modules with many small methods that are mostly pass-throughs
- Wrappers that add no behavior
- Utilities with broad APIs and thin internals
- "Manager" or "Helper" classes that leak their complexity to callers

**Leaky abstractions**
- Interfaces that require callers to know implementation details
- Types that expose internal representations
- Async boundaries that require callers to handle transport concerns
- Public methods that exist only for tests

**Misplaced complexity**
- Business logic in view code
- Data manipulation in controllers
- Configuration in multiple incompatible places
- State management spread across layers

**Duplication patterns**
- The same transformation implemented in three places
- Copy-pasted code with small variations (the variations usually indicate a missing parameter or a missing abstraction)
- Multiple ways to do the same thing (multiple date formatters, multiple HTTP clients)

<!-- TODO: for each category, write a concrete detection heuristic. "Here's what a leaky abstraction looks like in TypeScript: ..." -->

For each candidate, capture: file:line, what's wrong, why it matters, estimated effort to fix.

### Phase 3: Persona pass (Ouroboros-inspired)

Before writing the proposal, run the candidates past multiple mental perspectives. Each persona asks a different question.

<!-- TODO: each persona needs a real prompt. Starter versions below — flesh out with examples and specific questions. -->

**Contrarian** asks: *"What if the opposite were true?"*
- Is the "shallow module" actually correct because the interface is a boundary?
- Is the "leaky abstraction" actually fine because it's internal-only?
- Am I proposing a change because it's better, or because it's different?

**Simplifier** asks: *"What's the simplest thing that could work?"*
- Is there a proposal that's smaller and achieves 80% of the benefit?
- Can we just delete code instead of refactoring it?
- Are we inventing a new abstraction when an existing one would do?

**Architect** asks: *"If we started over, would we build it this way?"*
- Is this a symptom of a deeper structural problem?
- Does this proposal fix the real issue, or just paper over it?
- What's the 3-year maintenance implication?

**Hacker** asks: *"What constraints are actually real?"*
- Is there a pragmatic shortcut we're missing because we're over-architecting?
- What would the solution look like if we ignored the "right way"?
- Are the constraints we're working around actually constraints, or just habits?

For each candidate, run through the four personas. Drop candidates that fail the Contrarian or Simplifier checks. Strengthen candidates that survive.

<!-- TODO: decide if personas are sequential (each one sees the previous's output) or parallel (each one reviews the original candidates fresh). My vote: parallel, then merge their verdicts. Sequential risks anchoring. -->

### Phase 4: Write the proposal

For each surviving candidate, write a proposal section:

```markdown
### Proposal N: <Short title>

**What:** <1-sentence description of the change>

**Where:** <Files and directories affected>

**Why:** <What problem this solves. Include concrete pain points — "this class has been touched in 12 bug fixes over 6 months" is more compelling than "this class is shallow".>

**How:** <Sketch of the change. Not code — direction. "Merge FooWrapper into Foo and narrow the public interface to three methods: X, Y, Z.">

**Tradeoffs:**
- Pro: <specific benefit>
- Con: <specific cost>
- Risk: <what could go wrong>

**Effort:** <Small (< 1 day) | Medium (1-3 days) | Large (> 3 days)>

**Persona notes:**
- Contrarian: <what the contrarian said>
- Simplifier: <what the simplifier said>
```

Present 2-5 proposals. If you can't find that many, present fewer. If you have more than 5, rank them and present the top 5.

### Phase 5: Grill gate

Invoke `max:grill-me` in spec mode against the proposal document. The grill will push on:
- Are the "why" statements concrete, or hand-wavy?
- Do the tradeoffs actually acknowledge costs, or are they marketing?
- Is the effort estimate grounded, or optimistic?

Iterate until below the 0.2 threshold or the user overrides.

### Phase 6: Output

Write the proposal to `docs/architecture-reviews/<date>-<focus>.md`. Print the path and the number of proposals.

Do not create tickets, open PRs, or make commits. That's the user's call after reading.

---

## Anti-patterns

- **Refactoring is not improvement.** A change that moves code around without reducing complexity isn't an improvement. It's just churn.
- **Proposing changes you wouldn't do yourself.** If you wouldn't actually make this change, don't propose it. Proposals should be serious, not speculative.
- **Reviewing from memory.** You have to read the code. Proposals based on "I remember this codebase has problem X" are usually wrong.
- **Bigger is better.** A proposal that touches 50 files isn't inherently better than one that touches 5. Often it's worse. Propose the smallest change that produces the benefit.

<!-- TODO: add 2-3 more anti-patterns specific to codebase review. What do bad architectural reviews look like? -->
