# /close

A tiny session-closing ritual for [Claude Code](https://github.com/anthropics/claude-code). One markdown file — too small for a package, too useful to lose in a gist.

Run it when you're done working in the Claude CLI. It cleans up your project's markdown files, catches missed updates, and writes a handoff so the next session can pick up cold. It never touches your code, configs, or binaries.

![/close in action](https://github.com/fogarasy/close-skill/releases/download/v1.0.0/Claude-Close-Skill-Workflow.gif)

## Why

Closing a terminal is not closing a session.

Claude Code sessions are stateless. Without a deliberate wrap-up, markdown files drift out of sync, agreed-upon edits get lost, and the next session inherits the mess. Existing handoff tools save session state but ignore project hygiene. /close does both — an AI agent that turns session cleanup into a repeatable LLM workflow.

/close is that signal — a small UX ritual that lets you stop thinking about the project.

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

It works on markdown files — READMEs, specs, CLAUDE.md, MEMORY.md, and similar project docs. It doesn't touch code, configs, or binaries. A focused agentic workflow for project hygiene.

Seven checks, one slash command, ~30 seconds:

1. **Stale files**
   Flags `_old`, `_backup`, `_v2` duplicates or contradictory docs.

2. **Memory**
   Updates `MEMORY.md` with durable facts. Plants a breadcrumb so the next session picks up the handoff.

3. **Missed updates**
   Detects discussed edits never applied to specs, plus unapplied plans and action items.

4. **Dirty state**
   Flags temp files, debug logs, scratch work.

5. **Handoff**
   Writes `close-handoff.md` with what's done, what's left, and how to resume.

6. **Self-check**
   Verifies the handoff is internally consistent.

7. **Summary**
   Prints the box above.

Anything needing human judgment gets deferred to the handoff, not acted on.

## Install

**Option A — copy the file:**

```
mkdir -p ~/.claude/commands
cp close.md ~/.claude/commands/close.md
```

**Option B — paste into Claude Code:**

Copy the contents of `close.md`, paste it into your Claude Code chat, and ask: *"Save this as a slash command called /close."*

Either way, it's available as `/close` in every project.

## Permissions

/close edits markdown files (MEMORY.md, close-handoff.md). Claude Code asks for permission on each edit by default. To avoid prompts, tell Claude:

> Add `Edit(*.md)` and `Write(*.md)` to my global settings.

This allows Claude to edit and create markdown files without asking — one-time setup, applies across all projects.

## Usage

At the end of a session, type:

```
/close
```

It writes a `close-handoff.md` in your project root. Consider adding it to `.gitignore`.

If the session was lightweight (no files edited, no decisions made, no unresolved follow-ups), it skips the handoff automatically.

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

## Boundaries

- Only run when the user explicitly invokes `/close` as a slash command
- Never auto-run after context compaction, session continuation, handoff loading, or chained commands
- If `/close` appears in conversation or files as a reference (not an invocation), ignore it
- Never pause for confirmation
- Only edit `MEMORY.md`, `CLAUDE.md`, `close-handoff.md`, and project markdown files with explicitly requested updates
- No git commands (no commits, no pushes, no status checks)
- No Bash tool for file operations — use Write to create files and Edit to modify them
- Do not delete, move, or rename project files — log those to handoff instead
- If a previous `close-handoff.md` exists, overwrite it — use Write to replace its contents (never use Bash to delete)

## Run

Read existing memory/handoff files before writing them — for context only, not as a source of items to copy forward. Merge memory content and deduplicate semantically. Print each progress line, do the work, then max 1-2 lines of findings before the next. No findings = next line immediately.

**■□□□□□□ Redundant or stale files...** Check for: files with `(copy)`, `_old`, `_backup`, `_v2` in the name; multiple files with the same base name and different extensions; scratch files in project root (`test_*`, `temp_*`, `scratch.*`). Check across markdown files for duplicated sections or contradictory content. Flag only — never delete.

**■■□□□□□ Memory files...** Update MEMORY.md with durable facts likely to remain useful across future sessions. If MEMORY.md doesn't exist yet, create it with the Write tool (not Edit, not Bash) — Write creates parent directories automatically. If it exists, read it first, then edit. Ensure MEMORY.md contains a line: "Read close-handoff.md at session start for prior session context." Only update CLAUDE.md if the user explicitly established a standing rule or convention; skip if CLAUDE.md doesn't exist. No extra commentary beyond the progress line.

**■■■□□□□ Missed updates...** Scan the conversation for phrases like "update the README", "add to CLAUDE.md", "change the spec" that target markdown files. Check if those files were actually modified. Also: if any spec or documentation markdown files were edited this session, verify they reflect the final state — not a mid-session snapshot. Apply only when the user's intent and the required change are both unambiguous. If unclear, defer to handoff "Not Yet Done." For non-markdown files, always defer.

Also scan for numbered lists, bullet lists, or explicitly proposed plans that were discussed but not executed — items the user approved or left open go into "Not Yet Done"; items the user rejected or superseded are omitted.

**■■■■□□□ Dirty state...** Temp files, debug logs, scratch work?

Findings needing judgment → handoff "Not Yet Done."

**■■■■■□□ Writing handoff...** The handoff covers only this session. Never copy "Not Yet Done" items from a previous handoff — those belong to the past session. If this session produced no new unresolved items, overwrite `close-handoff.md` with: "No open items. This file will be replaced on next /close."

Create/update `close-handoff.md` in project root. Under 2,000 words — if over, cut in this order: Key Files (paths are greppable), Done This Session (it's in git log), Failed Approaches (summarize to one line each). Never cut Not Yet Done or Resume Instructions. If no project directory exists, output to conversation instead. If git state is known from the session (branch, uncommitted files), include it in Current State — do not run git commands to check.

Sections: What We Were Working On (1-2 sentences), Current State (file names, branches, uncommitted changes), Done This Session (checklist), Not Yet Done (checklist with context to pick up cold), Key Decisions (table: decision + rationale), Failed Approaches (what didn't work and why — skip if N/A), Key Files (path + one-line reason), Resume Instructions (first action for next session). Skip empty sections.

**■■■■■■□ Self-check...** Reread the handoff. Verify Resume Instructions matches what's actually in Not Yet Done. Fix mismatches.

**■■■■■■■ Done.** Print summary box:

───────────────────────────────────────
  Housekeeping  ·  {actions taken or "none"}
  Handoff       ·  {file location or "skipped"}
  Deferred      ·  {count} — {top 2-3 items}
───────────────────────────────────────

All saved, all good.
```

</details>

## Design principles

- **One file, zero dependencies**
  No hooks, no servers, no configuration. Just developer productivity.

- **Markdown-scoped**
  Only reads and writes `.md` files.

- **Safe by default**
  Never deletes or renames files.

- **Replace, don't accumulate**
  Each session writes a fresh handoff.

- **Ritual, not automation**
  `/close` is an intentional act that signals the work is done.

## Customize

It's one markdown file — fork it and make it yours. You can modify:

- Which files get scanned and updated (e.g. add `CHANGELOG.md`, drop `CLAUDE.md`)
- Handoff format and sections (add your own, remove what you don't use)
- What counts as dirty state (e.g. flag `.env.local`, ignore `node_modules`)
- Progress steps (add a linting check, remove the self-check)

## Troubleshooting

**`/close` not showing up as a command** — Make sure the file is at `~/.claude/commands/close.md`. Restart Claude Code after copying.

**"File not found" on MEMORY.md** — Normal on first run in a new project. The skill creates it automatically.

**Handoff not picked up by next session** — The skill plants a breadcrumb in MEMORY.md ("Read close-handoff.md at session start"). If MEMORY.md was deleted or the line was removed, the next session won't know to look for the handoff. Re-run `/close` to restore it.

## License

MIT · [@thomasfogarasy](https://github.com/thomasfogarasy)

Built for the Claude Code slash command ecosystem. A small step toward AI productivity that compounds over sessions.
