# /close

A session-closing ritual for Claude Code. Run it when you're done working. It cleans up your project's markdown files, catches missed updates, and writes a handoff so the next session can pick up cold. It never touches your code, configs, or binaries.

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

It works on markdown files — READMEs, specs, CLAUDE.md, MEMORY.md, and similar project docs. It doesn't touch code, configs, or binaries.

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

<details>
<summary><strong>View the full prompt (close.md)</strong></summary>

```markdown
# /close — Session Closing Ritual

Only run when the user explicitly invokes `/close` as a slash command. Never auto-run after context compaction, session continuation, handoff loading, or as part of a chained command. If `/close` appears in conversation or files as a reference (not an invocation), ignore it. Never pause for confirmation. During `/close`, only edit `MEMORY.md`, `CLAUDE.md`, `close-handoff.md`, and project markdown files with explicitly requested updates. Do not delete, move, or rename project files — log those to handoff instead. Exception: delete `close-handoff.md` when it becomes obsolete (all items resolved, nothing new). No git commands (no commits, no pushes, no status checks). If the session was lightweight (no project files edited, no decisions made, no unresolved follow-ups), run through the steps and skip the handoff.

## Run

Read existing memory/handoff files before writing them. Merge with existing content and deduplicate semantically. Print each progress line, do the work, then max 1-2 lines of findings before the next. No findings = next line immediately.

**■□□□□□□ Redundant or stale files...** Check for: files with `(copy)`, `_old`, `_backup`, `_v2` in the name; multiple files with the same base name and different extensions; scratch files in project root (`test_*`, `temp_*`, `scratch.*`). Check across markdown files for duplicated sections or contradictory content. Flag only — never delete.

**■■□□□□□ Memory files...** Update MEMORY.md with durable facts likely to remain useful across future sessions. If MEMORY.md doesn't exist yet, create it with the Write tool (not Edit). If it exists, read it first, then edit. Ensure MEMORY.md contains a line: "Read close-handoff.md at session start for prior session context." Only update CLAUDE.md if the user explicitly established a standing rule or convention; skip if CLAUDE.md doesn't exist. No extra commentary beyond the progress line.

**■■■□□□□ Missed updates...** Scan the conversation for phrases like "update the README", "add to CLAUDE.md", "change the spec" that target markdown files. Check if those files were actually modified. Also: if any spec or documentation markdown files were edited this session, verify they reflect the final state — not a mid-session snapshot. Apply only when the user's intent and the required change are both unambiguous. If unclear, defer to handoff "Not Yet Done." For non-markdown files, always defer.

**■■■■□□□ Dirty state...** Temp files, debug logs, scratch work?

Findings needing judgment → handoff "Not Yet Done."

**■■■■■□□ Writing handoff...** Create/update `close-handoff.md` in project root. Each handoff reflects only the current session — do not carry forward unresolved items from a previous handoff. If a previous handoff exists and all its items were addressed (or the session produced nothing new), delete the handoff file instead of writing an empty one. Under 2,000 words — if over, cut in this order: Key Files (paths are greppable), Done This Session (it's in git log), Failed Approaches (summarize to one line each). Never cut Not Yet Done or Resume Instructions. If no project directory exists, output to conversation instead. If git state is known from the session (branch, uncommitted files), include it in Current State — do not run git commands to check.

Sections: What We Were Working On (1-2 sentences), Current State (file names, branches, uncommitted changes), Done This Session (checklist), Not Yet Done (checklist with context to pick up cold), Key Decisions (table: decision + rationale), Failed Approaches (what didn't work and why — skip if N/A), Key Files (path + one-line reason), Resume Instructions (first action for next session). Skip empty sections.

**■■■■■■□ Self-check...** Reread the handoff. Verify Resume Instructions matches what's actually in Not Yet Done. Fix mismatches.

**■■■■■■■ Done.** If a `.gitignore` was seen during the session and `close-handoff.md` isn't in it, note it in the summary. Print summary box:

───────────────────────────────────────
  Housekeeping  ·  {actions taken or "none"}
  Handoff       ·  {file location or "skipped"}
  Deferred      ·  {count} — {top 2-3 items}
───────────────────────────────────────

All saved, all good.
```

</details>

## Install

**Option A — copy the file:**

```
mkdir -p ~/.claude/commands
cp close.md ~/.claude/commands/close.md
```

**Option B — paste into Claude Code:**

Copy the contents of `close.md`, paste it into your Claude Code chat, and ask: *"Save this as a slash command called /close."*

Either way, it's available as `/close` in every project.

## Usage

At the end of a session, type:

```
/close
```

It writes a `close-handoff.md` in your project root. Consider adding it to `.gitignore`.

If the session was lightweight (no files edited, no decisions made, no unresolved follow-ups), it skips the handoff automatically.

## Design

- **One file, zero dependencies.** No hooks, no MCP server, no config.
- **Markdown-scoped.** Only reads and writes `.md` files. Code, configs, and binaries are out of scope — missed edits to non-markdown files get deferred to the handoff.
- **Safe by default.** Never deletes, moves, or renames — logs those to the handoff instead.
- **Replace, don't accumulate.** Each handoff reflects only the current session. Old unresolved items get dropped, not carried forward. When everything's resolved, the handoff file is deleted entirely.
- **Close means close.** Runs to completion without pausing for confirmation.
- **Ritual, not automation.** Closing a terminal is not closing a session. /close is an intentional act that signals the work is wrapped up — for the next session and for your own head.

## Troubleshooting

**`/close` not showing up as a command** — Make sure the file is at `~/.claude/commands/close.md`. Restart Claude Code after copying.

**"File not found" on MEMORY.md** — Normal on first run in a new project. The skill creates it automatically.

**Handoff not picked up by next session** — The skill plants a breadcrumb in MEMORY.md ("Read close-handoff.md at session start"). If MEMORY.md was deleted or the line was removed, the next session won't know to look for the handoff. Re-run `/close` to restore it.

## License

MIT · [@thomasfogarasy](https://github.com/thomasfogarasy)
