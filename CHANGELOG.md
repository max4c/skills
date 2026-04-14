# Changelog

## v0.7.0 — 2026-04-13

Added `--to-tickets` flag to `cookoff` — bridges the grilled-spec-to-`dahso:go` gap so an autonomous pipeline can run from idea to worktree without a human review stop in the middle.

### Added

- **cookoff `--to-tickets`** — after producing the grilled spec, decomposes its top-level numbered sections into Dahso Agent Tickets under a new Active project in the Agent Projects database. Every section becomes at least one ticket regardless of tag: `[AGREED]` copies verbatim, `[PLAUSIBLE DEFAULT]` gets an `## Agent Note` flagging the unconfirmed choice, and `[NEEDS MAX]` gets a ticket with an agent-chosen default plus a note recording which option was picked and why. Skipping `[NEEDS MAX]` is explicitly disallowed — wrong code in a worktree is cheap to delete; skipped work is a wasted night. Sections spanning multiple layers (client vs. Worker vs. CLI) split into separate tickets. The spec's success-criteria / acceptance section becomes a dedicated High-priority integration-test ticket. File overlaps across tickets populate `Linked` fields for `dahso:go`'s lane grouping. Reuses the database-ID cache owned by `dahso:flow` at `~/.dahso/flow-cache.json`.
- **cookoff `--to-tickets --go`** — chains straight to `dahso:go` after ticket creation, skipping the review pause. `--to-tickets` alone stops after the summary so the user can review tickets in the Dahso app first.

### Changed

- **cookoff exit discipline** — added a rule: on ticket-creation failure (CLI error, database not found, malformed schema), print the error and the full grilled spec so the user can create tickets manually. Do not retry in a loop, and do not fall through to `dahso:go` with a partial ticket set.

## v0.6.0 — 2026-04-13

Renamed `multi-grill` → `cookoff`.

### Changed

- **cookoff** (renamed from `multi-grill`) — skill name, heading, description triggers, and all `/multi-grill` invocation examples updated. Behavior unchanged. Breaking for anyone with `/multi-grill` muscle memory; update to `/cookoff`.

## v0.5.0 — 2026-04-13

Added `multi-grill` — grill-me orchestration that runs Claude and Codex as interrogator/answerer pairs so the user reviews the result instead of answering questions live.

### Added

- **multi-grill** — Two modes: relay (iterative Claude↔Codex back-and-forth via `task --resume-last`) and parallel (two swapped streams, each a one-shot Codex call, then merged). Same five-dimension ambiguity rubric as `grill-me`. Tags every resolved decision `[AGREED]` / `[PLAUSIBLE DEFAULT]` / `[NEEDS MAX]` (relay) or `[CONSENSUS]` / `[CONFLICT]` / `[A-only]` / `[B-only]` (parallel) so follow-up attention goes to real judgment calls only. Conservative UNCERTAIN discipline when the artifact references external context. Natural follow-up is `grill-me` scoped to `[NEEDS MAX]` items. **Requires the `openai-codex` plugin** — invokes `codex-companion.mjs task` directly via Bash. First skill in this plugin with a hard external dependency; without Codex installed, the skill will fail on first run with a clear error pointing at the missing script.

## v0.4.0 — 2026-04-10

Added `release-plugin` — a new skill that encodes the discipline of shipping Claude Code plugin updates. Born from the pain of learning (the hard way) that the plugin updater gates on the `version` field in `plugin.json`, not git SHAs, and that bugbook-style multi-branch repos with ongoing WIP need a worktree to avoid disturbing working state.

### Added

- **release-plugin** — Detects the plugin from cwd, recommends patch/minor/major version bump from the diff, updates plugin.json + CHANGELOG, stages surgically (specific paths in WIP repos), commits, pushes, merges to default branch via worktree when needed, and prints the `/plugin update` + `/reload-plugins` commands for the user to run in Claude Code. Pairs naturally with `verify-before-done` (run before release) and `grill-me` (run on hand-wavy CHANGELOG entries).

## v0.3.0 — 2026-04-10

Post-publication review against the skill-creator best-practices rubric.

### Changed

- **grill-me, verify-before-done, tdd** — Description "pushiness" to fight undertriggering. Descriptions now include explicit "use this skill whenever..." framing with broader trigger contexts.
- **write-prd, tech-spec** — Added concrete filled-in examples using the "Cmd+K focuses search bar" feature to anchor the target quality bar.
- **verify-before-done** — Added "When to use this vs. max:tdd" table clarifying the overlap with the tdd skill.
- **grill-me** — Added a Composition section listing all callers so the model sees the ecosystem when the skill loads in isolation.

### Added

- **`.claude-plugin/marketplace.json`** — Standalone marketplace manifest so `/plugin install max@max4c-skills` works predictably without relying on plugin-system auto-detection.

### Fixed

- **README install syntax** — Removed incorrect `github:` prefix from install commands. Corrected to `plugin@marketplace-name` format matching Claude Code's actual install command shape.

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
