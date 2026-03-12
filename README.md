# /close

A session-closing ritual for Claude Code. Run it when you're done working. It cleans up your project's markdown files, catches missed updates, and writes a handoff so the next session can pick up cold.

## Why

Claude Code sessions are stateless. Without an intentional wrap-up, markdown files drift out of sync, agreed-upon edits get lost, and the next session inherits the mess. Existing handoff tools save session state but ignore project hygiene. /close does both.

/close is manual by design. As a UX designer, I know that closing a terminal is not closing a session. There's no signal that the work is wrapped up. /close is that signal — a deliberate ending that lets you stop thinking about the project.

## What it does

```
■□□□□□□  Redundant or stale files...
■■□□□□□  Memory files...
■■■□□□□  Missed updates...
■■■■□□□  Dirty state...
■■■■■□□  Writing handoff...
■■■■■■□  Self-check...
■■■■■■■  Done.

───────────────────────────────────────
  Housekeeping  ·  updated MEMORY.md
  Handoff       ·  close-handoff.md
  Deferred      ·  2 — update spec, run tests
───────────────────────────────────────

All saved, all good.
```

Seven checks in ~30 seconds:

1. **Stale files** — flags `_old`, `_backup`, `_v2` files, duplicates, contradictory markdown content
2. **Memory** — updates MEMORY.md with durable facts (creates it if absent); plants a breadcrumb so the next session auto-reads the handoff; only touches CLAUDE.md for standing rules
3. **Missed updates** — scans the conversation for edits that were discussed but never applied; verifies spec files reflect the final state, not a mid-session snapshot
4. **Dirty state** — flags temp files, debug logs, scratch work
5. **Handoff** — writes `close-handoff.md` with what was done, what's left, key decisions, failed approaches, and a concrete first action for next session
6. **Self-check** — rereads the handoff to verify it's internally consistent
7. **Summary** — prints the box above

Anything needing human judgment gets deferred to the handoff, not acted on.

### Example handoff

```markdown
## What We Were Working On
Adding OAuth2 login flow to the auth module.

## Current State
Branch: feature/oauth · 2 uncommitted files

## Done This Session
- [x] Google OAuth provider integration
- [x] Token refresh endpoint

## Not Yet Done
- [ ] Apple OAuth provider — blocked on developer account setup
- [ ] Rate limiting on /auth/login — decided on 5/min, not yet implemented

## Key Decisions
| Decision | Rationale |
|----------|-----------|
| Session-based auth over JWT | Simpler revocation, no token blacklist needed |

## Failed Approaches
- Tried passport.js — too much abstraction for our two-provider setup

## Resume Instructions
Implement rate limiting on /auth/login (5 attempts/min). Then pick up Apple OAuth once the dev account is active.
```

## Install

Copy `close.md` to your global Claude Code commands:

```
mkdir -p ~/.claude/commands
cp close.md ~/.claude/commands/close.md
```

That's it. Available as `/close` in every project.

## Usage

At the end of a session, type:

```
/close
```

It writes a `close-handoff.md` in your project root. Consider adding it to `.gitignore`.

If the session was lightweight (no files edited, no decisions made, no unresolved follow-ups), it skips the handoff automatically.

## Design

- **One file, zero dependencies.** No hooks, no MCP server, no config.
- **Safe by default.** Only writes to markdown files. Never deletes, moves, or renames — logs those to the handoff instead.
- **Replace, don't accumulate.** The handoff overwrites the previous one, carrying forward unresolved items. Orientation, not audit trail.
- **Close means close.** Runs to completion without pausing for confirmation.

## License

MIT · [@thomasfogarasy](https://github.com/thomasfogarasy)
