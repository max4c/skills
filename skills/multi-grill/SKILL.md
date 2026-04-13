---
name: multi-grill
description: Auto-grill a plan, spec, or ticket by running Claude and Codex as interrogator/answerer pairs instead of interviewing the user. Two modes — relay (iterative back-and-forth between Claude and Codex) and parallel (two swapped streams merged into one spec). Produces a pre-chewed artifact where the user only reviews the result, not sits through the interview. Use this skill whenever the user says "multi-grill", "pre-grill this", "have codex grill it", "dual grill", "grill this without me", "run grill-me with codex", "cross-examine this plan", or wants the ambiguity-resolution work done autonomously before they review. Also use when the user has a plan/spec/PRD that would normally go through max:grill-me but explicitly wants to skip being interviewed themselves.
---

# Multi-Grill

Run grill-me style interrogation between Claude and Codex so the user only reviews the output instead of answering questions live. Uses the same five-dimension ambiguity rubric as `max:grill-me` (Goals, Acceptance, Boundaries, Alternatives, Assumptions) but with an AI playing the answerer — and in parallel mode, playing a second griller too.

The point: extract the easy 80% of a `grill-me` pass autonomously. Consensus defaults get baked in, real disagreements get flagged as `[NEEDS MAX]` — so the user's attention is spent only on the genuine judgment calls.

## When to use

- User has an artifact (plan, spec draft, ticket, PRD) and wants it stress-tested without sitting through the interview
- Explicit invocation: "multi-grill", "pre-grill", "have the AIs grill it", "/multi-grill"
- As a pre-filter before `max:grill-me`: multi-grill resolves the obvious questions; follow up with `max:grill-me` on only the `[NEEDS MAX]` items

## When NOT to use

- The user wants to be interviewed themselves — use `max:grill-me` directly
- There's no artifact yet (nothing to grill) — use `max:write-prd` or similar to generate a draft first
- The artifact is trivial (a one-line decision) — overkill

---

## Modes

**relay** — iterative back-and-forth, one question at a time:
- Claude generates a grilling question, sends it to Codex via `task`
- Codex answers as if it were the artifact's author
- Claude evaluates: concrete enough? If vague, follows up with `task --resume-last`
- Loop until the ambiguity rubric is below threshold
- Good for: deep probing on a small artifact where each answer shapes the next question

**parallel** — two streams with swapped roles, merged:
- Stream A: Claude generates a full question set → Codex answers all in one shot (as author-proxy)
- Stream B: Codex generates a full question set → Claude answers all in one shot (as author-proxy)
- Each stream runs as a one-shot Codex call (no iterative resume) so the two streams don't race on `--resume-last` state
- Merge: consensus answers become `[CONSENSUS]`, disagreements become `[CONFLICT: ...]`, unique questions appear tagged with which stream caught them
- Good for: broader coverage on a larger spec — two different grilling styles surface different gaps

Default: if the user doesn't specify, pick relay for short artifacts (<200 lines) and parallel for longer.

---

## Invocation

Typical user inputs:

- `/multi-grill` — use whatever plan/spec is in the current conversation
- `/multi-grill <path>` — grill the file at `<path>`
- `/multi-grill relay <path>` — force relay mode
- `/multi-grill parallel <path>` — force parallel mode

Before starting, confirm the artifact you're about to grill in one line ("Grilling: <path or short description>") so the user can redirect if you picked wrong.

---

## Calling Codex

Invoke Codex directly via Bash. Don't go through `codex:codex-rescue` — that's a one-shot forwarder designed for single rescue requests; multi-grill owns its own Codex conversation.

**Resolve the script path first:**

The companion script lives inside the openai-codex plugin cache. The version directory changes on updates, so always resolve dynamically:

```bash
CODEX_SCRIPT=$(ls -d ~/.claude/plugins/cache/openai-codex/codex/*/scripts/codex-companion.mjs 2>/dev/null | sort -V | tail -1)
if [ -z "$CODEX_SCRIPT" ]; then
  echo "ERROR: codex-companion.mjs not found. Is the openai-codex plugin installed?"
  exit 1
fi
```

Use `$CODEX_SCRIPT` for all subsequent calls. If `CLAUDE_PLUGIN_ROOT` is set in the environment, you can use `${CLAUDE_PLUGIN_ROOT}/scripts/codex-companion.mjs` instead — but the dynamic lookup is the safer default.

**Fresh call:**

```bash
node "$CODEX_SCRIPT" task --fresh "<prompt>"
```

**Continue the same thread:**

```bash
node "$CODEX_SCRIPT" task --resume-last "<next prompt>"
```

Rules:
- Leave `--model` and `--effort` unset unless the user specified them
- Don't pass `--write` — multi-grill is a read/analyze operation, not a code-edit operation
- If Codex returns an error or empty output, stop and report — don't fabricate answers

---

## Relay mode flow

### 1. Load artifact

- File path → Read it
- Conversation reference ("the plan we just discussed") → extract the relevant content into a single artifact blob
- Pasted content → use as-is

### 2. Set Codex's role (first turn)

Fire a fresh `task` with this prompt structure:

```
You are playing the role of the author of the following artifact. Another engineer is going to grill you on it across five dimensions: Goals, Acceptance, Boundaries, Alternatives, Assumptions. Answer each question as concretely as you can, as if you had actually designed this.

Important rules:
- When you're genuinely uncertain — when the real author would need to weigh in — start your answer with "UNCERTAIN:" and explain what you'd need to know.
- When the artifact references external context you don't have access to (e.g., "use the same auth flow as the mobile app", "follow our existing deployment pipeline"), do NOT guess. Start your answer with "UNCERTAIN:" and note the external dependency.
- Brevity over filler. A sentence or two per answer is usually enough.

Artifact:
---
<artifact content>
---

First question: <Claude's first grill question>
```

### 3. Iterate

For each follow-up:

- Use `--resume-last` to continue
- Read Codex's answer, judge it against the rubric (is the relevant dimension now concrete?)
- If vague, push back with a sharper follow-up
- If concrete, move to the next dimension
- Track which dimensions still need work

**Round budget:** Cap at 10 rounds total. To prevent one dimension from starving the others, move on from any single dimension after 3 rounds of follow-up. If a dimension isn't concrete after 3 rounds, flag it `[NEEDS MAX]` and proceed — you're better off covering all five dimensions shallowly than drilling into one while ignoring others.

### 4. Exit

Use the exact exit procedure from `max:grill-me`:

- Judge each of Goals / Acceptance / Boundaries / Alternatives / Assumptions on the 0/0.25/0.5/0.75/1.0 rubric
- Aggregate = mean
- Default threshold: 0.3 (moderate — same as ticket mode in grill-me)
- If below threshold, exit; if above, do one more round of targeted questions on the weakest dimension, then exit regardless

If you hit the 10-round cap without clearing the threshold, exit anyway and flag the residual weak dimensions as `[NEEDS MAX]` in the final spec.

### 5. Render the grilled spec

Rewrite the original artifact with resolved decisions inline. Tag each decision:

- `[AGREED]` — Codex gave a concrete answer Claude found reasonable on first response
- `[PLAUSIBLE DEFAULT]` — Codex answered concretely after a follow-up; reasonable but the user may want to override
- `[NEEDS MAX]` — Codex said UNCERTAIN, or Claude couldn't get to concrete after follow-up

Append the full ambiguity report (same format as `max:grill-me`).

---

## Parallel mode flow

### 1. Load artifact (same as relay)

### 2. Spawn two streams

Fire two Codex calls. Note: these are "concurrent-ish" — Stream A is a single Codex call while Stream B requires a Codex call followed by a Claude answering pass. In practice Stream A often finishes first. That's fine; the merge step waits for both.

Do not try to run iterative `--resume-last` conversations concurrently — `--resume-last` resolves to "the most recent thread for this session" and two parallel iterative streams would race. Each stream is a **single Codex call** with all context inline.

**Stream A prompt (Claude grills, Codex answers):**

Claude first generates a full question set (5-10 questions covering all five dimensions). Then fires one fresh Codex task:

```
You are playing the role of the author of the following artifact. Answer the questions below concretely, as if you had designed this.

Important rules:
- For any question where the real author would need to weigh in, start your answer with "UNCERTAIN:" and explain what you'd need.
- When the artifact references external context you don't have (other systems, prior decisions, existing codebases), do NOT guess — mark UNCERTAIN and note the dependency.
- Keep each answer tight — a sentence or two is usually enough.

Artifact:
---
<artifact content>
---

Questions:
1. <question about Goals>
2. <question about Acceptance>
...
```

**Stream B prompt (Codex grills, Claude answers):**

Claude fires one fresh Codex task asking for a question set:

```
You are reviewing the following artifact. Generate a focused set of 5-10 grilling questions across these five dimensions — Goals, Acceptance, Boundaries, Alternatives, Assumptions. Prioritize questions where two reasonable engineers would build different things if left unresolved. Output as a numbered list, one question per line, tagged with its dimension.

Artifact:
---
<artifact content>
---
```

Codex returns a question list. Claude then answers each one from the perspective of the artifact's author — flagging UNCERTAIN where a real author would need to weigh in (including external context references). This answering step happens in Claude's own context, no Codex call needed.

### 3. Merge

Compare the two resolved-spec outputs. For each decision:

- Same concrete answer in both streams → `[CONSENSUS]`
- Different concrete answers in both streams → `[CONFLICT: A said X / B said Y]`
- Only one stream surfaced the question → include it, tag with origin (`[A-only]` / `[B-only]`) and whether it was resolved or UNCERTAIN
- UNCERTAIN in either stream → `[NEEDS MAX]`

### 4. Present merged spec

Rewrite the artifact with all decisions tagged. Append one combined ambiguity report scored on the merged result.

---

## Output format

Always end the response with:

1. **Grilled artifact** — the original content with resolved decisions inline, each tagged per mode
2. **Ambiguity report** — same format as `max:grill-me`:

   ```
   Ambiguity Report:
     Goals:        0.25  ⚠ one gap
     Acceptance:   0.1   ✓ clear
     Boundaries:   0.0   ✓ clear
     Alternatives: 0.5   ⚠ underexplored
     Assumptions:  0.1   ✓ clear
     ──────────────────────────────
     Aggregate:    0.19  ✓ below threshold (0.3)
   ```

3. **Diff summary** — 3-5 bullets describing what got filled in vs. the original
4. **Next step suggestion** — if any `[NEEDS MAX]` items remain, suggest running `max:grill-me` scoped to those specific items; otherwise say the artifact is ready

---

## Exit discipline

- The user can override at any time ("enough", "ship it", "good enough") — exit immediately, print the report anyway so the override is informed
- If Codex errors out mid-stream, stop and report the error — don't fall back to doing the grill yourself (that defeats the point of the skill)
- Never mark an answer `[AGREED]` or `[CONSENSUS]` to paper over uncertainty — if in doubt, use `[NEEDS MAX]`

## Composition

Multi-grill is user-facing orchestration, not a subroutine. Don't invoke it from inside other skills (unlike `max:grill-me`, which is designed for subroutine use).

Natural follow-up: after multi-grill, if `[NEEDS MAX]` items remain, invoke `max:grill-me` scoped to just those items for a targeted live interview.
