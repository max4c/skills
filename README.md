# max skills

General building skills for Claude Code — project-agnostic skills for turning ideas into verified code.

Inspired by [Matt Pocock's skills](https://github.com/mattpocock/skills), the [Ouroboros](https://github.com/Q00/ouroboros) specification-first workflow, and the [Superpowers](https://github.com/obra/superpowers) plugin.

## What's inside

| Skill | Status | What it does |
|---|---|---|
| `grill-me` | v0.1 | Interview the user relentlessly about a plan, spec, or ticket. Produces a per-dimension ambiguity report and gate. |
| `write-prd` | v0.1 | Turn a vague idea into a grounded PRD. Codebase exploration, Socratic interview, grill gate before exit. |
| `tdd` | v0.1 | Test-driven development with red-green-refactor discipline and vertical-slice tracer bullets. Includes "when NOT to TDD" for UI code. |
| `tech-spec` | v0.1 skeleton | Turn a PRD into a technical implementation spec. Module decomposition, sequencing, grill gate. |
| `verify-before-done` | v0.1 skeleton | Discipline guard — three-level evidence rubric before claiming work is done. |
| `improve-codebase` | v0.1 skeleton | Architectural review that produces proposals, not commits. Deep-module detection + persona pass. |
| `design-interface` | v0.1 skeleton | Parallel exploration of UI/UX designs along explicit dimension axes. |

Skills marked "skeleton" have locked structure but need deeper authoring — the phase outlines are there, the specific content has TODO markers.

## Installation

### Standalone (just these general skills)

```bash
/plugin marketplace add github:max4c/skills
/plugin install max
```

### As a companion to the Bugbook plugin

```bash
/plugin marketplace add github:max4c/bugbook
/plugin install bugbook
/plugin install max
```

The `max4c/bugbook` marketplace bundles both plugins. Installing `max` alongside `bugbook` unlocks active spec grilling and ambiguity scoring inside Bugbook's `/flow`, `/prep`, and `/ticket` workflows.

## Composition

These skills compose with each other and with caller plugins:

```
caller → write-prd → grill-me → ambiguity report → return PRD
                         ↓
                   (grill-me is reusable as a subroutine)
```

Callers (including `bugbook:flow`) pre-load context into the conversation, invoke `max:write-prd`, and post-process the output for persistence. `write-prd` never reaches into external systems directly — it produces markdown; the caller decides where it goes.

## Philosophy

- **Composition over duplication.** Skills invoke each other rather than reimplementing shared concepts.
- **Grounded, not aspirational.** Specs and plans are built on actual codebase reading, not memory.
- **Evidence before assertions.** Nothing is "done" until it's been verified.
- **Qualitative scoring over accounting theater.** The LLM judges clarity directly; don't launder judgment through arithmetic.
- **Active interrogation over passive review.** Grill-me pushes on vague claims until they become specific or the user explicitly overrides.

## Contributing

This is a personal skill library. Issues and PRs welcome, but the bar for accepting changes is "does this match how I work" rather than "is this a general improvement". If you want a different style, fork it.

## License

MIT. See LICENSE.
