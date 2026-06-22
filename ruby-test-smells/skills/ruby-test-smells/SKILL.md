---
name: ruby-test-smells
description: Review Ruby test suites (RSpec and Minitest) for test smells using Gerard Meszaros' xUnit Test Patterns, thoughtbot's "Let's Not", and Sandi Metz's "Magic Tricks of Testing". Use when asked to review tests/specs, check test quality, audit a spec file or test diff, reduce flaky/slow/fragile tests, or simplify RSpec. Detects smells like Obscure Test, Mystery Guest, Eager Test, General Fixture, Assertion Roulette, Conditional Test Logic, Erratic/Flaky Test, Fragile Test, Slow Tests, Test Code Duplication, and RSpec DSL overuse (let/subject/before), and names the matching fix for each.
---

# Test Smells (xUnit Test Patterns + Ruby practice)

Detect test smells and recommend the matching fix. The framework-agnostic
catalog comes from Gerard Meszaros' *xUnit Test Patterns*; the Ruby-specific
layer comes from thoughtbot's *Let's Not* and Sandi Metz's *Magic Tricks of
Testing*.

- xUnit Test Patterns: http://xunitpatterns.com
- Let's Not (thoughtbot): https://thoughtbot.com/blog/lets-not
- The Magic Tricks of Testing (Sandi Metz): https://www.youtube.com/watch?v=URSWYvyc42M

## How to run a review

1. **Scope the target.** Default to the current diff: `git diff main...HEAD`
   (fall back to `git diff HEAD`). If the user names a spec/test file, review
   that. For a sweep, prefer `spec/` (RSpec) or `test/` (Minitest); start with
   `spec/models`, `spec/services`, `spec/requests` (or the `test/` equivalents).
2. **Run the linters first.** Use whatever the project configured: `bin/rubocop`
   / `bundle exec rubocop`. If `rubocop-rspec` is enabled it catches many of
   these mechanically (e.g. `RSpec/ExampleLength`, `RSpec/MultipleExpectations`,
   `RSpec/LetSetup`, `RSpec/NestedGroups`, `RSpec/InstanceVariable`,
   `RSpec/MessageSpies`, `RSpec/MultipleMemoizedHelpers`). Don't assume it's
   installed.
3. **Read each test against the catalog below.** For every finding report: the
   smell, the `file:line`, *why* it qualifies (cite the heuristic), and the
   specific fix. Prefer the fix that makes the test's cause-and-effect visible.
4. **Rank by payoff.** Lead with smells that make the suite untrustworthy or
   expensive to change (Erratic/Flaky, Fragile, Mystery Guest, Slow) over
   cosmetic ones. A smell is a *prompt to look*, not a guaranteed defect — say
   so when it's justified in context (e.g. a Shared Fixture of immutable
   reference data is fine).

## Output format

Group findings by file. For each:

```
spec/models/order_spec.rb:24 — Mystery Guest (data set up in a `before` block 30 lines away)
  Why: the example asserts on `order.total` but the order and its line items are
  built in a far-off `before`/`let`; the reader can't see the cause of the result.
  Fix: inline the setup into the example (or a named helper called from it) so the
  arrange step sits next to the assert. (Let's Not)
```

End with a short summary: top 3 things worth fixing, and what's fine as-is.

## The catalog: smell → detection → fix

Meszaros groups these as *Code Smells* (Obscure Test, Conditional Test Logic,
Test Code Duplication, Test Logic in Production, Hard-to-Test Code) and
*Behavior Smells* (Assertion Roulette, Erratic Test, Fragile Test, Slow Tests);
they're listed flat below. His third category, *Project Smells* (high
maintenance cost, developers not writing tests, production bugs), is team/PM-level
and out of scope for a code review.

### Obscure Test
- **Detect:** you can't tell what the test verifies at a glance. Usually caused
  by one of the sub-smells below (Mystery Guest, Eager Test, General Fixture,
  Irrelevant Information, Hard-Coded Data).
- **Fix:** make the Four-Phase structure visible (arrange / act / assert);
  inline or name the relevant setup; remove noise.

### Mystery Guest
- **Detect:** the test depends on data or resources it doesn't show — fixtures
  files, seeded DB records, or (in RSpec) `let`/`before` blocks far from the
  example. The cause of the asserted result isn't visible in the example.
- **Fix:** Fresh Fixture — build what the test needs *inside* the test (or a
  clearly named helper/Creation Method called from it). This is the core *Let's
  Not* argument: prefer explicit local setup over `let`/`before`.

### Eager Test
- **Detect:** one test exercises and verifies several behaviors; many unrelated
  assertions; a name like `test_everything`.
- **Fix:** split into one test per behavior — "one concept per test". Each test
  should have a single reason to fail.

### General Fixture
- **Detect:** a big shared fixture (a giant `before(:all)`, factory graph, or
  fixtures file) where each test uses only a slice; setup builds more than the
  test needs.
- **Fix:** Minimal Fixture / Fresh Fixture — construct only the objects this
  test exercises. Reserve Shared Fixture for immutable, read-only reference data.

### Irrelevant Information
- **Detect:** the test shows attributes/values that don't affect the behavior
  under test, drowning the ones that do.
- **Fix:** push defaults into a builder/factory and pass only the values that
  matter to this example, so the relevant inputs stand out.

### Hard-Coded Test Data
- **Detect:** magic IDs/values, especially database keys; assertions coupled to
  specific seeded records.
- **Fix:** generate data per test (factory/builder); never depend on a specific
  primary key or pre-existing row.

### Conditional Test Logic
- **Detect:** `if`/`case`/loops/`rescue`/`begin` or early returns inside a test;
  the test has more than one execution path.
- **Fix:** make the test a linear arrange-act-assert. Use a Guard Assertion
  instead of an `if`; split cases into separate tests or a parameterized table;
  extract loops into a Custom Assertion.

### Test Code Duplication
- **Detect:** copy-pasted setup or assertion blocks across examples/files.
- **Fix:** Extract a named helper / Creation Method for setup and a Custom
  Assertion (RSpec custom matcher / Minitest assertion) for verification. Prefer
  an explicitly-called method over hidden `before` magic (Let's Not).

### Assertion Roulette
- **Detect:** many bare assertions in one test with no messages; when it fails
  you can't tell which assertion broke or why.
- **Fix:** one concept per test; add failure messages; extract a Custom
  Assertion/matcher that reports context. Use `aggregate_failures` deliberately,
  not as a substitute for focused tests.

### Erratic / Flaky Test
- **Detect:** passes and fails inconsistently. Sub-causes: Interacting Tests
  (shared mutable state), Unrepeatable Test (leftover data), Test Run War
  (shared DB), order dependence, and Nondeterminism (real time, `Time.now`,
  random, network).
- **Fix:** Fresh Fixture and full isolation per test; freeze/inject time
  (e.g. `travel_to`/`Timecop`/an injected clock); seed or inject randomness;
  fake external services; never let one test depend on another's state or order.

### Fragile Test
- **Detect:** breaks when unrelated code changes. Interface Sensitivity (renames
  break it), Behavior Sensitivity, Data Sensitivity (DB changes), Context
  Sensitivity (time/locale/env). Overspecified mocks that assert on incidental
  calls are a common cause.
- **Fix:** test observable behavior, not implementation; stub only at real
  boundaries; assert on outcomes, not internal call sequences; use builders so
  data changes don't ripple.

### Slow Tests
- **Detect:** the suite (or a class of tests) is slow enough that people stop
  running it; heavy DB/IO or full-stack tests for logic that could be unit-level.
- **Fix:** push logic down to fast unit tests; minimize DB writes and factory
  graphs; fake external services; reserve Shared Fixture for immutable data;
  keep the test pyramid (few integration, many unit).

### Test Logic in Production
- **Detect:** production code carrying test-only hooks, flags, or branches
  (`if Rails.env.test?`), or methods that exist only for tests.
- **Fix:** inject dependencies and use Test Doubles instead of in-production
  seams; extract a Humble Object so the hard-to-test edge is thin.

### Hard-to-Test Code
- **Detect:** the code under test forces an awkward test — global state,
  singletons, hidden collaborators, `Time.now`/`rand`/network calls inline,
  constructors doing work.
- **Fix:** Inject Dependencies (pass collaborators, clock, RNG, HTTP client);
  separate the functional core from the imperative shell so the core tests
  without setup. (This smell points at a *production* design fix.)

## RSpec DSL overuse (Let's Not)

> **Calibrate before flagging.** `let`/`subject`/`before` are idiomatic in many
> suites, and some style guides (e.g. Better Specs) actively recommend them.
> This is a *contested* convention: follow the project's stated style, and only
> flag DSL use when it genuinely obscures cause-and-effect. Don't mass-flag every
> `let`.

*Let's Not* argues RSpec's DSL is frequently overused, raising maintenance cost
and lowering readability. Flag and prefer plain Ruby:

- **`let` / `let!`** — lazy, memoized setup hoisted away from the example: a
  Mystery Guest. Prefer a local variable in the example, or a well-named helper
  method called explicitly where it's used.
- **`subject` (named or implicit) + one-liner `it { is_expected.to ... }`** —
  hides what's under test and what's being asserted. Prefer an explicit example
  with a clear name and a visible action.
- **`before` for data setup** — moves the arrange step away from the assert.
  Prefer inline setup, or call a named helper from inside the example.
- **`its`** — extracted/deprecated; replace with an explicit expectation.
- **Deeply nested `context`/`describe`** — setup gets smeared across layers.
  Prefer flatter groups with explicit setup per example.

The goal is a test you can read top-to-bottom: arrange, act, assert, all in
view. Reserve `before`/`let` for genuinely shared, incidental wiring — not for
the data the assertion depends on.

## Minitest equivalents

The smells above are framework-agnostic; their Minitest manifestations:

- **`setup`/`teardown` methods** are the Minitest analog of `before`/`after`,
  with the same Mystery Guest risk when the data an assertion depends on is built
  in `setup` far from the test. Prefer building the fixture inside the test method
  (or a named helper) so arrange sits next to assert.
- **Shared `TestCase` base classes or `include`d setup modules** are a common
  General Fixture — each test uses only a slice of broad shared setup. Build only
  what the test needs.
- **`assert`/`refute` without a message** is Assertion Roulette: on failure you
  can't tell which check broke. Pass the message argument
  (`assert total.positive?, "expected a positive total"`) or extract a custom
  assertion.
- **Order dependence** surfaces Erratic Tests. Minitest randomizes order by
  default; an `i_suck_and_my_tests_are_order_dependent!` call is a red flag that
  tests share state. Isolate per-test state instead of pinning the order.
- **Test doubles:** use `Minitest::Mock` and `stub` (or a hand-rolled fake) the
  way the catalog uses RSpec doubles — mock only outgoing commands.
- **`Minitest::Spec`** offers a `let`/`before`/`describe` DSL with the same
  trade-offs as RSpec, so the *Let's Not* guidance above applies equally.

## What to test (Sandi Metz — Magic Tricks of Testing)

Test the **messages crossing your object's boundary**, by message type:

| Message direction | Query (returns a value, no side effect) | Command (has a side effect) |
| :---------------- | :--------------------------------------- | :--------------------------- |
| **Incoming**      | Assert on the returned value             | Assert on the direct public side effect |
| **Sent to self**  | Don't test (private)                     | Don't test (private) |
| **Outgoing**      | Don't test (it's the receiver's incoming) | Expect to send it (mock the message) |

Rules of thumb: test the interface, not the implementation; don't test private
methods directly; don't assert on outgoing queries (you may still stub one to
set up a test — just don't assert on it); only mock outgoing commands.
Keep tests minimal — assert each thing once, in the cheapest test that can prove
it.

## Four-Phase Test (structure every test should show)

1. **Setup (Arrange)** — build the fixture this test needs.
2. **Exercise (Act)** — invoke the behavior under test.
3. **Verify (Assert)** — check the outcome.
4. **Teardown** — release resources (usually automatic in Ruby/Rails).

Keep the four phases visible and in order; a test that blurs them is drifting
toward Obscure Test.

## Fix/pattern reference (when to reach for each)

- **Fresh Fixture / Minimal Fixture** — Mystery Guest, General Fixture, Erratic
  Test: build only what this test needs, inside the test.
- **Creation Method / Test Data Builder** — remove duplicated, noisy setup while
  keeping it explicit and called from the test.
- **Custom Assertion / matcher** — Assertion Roulette, Test Code Duplication:
  name a verification so failures explain themselves.
- **Guard Assertion** — replace conditional logic with an upfront assert.
- **Test Double (stub/mock/fake/spy)** — isolate from slow or nondeterministic
  collaborators; mock only outgoing commands.
- **Inject Dependencies / Humble Object** — Hard-to-Test Code and Test Logic in
  Production: move the awkward edge out of the way.
- **Inline setup / plain Ruby methods** — the Let's Not default: replace
  `let`/`subject`/`before` with visible, explicit setup.

## Notes

- Defer to the project's configured RuboCop/`rubocop-rspec`/Standard — don't
  re-flag what the linter already governs.
- A smell isn't always a defect: a Shared Fixture of immutable reference data,
  or a deliberate end-to-end test, can be the right call. Say why when you let
  one stand.
- Some Ruby style guides (e.g. Better Specs) actively recommend `let`; that
  conflicts with *Let's Not*. Follow the project's stated convention; if none,
  prefer the explicit, readable form.
- After suggesting changes, recommend running the affected tests to confirm they
  still pass and still fail for the right reason.
