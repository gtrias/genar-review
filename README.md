# genar-review

A code review skill with a Socratic, direct voice. Fights complexity, cognitive load, split-brain, and AI slop.

Two modes: PR review (drafts comments in a Socratic style, posts after your approval) and self-review (detailed findings for your own code before pushing).

Works with any stack, any language, any agent.

## How it works

**PR Review:**
1. Fetches the PR diff and reads full files touched
2. Digs deeper when something smells off (follows imports, checks callers)
3. Applies the review lens (complexity, SRP, split-brain, dead code, missing tests, AI slop, etc.)
4. Presents findings with the exact comment text + private rationale
5. You approve, edit, or discard — then it posts via `gh`

**Self-Review:**
1. Reads your branch diff against main
2. Same review lens, but output is direct and detailed (no Socratic questions)
3. No posting — just feedback for you to act on

## What it checks

| Priority | Concern |
|---|---|
| 1 | PR atomicity — doing more than one thing? |
| 2 | PR description — explains the root problem? |
| 3 | Complexity — cognitive load justified? |
| 4 | Single source of truth — split-brain risk? |
| 5 | Root cause — solving the real problem? |
| 6 | Dead code — unused stuff landing? |
| 7 | Tests — business logic without coverage? |
| 8 | Over-engineering — YAGNI? |
| 9 | Future pain — manual steps on every change? |
| 10 | Refactoring opportunities |
| 11 | AI slop — suspiciously generated code |
| 12 | Performance/Security — only when real |

## Voice

Ultra-short. Socratic for others, direct for yourself:

```
# PR review (Socratic)
"Do we really need a generator here? Can we simplify?"
"What happens if the user has no email set?"
"each time we add a provider we'll need to remember to set its priority, right?"
"magic string"

# Self-review (direct)
Issue: Generator used but only yields once. A plain function reduces cognitive load.
Suggestion: Replace with direct return, extract mapping to a named variable.
```

No hedging, no praise, no AI-sounding language. Comments that matter, nothing else.

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
- "Check this PR"

## Agent compatibility

Works with any coding agent that can:
- Read files
- Execute bash commands (git, gh CLI)
- Wait for user confirmation before posting

Tested with: Claude Code, PI.

## License

MIT
