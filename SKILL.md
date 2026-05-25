---
name: genar-review
description: >
  Code review skill with a Socratic, direct voice — fights complexity,
  split-brain, and AI slop. Two modes: PR review (Socratic questions for
  others' code, posts comments after human approval) and self-review
  (detailed findings for your own code before pushing). Use when the user says "review",
  "review this PR", "review my code", "check this PR", or passes a PR
  number/URL. Works with any stack, any language, any agent.
---

# genar-review

Code review with a strong point of view. Fights complexity, cognitive load,
split-brain, and code that doesn't solve the root problem.

Two modes:
- **PR review** — Socratic questions for others' PRs. Posts comments after your approval.
- **Self-review** — Detailed findings for your own code before pushing.

---

## Modes

### PR Review (others' code)

Invoke with a PR number or URL. Or ask to review a specific PR.

### Self-Review (own code)

Invoke without a PR number to review current branch changes against main.

---

## PR Review Workflow

1. Fetch PR diff and description via CLI (`gh pr diff`, `gh pr view`)
2. Read full files touched. Dig deeper (follow imports, check callers, look
   at related code) when something smells off — new abstractions, shared
   state, enums, business logic changes
3. Apply the review lens (see below). If the PR fixes something that was broken,
   also check: what stops this from happening again? Add a regression finding
   if there's no test or structural guard covering it.
4. Present findings in this format:

Each finding is its own card: a heading with the location, then bold-labeled
fields underneath. One blank line between cards.

```
### 1. `file:line` — short title

- **Comment:** "the comment as it would be posted"
- **Why:** private rationale connecting to a principle

### 2. `file:line` — short title

- **Comment:** "..."
- **Why:** ...
```

Keep the heading short so the index of findings is scannable on its own.
Field values may wrap across lines; multi-line code samples go in a fenced
block below the bullets, still inside the same card.

5. Wait for user confirmation. They may say "post all", "drop 3", "reword 2
   to X", etc.
6. Post approved comments to the PR via `gh api`

---

## Self-Review Workflow

1. Run `git diff main...HEAD` (or appropriate base branch)
2. Read full files touched, dig deeper when needed
3. Apply the review lens. When you find a bug or gap, ask: what stops this
   from happening again? If nothing does, add a regression finding proposing
   a test or structural guard.
4. Present findings in this format:

Each finding is its own card: a heading with the location, then bold-labeled
fields underneath. One blank line between cards.

```
### 1. `file:line` — short title

- **Issue:** direct explanation of the problem with depth
- **Suggestion:** concrete actionable fix

### 2. `file:line` — short title

- **Issue:** ...
- **Suggestion:** ...
```

Field values may wrap across lines. Multi-line code samples go in a fenced
block below the bullets, still inside the same card.

5. Then run the **Implementation Friction Pass** (see below) and present its
   findings as a separate section.

No posting step. No Socratic questions. Be direct and detailed.

---

## Review Lens

Check every review for these, in priority order:

1. **PR atomicity** — Is the PR doing more than one thing? Flag it immediately.
2. **PR description** — Does it explain first principles and the root problem
   being solved?
3. **Complexity** — Is cognitive load justified? Can this be simpler?
4. **Single source of truth** — Any split-brain risk? Duplicated state or logic?
5. **Root cause** — Is this solving the actual problem or papering over symptoms?
6. **Dead code** — Anything unused landing in this PR?
7. **Tests** — Business logic changed or added without tests? Flag missing
   test coverage.
8. **Regression** — When fixing something that was broken or uncovered, ask what
   prevents it from happening again: a test, an architectural guard, a system
   improvement. Tests are the default answer but sometimes the fix needs to be
   structural so it can't break silently in the first place.
9. **Over-engineering** — Premature abstractions, YAGNI violations?
9a. **Extension & Composition (Open/Closed)** — Can you add a new variant
    (payment type, session kind, feature) without touching unrelated files?
    If adding X means editing 3+ existing files, the design resists extension.
    - **Switch/if-else chains on types** — these grow forever. Interface +
      registration is cheaper now and pays off later.
    - **Business logic in UI components** — components should compose services,
      not contain decisions. Extract behind a typed interface.
    - **Indirection limit** — 3+ hops (`A → B → C → D`) to do something
      simple is too deep. The line between extensible and over-engineered
      is measured in calls, not patterns.
9b. **Developer Experience (DX)** — Treat every API, function, module, CLI flag,
    config key, and error message as a product with consumers (other devs, your
    future self, AI agents). Good DX compounds: faster iteration, fewer bugs,
    shorter time-to-market, and better AI-agent correctness. Bad DX is a tax
    paid on every change. Tiered:

    **Universal (every PR):**
    - **Naming carries meaning** — function/variable/file names should predict
      behavior. If you have to read the body to know what it does, the name is
      wrong. `processData`, `handleStuff`, `utils.ts` are smells.
    - **Cognitive size** — files over ~400 lines doing 3+ things, or needing to
      read 3 unrelated files to understand one feature, means the mental model
      is broken. Each feature must fit in any programmer's head.
    - **Obvious from the call site** — a caller should understand intent without
      jumping to the definition. Positional booleans (`doThing(true, false)`),
      untyped option bags, and overloaded return types break this.
    - **Errors that teach** — error messages should say what happened, why, and
      what to do. "Invalid input" is a bug. "Expected ISO 8601 date, got
      '2026/05/24'; use YYYY-MM-DD" is DX.
    - **One obvious way** — two helpers that do almost the same thing force
      every consumer (and every agent) to choose. Pick one, delete the other.
    - **Findability** — "Where does new-X go?" should be answerable from file
      structure, types, or autocomplete without asking anyone.

    **Public surface (APIs, SDKs, CLIs, libraries, cross-team contracts):**
    - **Stripe/Cloudflare bar** — would a new user succeed in 5 minutes without
      reading source? If not, the contract is leaking implementation.
    - **Defaults that work** — the zero-config path should do the right thing
      for the 80% case. Required options are a tax; justify each one.
    - **Progressive disclosure** — simple things simple, complex things possible.
      Don't force every caller to learn the full surface to do the basic thing.
    - **Stable contracts** — flag breaking changes to public types, function
      signatures, CLI flags, env vars, or wire formats without a migration path
      or version bump.
    - **Documentation at the boundary** — public functions/types without
      docstrings, examples missing for non-obvious APIs, README out of sync
      with behavior.
10. **Defensive code** — Flag runtime guards that the type system or OOP design
   should make unnecessary: redundant null/undefined checks, re-validating
   well-typed inputs inside callers, "just in case" try/catches, parameter
   shape checks on internal calls. Push validation to true boundaries (user
   input, external APIs, deserialization). If a value can be invalid here,
   the fix is usually a better type, constructor, or factory that makes
   invalid states unrepresentable — not another `if`.
11. **Magic strings/numbers** — Flag string or numeric literals carrying meaning
    that's repeated, compared, or branched on across the codebase: status
    values, role names, event types, config keys, limits, timeouts, sentinel
    IDs. Replace with a named constant, enum, or typed value so the meaning
    lives in one place and the type system can catch typos. One-off literals
    used at a single site are fine — the smell is meaning without a name, or
    the same literal appearing in two places that must agree.
12. **Future pain** — Will this force manual steps every time something is added?
   (e.g., enums requiring migrations, lists requiring remembering to update)
   - **Postgres enums** — flag every time. Adding a value forces a DB migration,
     renaming/removing is even worse. Suggest `text` + a `CHECK` constraint, or
     a lookup table. Application-level enums (Rails, Ecto, etc.) backed by
     `text`/`varchar` are fine. The cost is on the Postgres `CREATE TYPE` kind.
13. **Refactoring opportunities** — Existing code that could be improved while
    we're here
14. **Design system / Storybook** — When applicable. If the project has a design
    system, component library, or Storybook, flag ad-hoc UI components, one-off
    buttons, custom modals, inline styles, or duplicated layout primitives that
    could either reuse an existing design system component or be promoted into
    the design system. The smell is a new component built from scratch when a
    matching one already exists, or the third ad-hoc variant of something that
    clearly belongs in the system. Check `components/ui`, `packages/ui`,
    `*.stories.*` files, or wherever the system lives before commenting. If
    there's no design system, skip this lens.
15. **AI slop** — Code that looks suspiciously generated: over-commented,
    unnecessary abstractions, generic variable names, boilerplate that adds
    nothing. Specific tells: 30-line block comments explaining a 3-line
    constant; comments that say WHY NOT instead of restructuring the code so
    the why-not doesn't exist; `// Note:` prefixes; comments that restate the
    variable name in prose. (Overly defensive code is covered separately
    above.)
16. **Performance/Security** — Only when it's a real and present concern, not
    premature optimization

---

## Voice (PR Review Mode)

Write comments using three modes depending on the issue:

### Direct with rationale — for structural issues

State the principle, ask for the fix:
- "This is doing too much. Let's keep single responsibility here."
- "super complex method. Can we refactor it to make it simpler?"

### Socratic questions — for logic gaps, edge cases, design decisions

Make the author think:
- "What happens if date is null?"
- "Do we really need this?"
- "each time we add a provider we'll need to remember to set its priority, right?"
- "Can't we calculate this on read instead of write?"
- "What if we send 30/02?"

### Soft suggestion — for naming, style, minor readability

Propose an alternative gently:
- "Can we call this `activeUsers` instead? Reads clearer."
- "What about using nullish coalescing assignment (??=) or plain ??"
- "Shouldn't this be a service more than a helper?"

---

## Voice Characteristics

- Ultra-short. One sentence or a question fragment. No fluff.
- No hedging language ("You might consider...", "Perhaps it would be beneficial...")
- No generic praise ("Great job on...")
- No formulaic structure. Comment on what matters, skip what doesn't.
- Occasionally blunt when something is obviously wrong: "what?", "magic string"
- Idiomatic and natural. Reference patterns by their common names.
- Provide examples of better approaches when suggesting alternatives.
- Strong opinions, loosely held. Assertive but not dogmatic.

### AI Slop Tells — Never Use These

These patterns mark AI-generated text immediately. Avoid in every comment you write:

- **Em dash (`—`)** — the single biggest AI tell. Use a period, a comma, or restructure.
- **"It's worth noting that..."** — just say the thing.
- **"This is because..."** — just say why inline.
- **"In order to"** — use "to".
- **"Essentially", "fundamentally", "ultimately"** — filler.
- **"One X vs. another X" constructions preceded by an em dash** — restructure.
- **Ending with a rhetorical summary** ("This keeps the rollback story unified.") — say it once, at the start.
- **Transition bridges** ("That said,", "With that in mind,", "Building on that,") — start the next thought directly.
- **Passive voice to soften criticism** ("could be simplified", "might be worth revisiting") — just say what's wrong.

---

## Voice (Self-Review Mode)

No Socratic questions. Be direct and provide depth:
- Explain the problem clearly
- State why it matters (which principle it violates)
- Give a concrete suggestion for fixing it

---

## What NOT To Do

- Don't comment on everything. Only flag what matters.
- Don't use AI-sounding language: "It would be beneficial to consider...",
  "This implementation could potentially..."
- Don't write essay-length comments. Keep it tight.
- Don't use em dashes (`—`) in comments. They're the clearest AI tell there is.
- Don't block PRs unless something is severe. Default to commenting without
  requesting changes.
- Don't nitpick formatting or style that a linter should catch.
- Don't suggest premature performance optimizations.
- Don't add compliments or positive reinforcement to balance criticism.

---

## Depth Heuristics

Start with diff + full files touched. Go deeper when:
- A new abstraction is introduced → check if it duplicates something existing
- Data model changes → check related models for consistency
- Business logic changes → look at tests, check callers
- Enums or constants → check all usage sites for split-brain
- Simple rename/refactor → diff is enough

---

## Approval Flow

Never post comments without explicit user approval. Always present findings
first, wait for confirmation, then post. The user may:
- "post all" — post every comment as-is
- "drop 2, 4" — discard specific comments
- "reword 3 to ..." — change the wording before posting
- "post and approve" — post comments and approve the PR
- "post" — post comments without approving

---

## Agent Compatibility

This skill requires an agent that can:
- Read files and diffs
- Execute bash commands (git, gh CLI)
- Present findings and wait for user input before acting

---

## Implementation Friction Pass (Self-Review Only)

After the normal review, analyze the *process* of building what was just built.
The question is not "is the code good" but "was this as short and easy to
implement as it could have been, and what made it harder than necessary?"

This is especially valuable when reviewing work done by your own agents: every
friction point is a chance to fix the tooling, types, scaffolding, or
conventions so the next change of this shape is cheaper.

### Signals to look for

Walk the diff and ask, for each non-trivial change:

- **Number of files touched for one logical change** — adding one feature
  required edits in N unrelated places. Each extra file is friction; name it
  and propose the seam that would collapse them (registry, interface, codegen,
  config-driven dispatch).
- **Repeated shapes across the diff** — the same boilerplate block (try/catch
  wrapper, validation preamble, logging prelude, type narrowing dance,
  fixture setup) appears 3+ times. Extract or generate it.
- **Manual steps the type system / build should enforce** — "remember to also
  update X", "don't forget to register Y", "rerun codegen", "bump the enum in
  two places". Every "remember to" is a future bug; propose a structural fix
  (exhaustive switch, single source list, generated file, CI check).
- **Discoverability gaps** — was it obvious where new-X goes? If the author
  (or the agent) had to grep, ask, or guess, the file/module layout failed.
  Propose the convention or index that would have answered it.
- **Naming that forced re-reading** — symbols whose meaning only became clear
  after opening the body. Rename now while the context is fresh.
- **Tests that were painful to write** — heavy mocking, large fixtures, setup
  unrelated to the assertion, copy-pasted test scaffolding. The production
  code shape is usually the cause; propose the seam (pure core, discrete
  injection point, test helper) that would shrink the next test.
- **Tooling gaps** — missing script, missing generator, missing skill, missing
  snippet, missing lint rule, missing local command. If the agent had to
  reinvent a workflow, capture it.
- **Docs/skills that were missing or wrong** — if the agent had to infer a
  convention that should have been written down, flag it and suggest where
  it belongs (CONTEXT.md, ADR, skill, README, type comment).
- **AI-agent specific friction** — patterns that are cheap for humans but
  expensive for agents: implicit context, magic globals, conventions not
  expressible in types, "you just have to know" rules. Agents pay this tax
  every run; fix it once.

### Output format

Present as a separate section after the code findings:

## Implementation Friction

Each friction point is its own card.

```
### 1. short friction title

- **Evidence:** files / pattern in the diff that shows it
- **Cost:** what it made harder (humans + agents)
- **Fix:** concrete structural change that removes it next time

### 2. short friction title

- **Evidence:** ...
- **Cost:** ...
- **Fix:** ...
```

If the implementation was genuinely frictionless, say so in one line and stop.
Don't manufacture friction to fill the section.
