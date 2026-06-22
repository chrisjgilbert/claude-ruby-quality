---
name: ruby-code-smells
description: Review Ruby/Rails code for code smells using thoughtbot's Ruby Science catalog (https://github.com/thoughtbot/ruby-science). Use when asked to check for code smells, review Ruby code quality, find refactoring opportunities, or audit a class/method/diff against Ruby Science. Detects smells like Long Method, Large Class, Feature Envy, Case Statement, Shotgun Surgery, Divergent Change, Long Parameter List, Duplicated Code, Uncommunicative Name, STI, Comments, Mixin, and nil-checks, and names the matching refactoring for each.
---

# Code Smells (Ruby Science)

Detect code smells from thoughtbot's *Ruby Science* and recommend the matching
refactoring. Source: https://github.com/thoughtbot/ruby-science

## How to run a review

1. **Scope the target.** Default to the current diff:
   `git diff main...HEAD` (fall back to `git diff HEAD`). If the user names a
   file, class, or method, review that instead. For a whole-app sweep, prefer
   `app/models`, `app/services`, `app/controllers`, and `app/jobs`.
2. **Run the linters first** — they catch the mechanical smells for free.
   Use whatever the project has configured (e.g. `bin/rubocop` or
   `bundle exec rubocop`). If `reek` is available (`bundle exec reek <path>`),
   use it; it maps almost 1:1 to this catalog. Don't assume either is installed.
3. **Read each method/class against the catalog below.** For every finding,
   report: the smell, the `file:line`, *why* it qualifies (cite the heuristic),
   and the specific refactoring to apply. One smell can suggest several
   refactorings — name the best fit first.
4. **Rank by payoff, not count.** Lead with smells that make the code hard to
   change (Divergent Change, Shotgun Surgery, Large Class) over cosmetic ones.
   A smell is a *prompt to look*, not a guaranteed defect — say so when a smell
   is justified in context.

## Output format

Group findings by file. For each:

```
app/services/foo.rb:42 — Long Method (`#call`, 38 lines, 3 levels of nesting)
  Why: does fetching, parsing, and persistence in one method.
  Refactor: Extract Method for each phase; consider Extract Class if the
  parsing logic has its own state.
```

End with a short summary: top 3 things worth fixing, and what's fine as-is.

## The catalog: smell → detection → refactoring

### Long Method
- **Detect:** more than ~10 lines; multiple levels of abstraction in one body;
  comments separating "sections"; deep nesting.
- **Refactor:** Extract Method; Replace Temp with Query; Introduce Explaining
  Variable; Replace Conditional with Polymorphism/Null Object.

### Large Class / God Class
- **Detect:** many instance variables; low cohesion (methods touch disjoint
  subsets of state); the class name is vague (`Manager`, `Helper`, `Service`)
  and keeps growing. Watch fat models and fat controllers especially.
- **Refactor:** Extract Class; Extract Value Object; Extract Decorator; Move
  Method; Replace Subclasses with Strategies; Introduce Form/Parameter Object.

### Feature Envy
- **Detect:** a method calls another object's accessors repeatedly
  (`other.x`, `other.y`, `other.z`) and uses `self` little; logic that belongs
  on the data it operates on. Law of Demeter violations (`a.b.c.d`) are a tell.
- **Refactor:** Move Method onto the envied class; Extract Method then move it;
  Inline Class; add a delegating method (Demeter).

### Case Statement / Type Codes / Conditional Complexity
- **Detect:** `case` on a `type`/`kind`/`status` string or symbol; the same
  `case`/`if` branching on the same condition in more than one place; checking
  `is_a?`/`respond_to?`/`class ==`.
- **Refactor:** Replace Conditional with Polymorphism; Replace Conditional with
  Null Object; Replace Type Code with Subclasses/Strategies; Extract Method to
  name the condition.

### Shotgun Surgery
- **Detect:** one conceptual change forces edits across many files/classes.
  Repeated literals/magic values scattered around.
- **Refactor:** Inline Class / Move Method to consolidate; introduce a single
  object that owns the concept (Value Object, Convention over Configuration).

### Divergent Change
- **Detect:** one class changes for many unrelated reasons (validation,
  formatting, and I/O all in one). The inverse of Shotgun Surgery.
- **Refactor:** Extract Class along the axes of change (Form Object for
  validation, Decorator/Presenter for formatting, a client object for I/O).

### Long Parameter List
- **Detect:** 3+ positional params; booleans-as-flags; params that always
  travel together.
- **Refactor:** Introduce Parameter Object; Introduce Form Object; Extract
  Class; use keyword args; replace flag args with separate methods.

### Duplicated Code
- **Detect:** copy-pasted blocks; parallel conditionals; structurally identical
  methods differing only in a value.
- **Refactor:** Extract Method/Class; Extract Partial (views); Replace
  Conditional with Polymorphism; pull up into a shared object (favor
  composition over a Mixin).

### Uncommunicative Name
- **Detect:** single-letter or numbered names (`x`, `data2`, `tmp`); names that
  restate the type (`user_object`); abbreviations; method names that lie.
- **Refactor:** Rename; Introduce Explaining Variable; Extract Method to name a
  block of logic.

### Single Table Inheritance (STI)
- **Detect:** a `type` column with subclasses; subclasses that don't share most
  columns; conditionals on `type`.
- **Refactor:** Replace Subclasses with Strategies; composition / separate
  tables. Ruby Science is skeptical of STI — flag it, don't auto-condemn.

### Comments
- **Detect:** comments explaining *what* code does (vs *why*); commented-out
  code; a comment that could be a method name.
- **Refactor:** Extract Method with an intention-revealing name; Introduce
  Explaining Variable; delete dead comments. Keep *why* comments.

### Mixin (overused modules)
- **Detect:** modules used to share code rather than model a real "is-a";
  mixins that reach into the host's private state; `concerns/` as a junk drawer.
- **Refactor:** Replace Mixin with Composition; Extract Class; inject a
  collaborator instead of mixing in.

### Nil checks / repeated `&.` / `try`
- **Detect:** scattered `if x.nil?`, `x&.y`, `x.present?` guarding the same
  absent value in many places.
- **Refactor:** Replace Conditional/Nil-check with Null Object; push the default
  to where the value originates.

## Refactoring reference (when to reach for each)

- **Extract Method** — a method does more than one thing / has sections.
- **Extract Class** — a class has more than one responsibility.
- **Extract Value Object** — a primitive has behavior + validation (money, a
  percentage, an identifier).
- **Extract Decorator / Presenter** — view-specific formatting bloating a model.
- **Extract Partial / Validator / Form Object** — Rails-specific extractions.
- **Replace Conditional with Polymorphism** — branching on a type code.
- **Replace Conditional with Null Object** — branching on presence/nil.
- **Replace Subclasses with Strategies / Replace Mixin with Composition** —
  prefer composition over inheritance/mixins.
- **Introduce Explaining Variable / Parameter Object** — tame complex
  expressions and signatures.
- **Inject Dependencies** — hard-coded collaborators (HTTP clients, etc.) make a
  class hard to test; pass them in.

## Notes

- Defer to the project's configured RuboCop/Standard for formatting — don't
  re-flag what the linter already governs as a "smell".
- Don't propose a refactor that breaks a public interface without saying so.
- After suggesting changes, recommend running the project's test suite as the
  verification step.
