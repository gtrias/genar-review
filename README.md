# genar-review

A code review skill with a Socratic, direct voice. Fights complexity,
cognitive load, split-brain, and AI slop.

Three modes:
- **PR review**: drafts comments in a Socratic style, posts after your approval.
- **Self-review**: detailed findings for your own code before pushing.
- **Artifact review**: detailed findings for a plan, spec, doc, single file, or pasted text.

Works with any stack, any language, any agent.

## How it works

**PR review**
1. Fetches the PR diff and reads full files touched.
2. Digs deeper when something smells off (follows imports, checks callers).
3. Applies the review lens (see `SKILL.md`).
4. Presents findings with the exact comment text and private rationale.
5. You approve, edit, or discard. Then it posts via `gh` (or the GitHub MCP backend).

**Self-review**
1. Detects the base branch and diffs committed, staged, and unstaged changes.
2. Same review lens, output is direct and detailed (no Socratic questions).
3. Adds an Implementation Friction Pass: what made this harder than it needed to be.
4. No posting. Feedback for you to act on.

**Artifact review**
1. Reads the artifact (plan, spec, RFC, ADR, PRD, design doc, README, prompt, skill, single source file, pasted text) and any referenced material.
2. Applies the lens filtered to what is relevant (skips code-only items like defensive code or design system).
3. No posting. Direct findings.

The skill picks the mode from your request. If it cannot tell, it asks once.

## What it checks

The full lens lives in `SKILL.md`. Highlights: atomicity, root cause,
complexity, single source of truth, dead content, tests and regression
guards, extension and composition, developer experience, defensive code,
magic literals, future pain, design system fit, AI slop.

## Voice

Ultra-short. Socratic for others' PRs, direct for everything else.

```
# PR review (Socratic)
"Do we really need a generator here? Can we simplify?"
"What happens if the user has no email set?"
"each time we add a provider we'll need to remember to set its priority, right?"
"magic string"

# Self-review and artifact review (direct)
Issue: Generator used but only yields once. A plain function reduces cognitive load.
Suggestion: Replace with direct return, extract mapping to a named variable.
```

No hedging, no praise, no AI-sounding language, no em dashes. Comments
that matter, nothing else.

## Installation

```bash
# Install globally (works with any agent)
npx skills add gtrias/genar-review -g

# Or project-level
npx skills add gtrias/genar-review
```

## Usage

Tell your agent:
- "Review PR #123"
- "Review https://github.com/org/repo/pull/123"
- "Review my code" (self-review mode)
- "Review this plan" (paste or point to the artifact)
- "Check this PR"

## Agent compatibility

Any coding agent that can:
- Read files.
- Execute bash commands (git, gh CLI), or call equivalent MCP tools.
- Wait for user confirmation before posting.

Tested with: Claude Code, PI.

## License

MIT
