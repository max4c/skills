---
name: verify-before-done
description: Prevent false "it works" claims. Before declaring anything done, fixed, or passing, run the verification that actually proves it. Three-level evidence rubric (Mechanical / Behavioral / Consensus) with red flag detection for phrases like "should work", "looks good", "probably passing". Use this skill whenever you or the user is about to claim something is complete, fixed, working, or passing — even if the user hasn't explicitly asked for verification. Triggers include "I fixed it", "it works now", "tests are green", "ready to merge", "deploy it", or any claim of success that lacks an observable signal. Also invoke when reviewing agent output before committing or when an agent is about to set a ticket to Review/Done status.
---

# Verify Before Done

The discipline of proving work is done before claiming it's done. Evidence first, assertions second. Every success claim must be backed by an observable signal.

Inspired by the Superpowers `verification-before-completion` skill. Generalized for use by any agent or workflow.

---

## The core rule

**You don't know if the code works until you've seen it work.**

Corollary: "the build passed" is not "it works". "The test is green" is not "the feature is correct". "It compiled" is not "it runs". Each level of evidence answers a different question. Before claiming done, name what's true and what's still untested.

---

## The three levels of evidence

Every verification claim falls into one of three levels. Higher levels require more work but give more confidence. Match the level to the blast radius of being wrong.

### Level 1: Mechanical
Machine-checkable signals. No human judgment required.

- Build passes / compile succeeds
- Tests run and are green
- Linter has no errors
- Types check
- CI pipeline passes

**What this proves:** the code is well-formed and satisfies its explicit contracts.

**What this doesn't prove:** the code does what the user wants. Compilation means syntax is valid. Tests green means *the tests you wrote* pass — not that you wrote the right tests.

**How to run it:**
```
<language-appropriate build> && <test command>
```
Read the output. Not just the exit code — scan for warnings, skipped tests, partial failures.

### Level 2: Behavioral
The code actually performs the intended behavior when exercised end-to-end.

- Manual smoke test — launch the app, do the thing, observe the result
- Integration test against the real target (not a mock)
- Script-driven UI verification (computer-use, Playwright, XCUITest, Puppeteer)
- Real API call with real credentials (on staging, not prod)
- Database query that confirms the persistence layer worked

**What this proves:** the feature does what it claims under realistic conditions. The user's happy path actually works.

**What this doesn't prove:** it works for edge cases, other users, or unusual environments. You verified ONE execution; the next one might surface a race, a locale issue, a network fault, a timezone bug.

**How to run it:**
1. Launch the real thing (app, service, CLI)
2. Execute the behavior the way a user would
3. Observe the outcome — visually, or by querying a downstream signal (database row, log entry, API response)
4. Write down exactly what you observed

### Level 3: Consensus
Multiple independent checks agree. Used for load-bearing claims where being wrong is expensive.

- Code review by another person (or another AI model with different training)
- Acceptance test + manual smoke test + peer sanity check
- Multi-environment verification (local + staging + prod-like)
- User confirms the behavior matches their expectation
- Staged rollout with telemetry confirming expected behavior in real traffic

**What this proves:** the claim is robust across perspectives and environments. Reduces the chance that one verification missed a whole class of failure.

**What this doesn't prove:** nothing will ever break. (Nothing proves that.)

**How to run it:**
- Minimum two independent reviewers
- Minimum two distinct verification paths (e.g., unit tests + smoke test, not two unit test runs)
- Named dissent — if a reviewer flagged a concern, address it explicitly, don't hand-wave

---

## When to require which level

The required level scales with the blast radius of being wrong.

| Work type | Minimum level | Rationale |
|---|---|---|
| Local refactor (renamed variable, moved function) | L1 | If it compiles, semantics are preserved |
| Bug fix in pure logic (parser, formatter, pure function) | L1 + unit test that reproduces the bug | Test captures the specific fix; L1 confirms no regression |
| Bug fix in stateful logic (caching, concurrency, I/O) | L2 | Tests can miss race conditions; run the real thing |
| New internal utility with clean interface | L1 + unit tests of the behavior | TDD-appropriate; L2 is overkill |
| UI feature (new view, new button, new flow) | L2 | "It compiles" doesn't prove it's wired into navigation; smoke test is the only honest check |
| Data migration (schema change, backfill) | L2 + L3 | Test on staging data with a rollback plan; get a peer eye on the migration script |
| Payment or auth code | L3 always | Cost of a silent failure is catastrophic |
| Cross-cutting change (shared library, base class) | L3 always | One change, many callers — must verify the blast radius |
| Infrastructure change (CI, deploy, env vars) | L2 + actual deploy observed | You don't know if a deploy config works until it deploys |
| Delete dead code | L1 + grep for references | Prove nothing still calls it |

Pick the minimum level for your work type. Higher is always OK; lower is never OK.

---

## The verification checklist

Before claiming "done":

1. **Name the claim.** What specifically are you saying works?
   - Bad: "it works"
   - Good: "the sidebar drag feature now preserves item order after drop on macOS 15 in the primary window"

2. **Choose the evidence level.** Pick L1, L2, or L3 based on the claim's blast radius.

3. **Run the verification.** Actually execute the check. Don't describe what you'd do — do it.

4. **Read the output carefully.** Not just the exit code. Look for warnings, skipped tests, partial failures, stderr content, timing anomalies.

5. **State the result with precision.**
   - Bad: "looks good"
   - Good: "xcodebuild passed, 147 tests green, 0 failures, 2 warnings (both pre-existing)"

6. **Note what's still untested.** If the verification was L1 only, say so.
   - Good: "build passes; behavior in the actual app not yet verified"
   - Bad: silence about the gap

---

## Red flags

Watch for these phrases in yourself and others. Each is a warning sign that a false "it works" claim is imminent.

| Phrase | What it actually means | What to do |
|---|---|---|
| "Should work." | You didn't verify. "Should" is a hedge. | Run the check |
| "I think it's fine." | Think is not know. | Run the check |
| "The tests are probably passing." | You haven't looked. | Read the test output |
| "It compiled, so it works." | Compilation proves syntax, not behavior. | Run it, not just build it |
| "I made the same change as before and that one worked." | Different context, different potential outcome. | Verify this specific change |
| "I don't need to test this — it's a trivial fix." | Trivial fixes have the highest rate of regressions because people skip verification. | Verify anyway |
| "The user will catch it." | That's a failure mode, not a process. | You're the filter, not the user |
| "The CI will catch it." | CI catches the things it tests for, not everything. | Check what CI actually runs |
| "I'll verify after I commit." | You won't. | Verify before committing |
| "It worked on my machine." | Useless as a verification claim. | Run it where it matters (staging, prod-like, CI) |
| "The types look right." | Type-check is L1. Behavior check is L2. | Run the thing |
| "I manually traced through the logic." | Mental simulation is not verification. | Let the computer check |

When you catch yourself about to say any of these, stop and go back to the verification step.

---

## Anti-patterns

- **Verification theatre.** Running commands without reading the output, just to claim "I checked". Worse than not checking because it produces false confidence.
- **Scope inflation.** Claiming L3 when you only did L1, because the claim felt important. Calibrate honestly — L1 is fine for L1-appropriate work.
- **Skipping verification because "it's obvious".** The obvious fixes are exactly the ones that get rushed. Verify anyway.
- **Writing "verified: yes" without saying how.** Always state the evidence: which test file, which smoke test steps, which review.
- **Celebrating green tests without reading them.** If the test output scrolled past faster than you could read, you didn't verify — you watched a light turn green.
- **Using mocks as verification.** Mocks verify your mock, not the real system. At system boundaries, use real integrations.
- **Testing the happy path only.** The happy path is usually fine. Bugs hide in the failure modes. At least one verification should be a failure case.

---

## When to use this vs. `max:tdd`

Both skills cover "prove it works before claiming it works". The split is by what kind of code you're verifying:

| Work type | Use this |
|---|---|
| Parser, formatter, pure function, data model | **`max:tdd`** — write failing test, watch it fail, write minimal code, watch it pass |
| Service with a clean public interface | **`max:tdd`** — same cycle, integration-style tests |
| SwiftUI view, animation, layout | **`verify-before-done`** — L2 smoke test, can't meaningfully unit-test |
| System framework integration (EventKit, Core Data) | **`verify-before-done`** — L2 with the real framework |
| Bug fix where you first reproduce, then fix | **Both** — tdd for the test that reproduces, verify-before-done for the overall discipline |
| Cross-cutting refactor touching many files | **`verify-before-done`** — L2 or L3 with the full system |
| Deployment, config, infrastructure | **`verify-before-done`** — L2 observed deploy |

**Short rule:** `max:tdd` for testable layers (clean interfaces, deterministic, in-process). `verify-before-done` for everything else. If both apply, use tdd for the logic and verify-before-done for the end-to-end check.

---

## Integration with other skills

**From `max:tdd`:** TDD's GREEN step already requires watching the test pass. verify-before-done extends that discipline to non-TDD work (UI, integration, cross-cutting changes).

**From `max:grill-me`:** grill-me's Acceptance dimension scores how verifiable the claim is. Low scores on Acceptance mean the spec will be hard to verify — tighten the spec before you try to verify the implementation.

**From `bugbook:go`:** /go's step 6 already implements a Bugbook-specific version of this principle (launch app, navigate, execute eval steps, screenshot, confirm). verify-before-done is the generic principle; /go is the Bugbook-specific implementation. Both are OK — they're not duplicates, they're the principle and its application.

**From code reviewers (`feature-dev:code-reviewer`, `superpowers:code-reviewer`):** these agents provide L3 consensus as a second independent check. Invoke them for cross-cutting or risky changes.

---

## Further reading

- Obra's `verification-before-completion` skill in Superpowers — the canonical source for this discipline
- *Site Reliability Engineering* (Google) — chapters on monitoring, alerting, and rollout verification
- Gall's Law — "a complex system that works is invariably found to have evolved from a simple system that worked" — applies to verification too: don't invent complex verification schemes when simple ones work
