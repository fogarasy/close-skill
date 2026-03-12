# /close — Session Closing Ritual
<!-- Author: @thomasfogarasy · MIT License -->

Never pause for confirmation. During `/close`, only edit `MEMORY.md`, `CLAUDE.md`, `close-handoff.md`, and project markdown files with explicitly requested updates. Do not delete, move, or rename anything — log those to handoff instead. If the session was lightweight (no project files edited, no decisions made, no unresolved follow-ups), run through the steps and skip the handoff.

## Run

Read existing memory/handoff files before writing them. Merge with existing content and deduplicate semantically. Print each progress line, do the work, then max 1-2 lines of findings before the next. No findings = next line immediately.

**■□□□□□□ Redundant or stale files...** Check for: files with `(copy)`, `_old`, `_backup`, `_v2` in the name; multiple files with the same base name and different extensions; scratch files in project root (`test_*`, `temp_*`, `scratch.*`). Check across markdown files for duplicated sections or contradictory content. Flag only — never delete.

**■■□□□□□ Memory files...** Update MEMORY.md with durable facts likely to remain useful across future sessions; create it if it doesn't exist. Ensure MEMORY.md contains a line: "Read close-handoff.md at session start for prior session context." Only update CLAUDE.md if the user explicitly established a standing rule or convention; skip if CLAUDE.md doesn't exist. No extra commentary beyond the progress line.

**■■■□□□□ Missed updates...** Scan the conversation for phrases like "update the README", "add to CLAUDE.md", "change the spec" that target markdown files. Check if those files were actually modified. Also: if any spec or documentation markdown files were edited this session, verify they reflect the final state — not a mid-session snapshot. Apply only when the user's intent and the required change are both unambiguous. If unclear, defer to handoff "Not Yet Done." For non-markdown files, always defer.

**■■■■□□□ Dirty state...** Temp files, debug logs, scratch work?

Findings needing judgment → handoff "Not Yet Done."

**■■■■■□□ Writing handoff...** Create/update `close-handoff.md` in project root. Replace previous handoff but carry forward unresolved items unless completed this session. Under 2,000 words — if over, cut in this order: Key Files (paths are greppable), Done This Session (it's in git log), Failed Approaches (summarize to one line each). Never cut Not Yet Done or Resume Instructions. Skip the handoff entirely if nothing meaningful happened. If no project directory exists, output to conversation instead. If in a git repo, include branch + uncommitted file count in Current State.

Sections: What We Were Working On (1-2 sentences), Current State (file names, branches, uncommitted changes), Done This Session (checklist), Not Yet Done (checklist with context to pick up cold), Key Decisions (table: decision + rationale), Failed Approaches (what didn't work and why — skip if N/A), Key Files (path + one-line reason), Resume Instructions (first action for next session). Skip empty sections.

**■■■■■■□ Self-check...** Reread the handoff. Verify Resume Instructions matches what's actually in Not Yet Done. Fix mismatches.

**■■■■■■■ Done.** If in a git repo and `close-handoff.md` isn't in `.gitignore`, note it in the summary. Print summary box:

```
───────────────────────────────────────
  Housekeeping  ·  {actions taken or "none"}
  Handoff       ·  {file location or "skipped"}
  Deferred      ·  {count} — {top 2-3 items}
───────────────────────────────────────

All saved, all good.
```
