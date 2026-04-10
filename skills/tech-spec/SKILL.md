---
name: tech-spec
description: Turn a PRD into a technical implementation spec. Bridges the "what and why" of a PRD and the "how" an agent needs to execute. Module decomposition with deep-module emphasis, sequencing by tracer-bullet principles, and a grill gate before exit. Use when the user says "tech-spec", "write a technical spec", "turn this PRD into a plan", or after /write-prd when the next question is "how do we actually build this?".
---

# Tech Spec

Turn a PRD (the "what and why") into a technical implementation spec (the "how"). A tech-spec answers: which modules do we build, in what order, with what interfaces, and what can go wrong along the way.

Inspired by Ouroboros's Double Diamond execution decomposition (Discover → Define → Design → Deliver), John Ousterhout's *A Philosophy of Software Design* (deep modules), and the classic "design doc" form used at big engineering orgs.

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

**Hard-stop if these are missing:**
- **Requirements** — no concrete behaviors to decompose into modules
- **Non-Goals** — risk of silent scope creep during design

If either is missing, STOP and tell the user: "This PRD needs sharpening before I can tech-spec it. Want me to invoke `max:grill-me` on the PRD first, or do you want to revise it yourself?"

**Warn but proceed if these are weak:**
- **Approach** — direction is helpful but tech-spec can propose one
- **Open Questions** — unresolved items become explicit inputs to the tech spec phase

### Phase 2: Discover (explore the problem space)

Before designing the solution, understand the constraints the PRD's Approach section gestured at.

**What to look for:**
- **Files the Approach mentions** — read each one fully, don't skim
- **Sibling code** — the directory the Approach points at, the module it lives in
- **Shared abstractions** — utilities, helpers, types that would be reused or extended
- **Existing patterns** — how similar features are implemented in this codebase (not how they'd be implemented in general)
- **Recent history** — git log on the affected files for the last 30-60 days. Prior attempts, in-flight work, reverted commits all matter.
- **Test infrastructure** — what test patterns exist? Unit? Integration? Snapshot? This shapes the Design phase.
- **Cross-cutting concerns** — auth, logging, error handling, observability. Is there a convention?

**Output of this phase:** a mental model rich enough to decompose the work. You should be able to answer: "Given this codebase, what shape does the solution naturally want to take?"

**Time budget:** 10-20 minutes. If you find yourself reading beyond that, the PRD scope is too big — push back and ask for a slice.

### Phase 3: Define (commit to a decomposition)

Break the work into modules with clear interfaces. This is where deep-module thinking applies.

**Deep module test** (Ousterhout):
A module is *deep* if its public interface is much smaller than the implementation it encapsulates. A class that exposes 3 methods but internally handles 500 lines of complexity is deep. A class that exposes 20 methods that are each 2-line wrappers around something else is *shallow*.

**Good deep modules:**
- `TranscriptParser` — public: `parse(text) -> Transcript`. Internal: tokenization, speaker detection, timestamp normalization, punctuation recovery.
- `CacheEvictionPolicy` — public: `shouldEvict(entry, now) -> Bool`. Internal: LRU tracking, priority weighting, size accounting.
- `ExportManager` — public: `export(document, format) -> URL`. Internal: format-specific renderers, asset collection, zip packaging.

**Bad shallow modules:**
- `TextUtils` with 15 static methods each wrapping one line of string manipulation
- `DatabaseHelper` whose methods all take a SQL string and pass it to the underlying client
- `EventMiddleware` that exists only to re-emit events on a different name

**Decomposition steps:**

1. **List the behaviors from the PRD Requirements.** Each behavior is a candidate for a module or a method on a module.
2. **Cluster behaviors that share state or invariants.** Behaviors that touch the same data belong in the same module.
3. **Identify seams.** A seam is where test boundaries live. Each module should have exactly one public seam where tests can reach it. Multiple seams = shallow module.
4. **Name each module** with a noun that describes its responsibility, not its implementation. `TranscriptParser` good, `StringProcessor` bad.
5. **Sketch the public interface** for each module. 1-5 methods is the sweet spot. 10+ methods is a smell — the module is probably two modules pretending to be one.
6. **Call out reused abstractions.** If the decomposition is inventing a new pattern when the codebase already has one, merge them.

Present the decomposition to the user in chunks (2-3 modules at a time). Get approval on each module before proceeding to the next.

### Phase 4: Design (per-module specifics)

For each module in the decomposition, spec:

```markdown
### <Module Name>

**Responsibility:** <what this module does in one sentence. If it takes more than one sentence, it's two modules.>

**Public interface:**
```swift
struct TranscriptParser {
  func parse(_ text: String) throws -> Transcript
  func parseStream(_ bytes: AsyncBytes) -> AsyncThrowingStream<TranscriptFragment, Error>
}
```

**Invariants:**
- Parsed transcripts always have at least one speaker
- Timestamps are monotonically increasing
- Unknown speakers are tagged "Speaker N", not null

**Failure modes:**
- Empty input → throw `TranscriptError.empty`
- Unrecognized format → throw `TranscriptError.unsupported(format)`
- Partial data → return best-effort with a warning

**Tests (behaviors to verify):**
- Parses a well-formed VTT file into the expected structure
- Handles speaker labels with non-ASCII characters
- Errors on empty input with the correct error type
- Rejects malformed timestamps
```

**Rules:**
- **Concrete signatures, not prose.** Code blocks are more honest than "a parse method that takes text".
- **Invariants are load-bearing.** They tell future maintainers what must always be true.
- **Failure modes belong next to the interface.** Error handling isn't an afterthought.
- **Tests are behaviors, not implementation steps.** "Parses VTT" not "calls tokenizer then builds speaker map".

### Phase 5: Deliver (sequence the work)

Order the modules for execution. Wrong order causes cascading rework.

**Principles:**

1. **Foundations first.** Modules with no dependencies go first. You need data types before the services that manipulate them.

2. **Tracer bullet through the whole stack.** Before polishing any single module, prove the architecture works end-to-end with a thin vertical slice. This means building a minimal version of every layer to get one happy-path scenario working, then going back to fill in each layer.

3. **Risky modules early.** If something's likely to need rework (unfamiliar API, performance unknown, integration concern), find out early. A failed tracer bullet in week one is cheaper than a failed integration in week three.

4. **Leaf modules last.** Polish, edge cases, and non-critical paths come after the core path is proven.

**Example sequencing** (a hypothetical "add a transcript search feature"):

```
1. TranscriptIndex (foundation — data types + in-memory index)
2. TranscriptIndexer (foundation — populates the index from existing transcripts)
3. SearchQuery (tracer — minimal parser, just exact match)
4. SearchResultsView (tracer — wire UI to display results)
   ← END OF TRACER BULLET: end-to-end exact-match search works
5. SearchQuery (fill in — fuzzy match, operators, highlighting)
6. TranscriptIndexer (fill in — incremental updates, background rebuild)
7. SearchResultsView (fill in — keyboard nav, ranking, preview)
8. Performance tuning (last — only after all behavior is proven)
```

The tracer bullet comes at step 4, not step 8. After step 4 you KNOW the architecture is viable; the remaining work is filling in, not discovering.

### Phase 6: Grill gate

Invoke `max:grill-me` in spec mode against the tech spec. Threshold: 0.2 (same as PRDs — tech specs are what agents execute against).

The grill's dimensions map naturally:
- **Goals** — is the module decomposition obviously correct, or are there unresolved decisions?
- **Acceptance** — does each module have concrete "done when" criteria (tests listed)?
- **Boundaries** — are Non-Goals preserved from the PRD? Any scope creep introduced at the tech spec level?
- **Alternatives** — which decompositions did we consider and rule out?
- **Assumptions** — which constraints are verified (via Phase 2 exploration) vs. guessed?

Iterate on the spec until below threshold, or let the user override.

**Fallback if `max:grill-me` is not available:** self-review against the five dimensions, flag any gaps, ask one clarifying round, and exit.

### Phase 7: Output

Write the tech spec to `docs/tech-specs/<slug>.md` (or caller-specified destination). Print the path and ambiguity report.

Return control to the caller.

---

## Output format

```markdown
# <Feature Name> — Technical Spec

## Context
<Link to the source PRD. Summarize in 2-3 sentences what this spec implements.>

## Decomposition
<Bulleted list of modules with 1-line descriptions. This is the table of contents.>

- **TranscriptIndex** — in-memory index over transcript content
- **TranscriptIndexer** — populates the index from Bugbook pages
- **SearchQuery** — parses search input into executable queries
- **SearchResultsView** — displays ranked results with highlighting

## Modules

### TranscriptIndex
**Responsibility:** ...
**Public interface:** ...
**Invariants:** ...
**Failure modes:** ...
**Tests:** ...

### TranscriptIndexer
...

## Sequence

Ordered list of modules with rationale for the order. Call out the tracer bullet boundary.

## Open Questions

Anything unresolved that the PRD didn't cover. Tag with who needs to answer.

## Ambiguity Report

<Appended by grill-me at exit.>
```

---

## Anti-patterns

- **Restating the PRD as a tech spec.** If your tech spec just paraphrases the PRD in different words, you haven't added value. Tech specs answer "how", not "what".
- **Over-decomposing into shallow modules.** 20 tiny modules with bloated interfaces is worse than 5 deep modules with clean interfaces. When in doubt, merge modules.
- **Skipping the sequencing phase.** A correct decomposition in the wrong order wastes as much time as a wrong decomposition. The sequence is load-bearing.
- **Designing without reading the code.** Tech specs that don't reference existing patterns are aspirational fiction. Read first, design second.
- **Inventing abstractions the codebase doesn't need.** If the team doesn't already use dependency injection, don't propose it as a load-bearing part of the spec. Match the codebase's ambient complexity.
- **Specifying implementation details the spec shouldn't care about.** "Use a `@MainActor` on the view model" is an implementation choice. Tech specs define *what* the module should do, not *how* it's written line-by-line. If you catch yourself writing code in the spec, stop.
