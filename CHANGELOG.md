# Changelog

## v0.2.0 — 2026-04-10

All seven skills now fully authored. Skeletons filled in with concrete content, examples, and anti-patterns.

### Updated

- **tech-spec** — Concrete deep-module examples, exploration checklist, module output format with code signatures, worked sequencing example.
- **verify-before-done** — Three-level evidence rubric with work-type mapping table, 12-entry red flag phrase dictionary, anti-pattern list.
- **improve-codebase** — Before/after deep-module examples, detection heuristics per category, full four-persona briefs (Contrarian / Simplifier / Architect / Hacker), worked persona-pass flow.
- **design-interface** — Dimension library spanning navigation / density / interaction / visual / orientation / temporal. Concrete agent brief template for parallel dispatch. Comparison matrix format.

## v0.1.0 — 2026-04-10

Initial release. Seven skills, three deep-authored and four skeletons.

### Deep-authored

- **grill-me** — Socratic interrogation with per-dimension ambiguity scoring (Goals / Acceptance / Boundaries / Alternatives / Assumptions). Thresholds: 0.2 spec, 0.3 ticket, 0.4 freeform. Soft warning gate with informed override.
- **write-prd** — End-to-end PRD generation: context load → codebase exploration → Socratic interview → drafting → grill gate → output. Uses Max Forsey's spec format (Goals / Requirements / Non-Goals / Constraints / Approach / Open Questions). Designed to be invoked standalone or as a subroutine from workflow orchestrators like `bugbook:flow`.
- **tdd** — Vertical-slice tracer bullets + strict RED-GREEN-REFACTOR discipline. Includes a "when NOT to TDD" section acknowledging that SwiftUI views, animations, and system integrations need smoke tests instead.

### Skeletons (structure locked, content deferred to v0.2)

- **tech-spec** — PRD → technical implementation spec. Double Diamond phases (Discover → Define → Design → Deliver). Module decomposition with deep-module emphasis.
- **verify-before-done** — Three-level evidence rubric (Mechanical / Behavioral / Consensus). Red flag list for false "it works" claims.
- **improve-codebase** — Architectural review producing proposals (not commits). Deep-module detection + Ouroboros-inspired persona pass (Contrarian / Simplifier / Architect / Hacker).
- **design-interface** — Parallel UI/UX exploration along explicit dimension axes. Prevents anchoring on the first sketch.

### Inspiration

- [Matt Pocock's skills](https://github.com/mattpocock/skills) — the `write-a-prd` and `tdd` starting structure, the personal skill library shape
- [Ouroboros](https://github.com/Q00/ouroboros) — the ambiguity gate concept and persona pass
- [Superpowers](https://github.com/obra/superpowers) — the RED-GREEN-REFACTOR discipline and `verification-before-completion` principle
