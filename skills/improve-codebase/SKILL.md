---
name: improve-codebase
description: Architectural review and improvement proposals. Reads the codebase for structural opportunities — shallow modules that should be deepened, leaky abstractions, misplaced complexity — and proposes specific changes with tradeoffs. Uses a four-persona pass (Contrarian, Simplifier, Architect, Hacker) to stress-test proposals before presenting. Not a refactoring bot; it produces proposals, not commits. Use when the user says "improve-codebase", "architectural review", "where is this codebase weak", or before starting a big refactor.
---

# Improve Codebase

Architectural review that produces proposals, not commits. Read the codebase, identify structural improvement opportunities, and present them with tradeoffs for the user to decide. The output is a decision document.

Inspired by Matt Pocock's `improve-codebase-architecture`, John Ousterhout's *A Philosophy of Software Design*, and Ouroboros's "Nine Minds" persona system. Synthesizes shallow-module detection with multi-perspective review.

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

**The goal of good architecture is to maximize depth.** improve-codebase looks for shallow modules and proposes ways to deepen them, either by (a) absorbing caller logic into the module, (b) narrowing the interface, or (c) merging the module into a bigger one.

### Concrete examples

**Example 1 — A shallow wrapper that should be deepened**

```typescript
// BEFORE (shallow): 8 public methods, each a pass-through
class UserRepository {
  findById(id: string) { return db.users.where({id}).first() }
  findByEmail(email: string) { return db.users.where({email}).first() }
  findAll() { return db.users.toArray() }
  create(user: User) { return db.users.insert(user) }
  update(id: string, patch: Partial<User>) { return db.users.where({id}).update(patch) }
  delete(id: string) { return db.users.where({id}).delete() }
  count() { return db.users.count() }
  exists(id: string) { return db.users.where({id}).count() > 0 }
}
```

The interface is as wide as the query capabilities of the underlying database. Every caller has to decide which method to use, and adding a new query means adding a new method. This is shallow — the repository isn't adding anything beyond routing to the database.

```typescript
// AFTER (deep): 2 methods that capture the actual business concerns
class UserRepository {
  find(criteria: UserCriteria): Promise<User[]>
  save(user: User): Promise<User>
  // Behind the scenes: caching, denormalization, event emission, soft-delete handling
}
```

The interface is narrower, but the implementation can now do more — cache results, emit events on save, handle soft-deletes uniformly. Callers don't know or care.

**Example 2 — A leaky abstraction**

```swift
// BEFORE: the public interface exposes internal representation
struct MarkdownDocument {
    var tokens: [Token]
    var parseState: ParseState
    var sourcePositions: [SourcePosition]

    func render() -> String { ... }
}
```

Callers can poke at `tokens` and `parseState` directly. Anyone who does becomes coupled to the internal representation. When you want to change how parsing works, you can't — callers depend on the tokens.

```swift
// AFTER: public interface is just what callers actually need
struct MarkdownDocument {
    func render() -> String
    func plainText() -> String
    func outline() -> [Heading]

    private var representation: Representation  // implementation detail
}
```

### Other structural weaknesses

**Leaky abstractions** — interfaces that require callers to know implementation details. Watch for:
- Public types that expose internal data structures
- Methods named for what they do internally rather than what they accomplish
- Return types that force callers to handle intermediate states

**Misplaced complexity** — business logic in the wrong layer. Watch for:
- Data transformations in view code
- UI state management in model code
- Configuration scattered across multiple files with no single source of truth
- State management spread across layers (some in the model, some in the view, some in a singleton)

**Duplication patterns** — the same concept implemented multiple ways. Watch for:
- The same transformation implemented in three places
- Copy-pasted code with small variations (the variations usually indicate a missing parameter or a missing abstraction)
- Multiple ways to do the same thing (two date formatters, three HTTP clients, four error types for the same failure mode)

---

## Phases

### Phase 1: Orient

Before looking for problems, understand what the codebase does.

**Orientation checklist (adapt to codebase type):**

**Web app:**
- Read `package.json` / `pyproject.toml` / equivalent — what framework? what version?
- Read `src/index.*` or equivalent entry point
- Read `README.md` if it exists
- Look at the top-level src/ directories — how is code organized (by feature, by layer, by domain)?
- Look at `routes/` or `pages/` to understand user-facing surface

**macOS/iOS app:**
- Read `Package.swift` or `*.xcodeproj` structure
- Read `App.swift` / `AppDelegate.swift` entry point
- Look at `Sources/` or `[AppName]/` top-level — feature folders, MVVM, VIPER?
- Look at `Views/` vs `Models/` vs `Services/` — is the separation clean?

**CLI tool / library:**
- Read the main entry point
- Read the public API surface (whatever's exported or in `lib/`)
- Read one or two example usages from tests or docs

**Ask the user:** "What area should I focus on?" Options they might pick:
- A specific directory
- "Anywhere" — in which case, look at the directory with the highest recent churn (`git log --format= --name-only | sort | uniq -c | sort -rn | head`)
- "Something that's been bugging me" — follow up with "which thing?"

**Time budget:** 10-15 minutes. Don't read every file. Build a mental model sufficient to identify candidates.

### Phase 2: Identify candidates

Walk the codebase looking for structural weaknesses. For each candidate, capture: file:line, category, what's wrong, why it matters, estimated effort to fix.

**Detection heuristics per category:**

**Shallow module detection:**
- Count public methods per class. 10+ methods is a smell.
- Read each public method's implementation. If it's <5 lines and does nothing beyond delegation or type conversion, it's shallow.
- Look for "Manager", "Helper", "Utility", "Service" classes — these names often signal shallow modules.
- Check: "If I removed this class and inlined its methods, would anything be lost?" If no, it's shallow.

**Leaky abstraction detection:**
- Public types that return internal representations (e.g., returning a database row directly)
- Methods that force callers to handle intermediate states (e.g., `startLoading() / finishLoading()` instead of `load()`)
- Classes with methods that exist only for tests (`getInternalState()`, `setTestMode()`)

**Misplaced complexity detection:**
- Business logic inside view files (calculations, validations, API calls)
- Models with UI concerns (loading states, error messages for users)
- Configuration in multiple incompatible places (env var here, constant there, config file somewhere else)

**Duplication detection:**
- Grep for similar function bodies
- Look for types with overlapping fields
- Multiple implementations of the same concept (date parsing, HTTP retry, error serialization)

For each candidate, resist the urge to describe the fix yet. Just catalog the weakness.

### Phase 3: Persona pass

Before writing the proposal, run each candidate past four mental perspectives. Each persona asks a different question. Run them in parallel mentally — don't let one persona anchor the others.

**Contrarian** — *"What if the opposite were true?"*
- Is the "shallow module" actually correct because the interface is a stable API boundary that shouldn't be refactored?
- Is the "leaky abstraction" actually fine because it's internal-only and the leak is a performance optimization?
- Am I proposing a change because it's better, or because it's different from how I would have written it?
- Would the user who originally wrote this code have a reason I'm missing?

If the Contrarian's question lands, DROP the candidate. Not every "shallow" module is wrong; some are boundaries.

**Simplifier** — *"What's the simplest thing that could work?"*
- Is there a proposal that's smaller and achieves 80% of the benefit?
- Can we just delete code instead of refactoring it?
- Are we inventing a new abstraction when an existing one in the codebase would do?
- Is the problem big enough to warrant any change at all?

If the Simplifier offers a smaller alternative, REPLACE the candidate with the smaller version.

**Architect** — *"If we started over, would we build it this way?"*
- Is this a symptom of a deeper structural problem upstream?
- Does this proposal fix the real issue, or just paper over it?
- What's the 3-year maintenance implication of fixing this vs not fixing it?
- Is there a cross-cutting pattern that would solve this candidate AND three others at once?

If the Architect identifies a bigger fix that subsumes the candidate, promote the bigger fix.

**Hacker** — *"What constraints are actually real?"*
- Is there a pragmatic shortcut we're missing because we're over-architecting?
- What would the solution look like if we ignored "the right way"?
- Are the constraints we're working around actually constraints, or just habits?
- Could we just delete the problem instead of solving it?

If the Hacker proposes "just delete it", that's often the best answer.

**Run all four per candidate.** A candidate that survives all four personas with its original shape is a strong proposal. A candidate that gets rewritten by one persona and dropped by another is probably not ready.

### Phase 4: Write the proposal

For each surviving candidate, write a proposal section:

```markdown
### Proposal N: <Short action-oriented title>

**What:** <1-sentence description of the change>

**Where:** <Files and directories affected. Be specific — file:line ranges if possible.>

**Why:** <What problem this solves. Include concrete pain points — "this class has been touched in 12 bug fixes over 6 months" is more compelling than "this class is shallow". Cite git history, bug reports, or specific code smells.>

**How:** <Sketch of the change. Not code — direction. "Merge FooWrapper into Foo and narrow the public interface to three methods: X, Y, Z.">

**Tradeoffs:**
- Pro: <specific benefit — be concrete, not "cleaner code">
- Con: <specific cost — churn, risk, learning curve for other contributors>
- Risk: <what could go wrong, what's the worst case>

**Effort:** <Small (< 1 day) | Medium (1-3 days) | Large (> 3 days)>

**Persona notes:**
- Contrarian: <what the contrarian asked and how the proposal survives it>
- Simplifier: <what the simplifier offered, whether the proposal adopted it>
- Architect: <what the architect saw>
- Hacker: <what the hacker would do instead, if anything>
```

Present 2-5 proposals. If you can't find that many strong candidates, present fewer. Don't pad. A proposal document with 2 strong proposals is better than one with 5 mediocre ones.

### Phase 5: Grill gate

Invoke `max:grill-me` in spec mode against the proposal document. The grill will push on:
- Are the "why" statements concrete, or hand-wavy?
- Do the tradeoffs actually acknowledge costs, or are they marketing?
- Is the effort estimate grounded, or optimistic?
- Have we considered the Contrarian's objection fairly?

Iterate until below the 0.2 threshold or the user overrides.

### Phase 6: Output

Write the proposal to `docs/architecture-reviews/<date>-<focus>.md`. Print the path and the number of proposals.

Do not create tickets, open PRs, or make commits. That's the user's call after reading.

---

## Anti-patterns

- **Refactoring is not improvement.** A change that moves code around without reducing complexity isn't an improvement. It's just churn. Every proposal should make the codebase smaller, simpler, or more resilient — not just different.
- **Proposing changes you wouldn't do yourself.** If you wouldn't actually make this change, don't propose it. Proposals should be serious, not speculative.
- **Reviewing from memory.** You have to read the code. Proposals based on "I remember this codebase has problem X" are usually wrong — memory compresses the wrong things.
- **Bigger is better.** A proposal that touches 50 files isn't inherently better than one that touches 5. Often it's worse. Propose the smallest change that produces the benefit.
- **Ignoring the why.** A proposal without a specific pain point is a solution looking for a problem. The best proposals reference actual bug reports, frequent-churn files, or developer complaints.
- **Universal advice.** "This codebase should use dependency injection everywhere" is not a proposal, it's a preference. Good proposals are specific to this codebase and its actual state.
- **Skipping the persona pass.** The personas exist to catch motivated reasoning. If you skip them, your proposals reflect your biases, not the codebase's needs.

---

## Further reading

- John Ousterhout, *A Philosophy of Software Design* — especially chapters on deep modules, information hiding, and complexity
- Martin Fowler, *Refactoring* — when you know the target pattern, this is the playbook
- Matt Pocock's original `improve-codebase-architecture` skill — the starting framing
- Fred Brooks, *No Silver Bullet* — a reminder that architectural improvements are marginal, not transformative
