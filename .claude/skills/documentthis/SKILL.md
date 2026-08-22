---
name: documentthis
description: >
  Deprecated — superseded by /documentthisweb, /documentthisMacOS, and
  /documentthisWindows. Use whenever someone invokes /documentthis expecting the old
  unified web-documentation skill; redirect them to /documentthisweb (or the
  macOS/Windows variant if that's really the target), which carries the same standard
  plus the companion-doc requirements this file predates.
---

# Document This — deprecated, use /documentthisweb

This skill has been split into three platform-specific skills for easier long-term
maintenance:

- `/documentthisweb` — web projects (HTML/CSS/JS/framework/CMS)
- `/documentthisMacOS` — native macOS projects
- `/documentthisWindows` — native Windows projects

If invoked, tell the user this skill is deprecated and run `/documentthisweb` instead
(or the macOS/Windows variant, if that's the actual target) — it carries the same
documentation-first standard this file used to hold, plus the companion-document
requirements (style guide, changelog, components guide, security checklist, dashboard,
repo hygiene check, second-opinion review prompt) added since this file was split.

Source: github.com/tylerdevops/Documentation-Prompt-for-AI
