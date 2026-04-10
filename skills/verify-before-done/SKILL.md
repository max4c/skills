---
name: verify-before-done
description: Prevent false "it works" claims. Before declaring anything done, fixed, or passing, run the verification that actually proves it. Use as a discipline guard before commits, PRs, or status updates. Invoke when the user says "verify this", "are you sure it's fixed", or when an agent is about to claim success without evidence. STATUS — v0.1 skeleton, structure complete but rubric content needs deeper authoring.
---

# Verify Before Done

The discipline of proving work is done before claiming it's done. Evidence first, assertions second. Every success claim must be backed by an observable signal.

**This is a Level 3 skeleton.** The principles are locked; the specific verification checklists for different work types need deep authoring.

Inspired by the superpowers `verification-before-completion` skill. Generalized for use by any agent or workflow.

---

## The core rule

**You don't know if the code works until you've seen it work.**

Corollary: "the build passed" is not "it works". "The test is green" is not "the feature is correct". "It compiled" is not "it runs". Each level of evidence answers a different question. Before claiming done, name what's true and what's still untested.

---

## The three levels of evidence

Every verification claim falls into one of three levels. Higher levels require more work but give more confidence.

### Level 1: Mechanical
Machine-checkable signals. No human judgment required.
- Build passes / compile succeeds
- Tests run and are green
- Linter has no errors
- Types check
- CI pipeline passes

**What this proves:** the code is well-formed and satisfies its explicit contracts.
**What this doesn't prove:** the code does what the user wants.

### Level 2: Behavioral
The code actually performs the intended behavior when exercised end-to-end.
- Manual smoke test — launch the app, do the thing, observe the result
- Integration test against the real target (not a mock)
- Script-driven UI verification (computer-use, Playwright, XCUITest)
- Real API call with real credentials
- Database query that confirms the persistence layer worked

**What this proves:** the feature does what it claims to do under realistic conditions.
**What this doesn't prove:** it works for edge cases, other users, or unusual environments.

### Level 3: Consensus
Multiple independent checks agree. Used for load-bearing claims where being wrong is expensive.
- Code review by another person (or another AI model)
- Acceptance test + manual smoke test + peer sanity check
- Multi-environment verification (local + staging + prod-like)
- User confirms the behavior matches their expectation

**What this proves:** the claim is robust across perspectives and environments.
**What this doesn't prove:** nothing will ever break. (Nothing proves that.)

---

## When to require which level

The required level scales with the blast radius of being wrong.

<!-- TODO: write a concrete rubric mapping work types to required levels. Starter:

| Work type | Minimum level |
|---|---|
| Local refactor (renamed variable) | L1 |
| Bug fix in pure logic (parser, formatter) | L1 + L2 (test reproduces bug, then passes) |
| UI feature (new view, new button) | L2 (smoke test, verify navigation wiring) |
| Data migration | L2 + L3 (test on staging data + peer review) |
| Payment or auth code | L3 always |
| Cross-cutting change (shared library) | L3 always |

Refine this table based on the workflow the caller is running. Generic rules here; callers can tighten. -->

---

## The verification checklist

Before claiming "done":

- [ ] **Name the claim.** What specifically are you saying works? "It compiles" vs. "the user can checkout" are different claims.
- [ ] **Choose the evidence level.** Pick L1, L2, or L3 based on the claim's blast radius.
- [ ] **Run the verification.** Actually execute the check — don't describe what you'd do, do it.
- [ ] **Read the output carefully.** Not just the exit code. Look for warnings, skipped tests, partial failures.
- [ ] **State the result with precision.** "Build passed, 127 tests green, no warnings" not "looks good".
- [ ] **Note what's still untested.** If the verification was L1 only, say "build passes; behavior not yet verified".

---

## Red flags

Watch for these and pause when you see them. Each is a sign someone is about to claim something untrue.

- **"Should work."** Claim needs verification, not a hedge.
- **"I think it's fine."** Think is not know. Run the check.
- **"The tests are probably passing."** Run the tests. "Probably" is not a verification.
- **"It compiled, so it works."** Compilation proves syntax, not behavior.
- **"I made the same change as before and that one worked."** Might be, might not be. Verify this specific change.
- **"I don't need to test this — it's a trivial fix."** Trivial fixes have the highest rate of regressions because people skip verification.
- **"The user will catch it."** That's a failure mode, not a process.

When you catch yourself about to say any of these, stop and go back to the verification step.

<!-- TODO: add more red flags based on real failure modes. Matt Pocock and obra both have good examples in their writing. -->

---

## Integration with other skills

**From /tdd:** TDD's GREEN step already requires watching the test pass. verify-before-done extends that discipline to non-TDD work (UI, integration, cross-cutting changes).

**From /grill-me's ambiguity report:** the "Acceptance" dimension scores how verifiable the claim is. Low scores on Acceptance mean the verification is likely to be vague — tighten the claim before verifying.

**From bugbook:go:** /go's step 6 already implements a Bugbook-specific version of this principle (launch app, navigate, eval, screenshot). verify-before-done is the generic version for agents outside Bugbook.

<!-- TODO: write more concrete integration examples. Show how this skill composes with other skills in practice. -->

---

## Anti-patterns

- **Verification theatre.** Running commands without reading the output, just to claim "I checked". Worse than not checking.
- **Verification scope creep.** Claiming L3 when you only did L1, because the claim felt important. Calibrate honestly.
- **Skipping verification because "it's obvious".** The obvious fixes are exactly the ones that get rushed. Verify anyway.
- **Writing "verified: yes" without saying how.** Always state the evidence: which test, which smoke, which review.

<!-- TODO: write 2-3 more anti-patterns. What failure modes have you seen in practice? -->
