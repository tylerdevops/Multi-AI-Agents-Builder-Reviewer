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

## 2026-09-26 — Claude Code — Introduce ATO 3.0 spec as the project's next version
**Branch:** `feature/ato-3-0-docs` | **Commit(s):** `[fill in on commit]`
**Files touched:** `README.md`, `ato-3.0/SPEC.md`, `ato-3.0/diagram.png`

**What/why:** Owner dropped `ATO_3.0_PLAIN_TEXT_SPEC.md` and `ATO_3.0.png` into the repo and asked to update the project with this new version. ATO 3.0 (AI Team Orchestrator) automates the same decision this repo's Plan A/B/C prompt makes manually — which model builds, which reviews — via a router (Jev) and a model reputation/three-strike engine. Moved both files into `ato-3.0/` and added a README section introducing ATO 3.0, linking to the spec and diagram, and explaining its relationship to the existing Plan A/B/C content.
**Decisions made:** Docs-first, in this repo. Owner chose to update documentation now and treat the actual Phase 1 MVP build (spec §45) as separate follow-up work, rather than starting implementation in the same pass. Kept the existing Plan A/B/C prompt intact rather than deleting it — it's still the active process until ATO 3.0 has a working MVP, and the README now says so explicitly. Chose a subfolder (`ato-3.0/`) over root-level files to keep the two generations of the concept visually separated.
**Ruled out:** Starting the Python implementation (router, classifier, reputation engine, CLI, tests from spec §45) in this same change — owner explicitly deferred that to a follow-up. Also ruled out replacing/deleting the Plan A/B/C prompt outright, since nothing about ATO 3.0 is built yet and the repo would otherwise be left with no working process.
**Open questions / left for next agent:** Whether the Phase 1 MVP (task records, classification, model registry, reputation scores, three-strike quarantine, JSONL/Markdown logging, Jev adapter interface, token/budget state, unit tests — per spec §45) gets built inside this repo (e.g. under a new `ato/` source tree per spec §42) or in a separate repository. Also unresolved: whether/when the Plan A/B/C prompt gets formally retired once ATO 3.0 has a working router.

## 2026-09-26 — Claude Code — Security/privacy pass, and restore diagram credit
**Branch:** `feature/ato-3-0-docs` | **Commit(s):** `[fill in on commit]`
**Files touched:** `SECURITYCHECK.md`, `.gitignore`, `ato-3.0/diagram.png`, `LICENSE`

**What/why:** Ran `/documentthisweb`, scoped down to just a security/privacy pass since this repo has no actual web-app source for the rest of that skill's checklist to apply to. Scanned all tracked files and git history for secrets, credentials, and personal-detail exposure; found none. Added `.gitignore` (repo had none — `.DS_Store` was untracked but unprotected against a future broad `git add`). Mid-pass, owner clarified that the "Built by Tyler \| ty1er.com" footer on `ato-3.0/diagram.png` — stripped in the previous session's "faceless repo" pass — should stay, since it's their own authorship credit on their own work, not a third-party personal detail or a secret. Restored it from the pre-strip commit (`3b090ed`) and updated `SECURITYCHECK.md`'s finding for that item from "fixed" to N/A with the reasoning.
**Decisions made:** Left the `LICENSE` copyright holder as `Multi-AI-Agents-Builder-Reviewer contributors` (changed from "Tyler" in the previous session) rather than reverting it too — the owner's request was specifically about the diagram credit, not the license, and this file distinguishes "owner's own attribution, kept intentionally" (diagram) from "no secrets/third-party personal data" (everything else) rather than treating "faceless" as one blanket rule.
**Ruled out:** Editing git commit author name/email to scrub the owner's identity from `git log` — that's account/VCS metadata, not repository content, and rewriting history wasn't requested.
**Open questions / left for next agent:** Confirm with the owner whether the `LICENSE` copyright holder should also go back to their name, now that we know the diagram credit was intentional and "faceless" was narrower in scope than first read.
