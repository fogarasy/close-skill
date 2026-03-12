# /close — Specification

## What it is

A session closing ritual for Claude Code. A single markdown file (`~/.claude/commands/close.md`) that runs as a slash command at the end of any work session. It cleans up project state and writes a handoff document so the next session can start cold.

## Problem

Born from a specific pain: specifying project context across multiple markdown files (CLAUDE.md, MEMORY.md, READMEs, specs), then watching Claude edit them organically throughout a conversation. By session end, it was unclear which files had actually been updated as discussed, whether contradictions had crept in between files, or whether the markdown landscape had accumulated redundant or stale content. The uncertainty compounded across sessions.

The general pattern:

- Markdown files drift out of sync as Claude edits them piecemeal during conversation
- Agreed-upon updates get lost — discussed but never applied
- Contradictory content accumulates across files that were edited at different points in the session
- No clear picture at session end of what the markdown state actually looks like
- Next session inherits the mess and adds to it

Existing handoff tools (12+ surveyed) solve context transfer but ignore project hygiene. They save session state and move on. The mess compounds.

## What it does

Seven-step ritual with an ASCII progress bar. Each step prints findings inline (max 1-2 lines), then advances. The whole thing runs in ~30 seconds.

| Step | Check | Action |
|------|-------|--------|
| ■□□□□□□ | Redundant or stale files | Concrete checklist: `_old`/`_backup`/`_v2` names, duplicate base names, scratch files, contradictory markdown content |
| ■■□□□□□ | Memory files | Update MEMORY.md with durable facts (create if absent); plant breadcrumb for handoff discovery; CLAUDE.md only for standing rules (skip if absent) |
| ■■■□□□□ | Missed updates | Scan conversation for update requests targeting markdown files, verify they were applied; verify edited spec/doc files reflect final state, not mid-session snapshot |
| ■■■■□□□ | Dirty state | Flag temp files, debug logs, scratch work |
| ■■■■■□□ | Handoff | Write/replace `close-handoff.md` with carry-forward; priority-based trimming if over 2k words |
| ■■■■■■□ | Self-check | Reread handoff, verify Resume Instructions matches Not Yet Done |
| ■■■■■■■ | Done | Print summary box; note if `close-handoff.md` not gitignored |

### Handoff schema

Converged from 12+ community implementations:

- What We Were Working On (1-2 sentences)
- Current State (files, branches, uncommitted changes)
- Done This Session (checklist)
- Not Yet Done (checklist with context)
- Key Decisions (table: decision + rationale)
- Failed Approaches (what not to retry)
- Key Files (path + one-line reason)
- Resume Instructions (first action for next session)

## Design principles

**Close means close.** No confirmation prompts, no new conversations. The progress bar runs to completion every time.

**Explicit write boundary.** Only `MEMORY.md`, `CLAUDE.md`, `close-handoff.md`, and project markdown files with explicitly requested updates. No source code, configs, or tests. No deletions, moves, or renames — those go to handoff.

**Read before write.** Merge with existing content. Deduplicate semantically. Never clobber.

**Replace with carry-forward.** The handoff overwrites the previous one, but unresolved items carry forward unless completed. Orientation, not audit trail.

**Durable facts only.** Memory updates are limited to facts likely to remain useful across future sessions. No hypotheses, temporary plans, or one-off observations.

**Lightweight skip.** If no project files were edited, no decisions made, and no unresolved follow-ups remain — skip the handoff. No empty ceremony.

**Zero dependencies.** One markdown file. No hooks, no MCP server, no config. Copy the file, it works.

## CLI visual language

The progress bar uses filled/empty Unicode squares:

```
■□□□□□□  Redundant or stale files...
■■□□□□□  Memory files...
■■■□□□□  Missed updates...
■■■■□□□  Dirty state...
■■■■■□□  Writing handoff...
■■■■■■□  Self-check...
■■■■■■■  Done.
```

Each stage prints 0-2 lines of findings between progress lines. The final step prints a summary box:

```
───────────────────────────────────────
  Housekeeping  ·  updated MEMORY.md
  Handoff       ·  close-handoff.md
  Deferred      ·  3 — article draft, skill name, link check
───────────────────────────────────────

All saved, all good.
```

## Development history

- **v1 (90 lines):** Full prose, examples, rationale. Written for humans.
- **v2 (48 lines, -47%):** Removed prose, collapsed to lists, killed step framing.
- **v3 (40 lines, -17%):** UX review — merged duplicate instructions, constrained scope, added fallbacks.
- **v4 (25 lines, -37%):** Merged progress bar steps, inline handoff template, added missed-updates check.
- **v5-v6:** Hook exploration (SessionEnd shell script) — built and removed. Platform-fragile, silent failures.
- **v7-v8 (24 lines):** Cross-model crit with ChatGPT. Named exact writable files, added deduplication rule, tightened memory criteria, resolved line 3/13 conflict.
- **v9:** External crit (Claude Opus acting as senior dev reviewer). Made stale-file check concrete with greppable patterns. Added conversation-scanning heuristic for missed updates. Added self-check step (verify handoff coherence). Priority-based word-limit trimming. Gitignore nudge. Attempted bash firework animation — doesn't work in Claude Code tool output (printf \r captured flat); replaced with closing sign-off line. Final: "All saved, all good."
- **v10:** Dual-expert review. Tightened apply/defer boundary (both intent and change must be unambiguous). Added first-run behavior (create MEMORY.md if absent, skip CLAUDE.md if absent). Fixed words vs tokens inconsistency. MEMORY.md breadcrumb for handoff discovery — next session auto-reads the handoff because MEMORY.md hints to do so.

Total compression: 90 → 24 lines through v8 (73%), then v9-v10 expanded to ~36 lines to reduce ambiguity in the steps that depend most on LLM judgment.

## File

```
~/.claude/commands/close.md
```

Global — available as `/close` in every Claude Code project.

## Handoff file

```
close-handoff.md
```

Namespaced to avoid collision. Lives in project root. Naming went through three iterations: `handoff.md` (too generic) → `.claude/handoff.md` (assumes directory exists) → `close-handoff.md` (no assumptions, no collisions).

## What it is not

- Not a logging tool. Use git log or dedicated session loggers for history.
- Not a hook. It's intentional — you choose to run it.
- Not an MCP server. No infrastructure, no setup beyond one file.
- Not a replacement for MEMORY.md or CLAUDE.md. It writes to them; it doesn't replace their purpose.

## Prior art

Handoff tools surveyed: Sonovore/claude-code-handoff, willseltzer/claude-handoff, GGPrompts /wipe, who96/claude-code-context-handoff, ContextVault, Black Dog Labs. Session-closing tools: AccidentalRebel /retrospective, Tmeister /project:session-end, affaan-m SessionEnd hook, cosgroveb/claude-session-summary, claudefa.st session lifecycle hooks.

Key borrowings: Failed Approaches section (willseltzer), Key Decisions with rationale (GGPrompts), orientation cap inspired by Black Dog Labs (theirs: 2,000 tokens; ours: 2,000 words — more intuitive for LLM self-judgment), converged handoff schema (community consensus).
