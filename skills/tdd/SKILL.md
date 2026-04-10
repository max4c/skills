---
name: tdd
description: Test-driven development with red-green-refactor discipline and vertical-slice tracer bullets. Use when building features or fixing bugs in testable code, mentions "TDD", "red-green-refactor", "write tests first", or when you want behavior verified through real tests rather than manual smoke tests. Skip for UI code where smoke tests and visual verification are more honest.
---

# TDD

Test-driven development, done right. One test, one implementation, repeat. Tests describe behavior through public interfaces, not implementation details.

Synthesis of Matt Pocock's `tdd` (vertical-slice tracer bullets) and the superpowers `test-driven-development` discipline (strict RED-GREEN-REFACTOR with enforcement). Plus a critical addition: **when NOT to TDD**, for codebases where not every layer is testable.

---

## Philosophy

**Core principle:** Tests verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't.

**Good tests** are integration-style. They exercise real code paths through public APIs. They describe *what* the system does, not *how*. A good test reads like a specification: "parses a wikilink with an alias and produces the expected tokens". These tests survive refactors because they don't care about internal structure.

**Bad tests** are coupled to implementation. They mock internal collaborators, test private methods, or verify through external means (querying a database directly instead of using the interface). The warning sign: your test breaks when you refactor, but behavior hasn't changed. If you rename an internal function and tests fail, those tests were testing implementation, not behavior.

---

## When to use TDD

TDD is the right discipline when **both** conditions hold:

1. The code has a clean public interface you can call from a test
2. The behavior is deterministic and verifiable in-process

Good fits:
- Parsers, serializers, formatters
- Pure functions (sort, filter, transform)
- Service layers with clear contracts (a `TranscriptProcessor` that takes text and returns structured output)
- Data models with invariants (e.g., a cache that must evict oldest entries)
- CLI commands with text I/O
- Anything where a `XCTAssertEqual(actual, expected)` is the right shape of verification

## When NOT to use TDD

TDD is the wrong discipline when the thing you care about isn't verifiable through an in-process assert. Writing tests anyway produces shallow coverage — the test passes but the real bug hides in the layer you couldn't test.

Don't TDD:
- **SwiftUI views** — what you care about is "it renders correctly and the user can interact with it". That's a smoke test or a computer-use verification, not an XCTAssert.
- **Animations and transitions** — subjective, visual, not verifiable in code.
- **Integration with system frameworks** — EventKit, Core Data, AVFoundation. Mocking the framework defeats the purpose; using the real framework makes tests slow and flaky.
- **Network or AI API calls** — unless you're testing the code *around* the call (parsing, retry logic, rate limiting), the call itself needs an end-to-end test.
- **Layout and visual polish** — "does it look right" is a human judgment call.

For these, use smoke tests (launch the app, do the thing, screenshot, compare) or computer-use verification (script the UI). Don't pretend TDD works here.

**Swift-specific note:** in a SwiftUI app, most view code is not TDD-friendly. Parsers, models, services, and pure logic are. When in doubt, ask: "if the test passes, is the behavior I care about actually verified?" If not, don't write the test.

---

## Anti-pattern: horizontal slices

**DO NOT write all tests first, then all implementation.** This is "horizontal slicing" — treating RED as "write all tests" and GREEN as "write all code."

This produces **crap tests**:

- Tests written in bulk test *imagined* behavior, not *actual* behavior
- You end up testing the *shape* of things (data structures, function signatures) rather than user-facing behavior
- Tests become insensitive to real changes — they pass when behavior breaks, fail when behavior is fine
- You outrun your headlights, committing to test structure before understanding the implementation

**Correct approach:** Vertical slices via tracer bullets. One test → one implementation → repeat. Each test responds to what you learned from the previous cycle. Because you just wrote the code, you know exactly what behavior matters and how to verify it.

```
WRONG (horizontal):
  RED:   test1, test2, test3, test4, test5
  GREEN: impl1, impl2, impl3, impl4, impl5

RIGHT (vertical):
  RED → GREEN: test1 → impl1
  RED → GREEN: test2 → impl2
  RED → GREEN: test3 → impl3
  ...
```

---

## Workflow

### 1. Planning (before any code)

- [ ] Confirm the code you're about to write is TDD-appropriate (see "When NOT to use TDD")
- [ ] Identify the public interface. What's the smallest testable surface?
- [ ] List the behaviors to test (not implementation steps)
- [ ] Look for opportunities to extract a **deep module** — small interface, deep implementation. Deep modules are the best TDD targets.
- [ ] Prioritize: which behaviors matter most? Start with the one that proves the end-to-end path works.
- [ ] Get user approval on the test plan before writing code

Ask: "What should the public interface look like? Which behaviors are most important to test first?"

**You can't test everything.** Focus on critical paths and complex logic, not every edge case. Let the user tell you what matters.

### 2. Tracer bullet

Write ONE test that confirms ONE thing end-to-end:

```
RED:   Write test for the most important behavior → test fails (WATCH IT FAIL)
GREEN: Write minimal code to pass → test passes (WATCH IT PASS)
COMMIT
```

**Watch the test fail, then watch it pass.** Skipping the failure observation means you might be writing a test that always passes. Skipping the pass observation means you might be writing code that happens to work but doesn't actually exercise the test path.

The tracer bullet proves the path works end-to-end. It's allowed to be ugly — the point is to get a passing test before adding more.

### 3. Incremental loop

For each remaining behavior:

```
RED:   Write next test → fails (watch it fail)
GREEN: Minimal code to pass → passes (watch it pass)
COMMIT
```

**Rules:**
- One test at a time
- Only enough code to pass the current test
- Don't anticipate future tests
- Keep tests focused on observable behavior, not implementation
- Commit after each green. Small commits are a feature.

**If you catch yourself writing code without a failing test first, stop.** Back out the code, write the test, watch it fail, then write the code. The discipline is what makes TDD produce good tests — skipping the discipline produces bad tests.

### 4. Refactor

After all tests pass, look for cleanup opportunities:

- [ ] Extract duplication
- [ ] Deepen modules (move complexity behind simple interfaces)
- [ ] Apply SOLID principles where natural
- [ ] Consider what new code reveals about existing code
- [ ] Run tests after each refactor step

**Never refactor while RED.** Get to GREEN first, then refactor with tests as your safety net.

---

## Checklist per cycle

```
[ ] Test describes behavior, not implementation
[ ] Test uses public interface only
[ ] Test would survive internal refactor
[ ] I watched the test fail before writing code
[ ] I watched the test pass after writing code
[ ] Code is minimal for this test
[ ] No speculative features added
```

If any item is unchecked, the cycle is incomplete. Complete it or revert.

---

## Red flags

Watch for these and stop when you see them:

- **"I'll write the test after the code works."** You won't. Or if you do, the test will describe what you wrote, not what you intended. Write the test first.
- **"This mock captures what the real code does."** Only if you've verified that — and verifying requires a real test. Mocks are useful at system boundaries, not internal ones.
- **"The test passes, so we're done."** Only if you watched it fail first. Otherwise the test might be tautologically true.
- **"I'll refactor after I add one more feature."** Refactor now, with the current test suite as your safety net. Refactoring under RED is reckless.
- **"Let me just test the shape."** Shape tests (does this function exist, does it return a string) are noise. Test behavior.

---

## Further reading

- Matt Pocock's original `tdd` skill — the vertical-slice framing
- Superpowers `test-driven-development` — the enforcement discipline
- John Ousterhout's *A Philosophy of Software Design* — deep modules vs shallow modules
- Kent Beck's *Test-Driven Development: By Example* — the canonical source
