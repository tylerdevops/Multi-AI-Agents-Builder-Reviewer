# Changelog

This file is an append-only record of *why* things happened — not *what* changed (git history already has that). Each entry is written by the agent that did the work, at the end of its session, before handing off to the PM or the next agent.

## Rules

1. **Append only.** New entries go at the bottom. Never edit or delete a past entry — if something in an old entry turns out to be wrong, add a *new* entry that says so and links back to it.
2. **One entry per session/task, not per commit.** Commit messages already cover commit-level detail; this file is for the reasoning git doesn't capture — assumptions, alternatives ruled out, and open questions for the next agent.
3. **Read the last 10 entries by default** before starting work. Only look further back if the current task specifically needs older context (e.g., "why did we choose X" and X isn't in the recent entries).
4. **This is not a diff summary.** If an entry could be regenerated from `git log`, it's too thin — add the reasoning that made the diff happen.
5. **Archiving:** once this file passes ~30 entries, the PM (Codex) moves everything older than the most recent 15 into `changelog/archive/[YYYY-MM].md` and leaves a one-line pointer at the top of the Entries section (see template below). This keeps the file agents read on every session small.

## Entry format

Copy this block for each new entry:

```
## [YYYY-MM-DD] — [Agent] — [Task/feature name]
**Branch:** `feature/[name]` | **Commit(s):** `[short hash(es)]`
**Files touched:** `path/one`, `path/two`

**What/why:** [1–3 sentences — what was attempted and the reasoning, not a diff restatement]
**Decisions made:** [Any nontrivial choice and why, e.g. "used JWT over session cookies because X"]
**Ruled out:** [Approaches considered and rejected, and why — saves the next agent re-litigating]
**Open questions / left for next agent:** [Anything unresolved, deferred, or uncertain]
```

## Archive pointer

_(Populated by the PM once the first archive is created — e.g. "Entries before 2026-09 live in `changelog/archive/2026-08.md`.")_

---

## Entries

## 2026-08-18 — Claude Code — Repo scaffolding: add changelog system
**Branch:** `main` | **Commit(s):** `[fill in on commit]`
**Files touched:** `CHANGELOG.md`

**What/why:** Added this file so agents (Codex, Claude Code, Antigravity/Gemini) can hand off reasoning and context across sessions. The repo's Plan A/B/C workflow already separates builder and reviewer roles across sessions, but git history only shows *what* changed — not why a decision was made or what was tried and abandoned. This closes that gap without duplicating anything git already tracks.
**Decisions made:** Append-only, one entry per session rather than per commit — avoids merge conflicts when two agents touch the file around the same time, and keeps the log at "decision" granularity instead of "diff" granularity.
**Ruled out:** A structured JSON/YAML log — rejected because it's harder for an agent to skim for context mid-session and adds parsing overhead this repo's scale doesn't need.
**Open questions / left for next agent:** The 30-entry archive threshold is a guess — revisit once real usage shows how fast this file actually grows.
