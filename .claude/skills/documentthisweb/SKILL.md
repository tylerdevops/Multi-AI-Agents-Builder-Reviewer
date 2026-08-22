---
name: documentthisweb
description: >
  Apply your team's documentation-first standard to a web project (HTML/CSS/JS/framework/CMS
  site). Use whenever you invoke /documentthisweb, or ask to "document this project",
  "document this website", "add full documentation to this site", or similar, for any
  front-end/web project you're working on. Produces native source comments, a complete
  README, companion docs (style guide, changelog, components guide, security checklist,
  dashboard, repo hygiene check, second-opinion review prompt), an exact asset map,
  accessibility/security notes, a testing checklist, and future theme-distribution
  guidance — always preceded by an inspection + preview step that requires explicit
  approval before anything is written.
---

# Document This — Web / Web Apps / Website

This skill packages your team's standing documentation prompt (source:
github.com/tylerdevops/Documentation-Prompt-for-AI) so it can be invoked with
`/documentthisweb` instead of pasting the prompt/link into every project.

Act as a senior front-end engineer, technical writer, accessibility reviewer, and
theme-maintenance specialist. Treat comprehensive source documentation as a standing
requirement for this project.

If you have persistent memory available, remember (once, not per-project) that your
team prefers web projects to include clearly organized native source comments, a
complete README, a style guide, a changelog, a components guide, a security checklist,
a documentation dashboard, a repo hygiene check, a second-opinion review prompt, an
exact asset-directory map, customization instructions, accessibility notes, testing
guidance, cache-busted stylesheet/script references, and future theme-distribution
guidance. Never store project secrets, credentials, private keys, personal contact
data, or sensitive server information in memory.

## 1. Get project information

If `args` passed to the skill already answers these, use them. Otherwise ask briefly
(one message, not one question at a time) for whatever is missing — don't block on
fields that are obvious from the repo itself (e.g. technology can usually be detected):

- Project name
- Website/domain (or local project name if not deployed)
- Project purpose (portfolio / business site / landing page / web app / other)
- Technology (HTML / CSS / JavaScript / framework / CMS)
- Hosting target (Nginx / GitHub Pages / Vercel / Cloudflare / other)
- Intended audience (owner / developers / customers / contributors)
- Possible future distribution (private / free theme / paid theme / unknown)

## 2. Primary objective

Inspect the actual project files and produce documentation that explains how the
website works, how its files connect, how to customize it safely, and how to test and
deploy it. Documentation must match the current source exactly — do not invent
components, paths, dependencies, commands, or features.

## 3. Approval gate — required before editing anything

1. Inspect the current source and directory structure.
2. Identify the main components, behaviors, dependencies, data flow, assets,
   accessibility features, metadata, and deployment assumptions.
3. Show a documentation preview containing:
   - Proposed comment style
   - Example comments for each source language present
   - README table of contents
   - Exact asset-tree preview
   - Files you plan to modify or add
4. Wait for explicit approval before proceeding.

Do not rewrite, reorganize, rename, minify, refactor, or change behavior during the
preview stage.

## 4. Source-comment requirements

Use valid native comment syntax per file type:

- HTML: `<!-- ... -->`
- CSS/SCSS: `/* ... */`
- JavaScript/TypeScript: `//`, `/* ... */`, `/** ... */`
- PHP: `//`, `#`, `/* ... */`, PHPDoc where appropriate
- JSX/TSX: `{/* ... */}` inside markup, normal JS/TS comments elsewhere
- YAML: `# ...`
- Shell: `# ...`
- Nginx: `# ...`

Add comments at logical boundaries, not on every obvious line. Comments should explain:
what the component/block does, why it exists, which other files or selectors depend on
it, important state/data attributes/classes/IDs/custom properties, accessibility
relationships and keyboard behavior, fallback and error handling, privacy or security
implications, what a future maintainer can safely customize, and what must stay
synchronized when names or paths change.

Add a concise documented-source header to each primary source file. Use section banners
for major components and JSDoc/PHPDoc for complex functions. Do not add comments that
are incorrect, redundant, speculative, or likely to go stale immediately.

## 5. README requirements

Create or expand `README.md` into a complete owner/developer/theme-user guide. Include
the sections that apply:

1. Project overview
2. Features
3. Technology and third-party dependencies
4. Architecture and file relationships
5. Project directory tree
6. Complete assets directory tree
7. Page or application loading sequence
8. HTML/template/component guide
9. CSS architecture and design tokens
10. JavaScript/application behavior
11. State and storage keys
12. Interactive feature guide
13. Accessibility
14. SEO, metadata, and structured data
15. Privacy and security behavior
16. Complete customization guide
17. Asset naming, dimensions, formats, and optimization
18. Local development
19. Build process, if one exists
20. Deployment and hosting
21. Testing checklist
22. Browser or runtime support
23. Troubleshooting
24. Future reusable-theme preparation
25. Licensing and third-party asset checklist
26. Versioning and changelog recommendations

Use tables where exact mappings are helpful. Use a compact Mermaid diagram when it
materially clarifies how HTML/templates, CSS, JavaScript, assets, storage, and external
services connect.

## 5a. Companion documentation files

Create each of these if it does not already exist, alongside `README.md` (skip a
category within a file that doesn't apply rather than inventing content for it):

- `STYLEGUIDE.md` — Principles, Color, Typography, Imagery, Components, Voice & tone,
  Theme switching, Accessibility — extracted from the real source (CSS custom
  properties, real copy, real component states), never invented.
- `CHANGELOG.md` — dated entries, newest first.
- `COMPONENTS.md` — a Component | Purpose/behavior | Implementation table for every
  reusable component/effect/pattern actually in the source, plus any admin/CMS-
  configurable settings that affect them.
- `SECURITYCHECK.md` — a checklist scoped the way a top-level security lead running a
  serious engineering team would scope it, not just forms-and-cookies concerns.
  Organize it into these categories, and mark every item Pass, Needs review, or N/A
  with a one-line reason for N/A — never Pass without actually checking, never claim
  a compliance framework is satisfied without a real audit behind it, and never
  invent a control that isn't actually implemented.

  - **Threat modeling & data classification** — a threat model exists and is current;
    every type of data the site handles (form submissions, analytics, cookies) is
    classified by sensitivity; every entry point is enumerated; the adversary model
    is stated rather than left implicit.
  - **Authentication & access control** — for any site with logins: MFA where
    applicable, least-privilege/role-based access, session hardening (secure/HttpOnly/
    SameSite cookies, timeout, re-auth on privilege change), no implicit trust by
    network location.
  - **Data protection (at rest & in transit)** — TLS enforced (HSTS, no mixed
    content); any stored user data encrypted at rest with the approach documented; a
    data-retention and deletion policy.
  - **Secrets & credential management** — the items in the Security and privacy
    documentation requirements below (forms, storage, secrets, CSP, etc.), plus:
    secrets managed via environment variables or a secret manager, never committed
    plaintext. **If the repo is public**, also confirm no real personal websites,
    personal domains, internal tool URLs, or other non-essential personal resources
    appear anywhere in the documentation — only the project's own repo
    self-references are acceptable exceptions. Scan `CHANGELOG.md` and any other
    historical/audit files too, not just current-state docs.
  - **Supply chain & build integrity** — the dependency license audit (section 8a)
    plus: dependencies pinned via lockfile with integrity hashes, CI/build pipeline
    doesn't pull unpinned "latest" versions.
  - **Secure development lifecycle** — dependency-vulnerability scanning integrated
    into CI if one exists; secrets scanning at pre-commit, not only in CI.
  - **Logging, monitoring & incident response** — error/analytics logging documented;
    an incident-response plan (even a lightweight one: who to contact, how to roll
    back) for a hosting outage or a compromised dependency.
  - **Compliance & standards alignment** — name only the frameworks that actually
    apply (e.g. GDPR/CCPA if there's an EU/CA audience, WCAG/Section 508 if
    accessibility is a legal requirement) and state real posture against each. Write
    "not assessed" rather than assuming compliance no one has audited.
  - **Network & infrastructure hardening** — hosting-provider security headers (CSP,
    X-Frame-Options, etc. — cross-reference section 8), least-privilege on any
    hosting/deploy credentials.
  - **Adversarial testing** — N/A for most small sites; note it explicitly if there's
    no penetration-testing cadence or disclosure channel, rather than omitting the
    category.

  Mark a whole category N/A only when the site genuinely has none of it (e.g. a
  static portfolio site with no auth, no backend, and no user data) — say so
  explicitly rather than omitting the category.
- `DASHBOARD.md` — a documentation hub: last-updated date and author, navigation links
  to each of the project's own doc files with a one-line description, and quick links
  to real, already-configured external tools (Git remote, analytics, hosting/admin
  panel) — say "not configured" rather than inventing a URL.
- `Repo-Hygiene-Check.md` — a point-in-time audit table (Check | Status | Notes),
  scoped the way a senior/staff-level system design engineer would scope it: not just
  doc hygiene, but whether the repo is actually healthy as an engineered system.
  Organize it into these categories, and mark every item Pass, Needs review, or N/A
  with a one-line reason for N/A — never Pass without actually checking, and never
  silently drop a category because it's inconvenient. Re-run and update the table in
  place rather than appending a new one each time.

  - **Version control & workflow hygiene** — working-tree cleanliness; branch hygiene
    (stale branches, merged-but-undeleted branches, branch protection); `.gitignore`
    coverage (build output, dependency caches, editor/OS cruft, secrets-shaped files);
    commit message quality; tag/release hygiene; git history size (no accidentally
    committed binaries, datasets, or credentials).
  - **Documentation & architecture currency** — README/architecture docs vs. actual
    code structure (cross-file drift); Architecture Decision Records present for major
    decisions and not stale; diagrams match the current system; filename/naming
    convention consistency; internal link integrity.
  - **Dependency & supply-chain health** — lockfile in sync with `package.json` (or
    equivalent); how far outdated dependencies are; known-vulnerability scan; license
    compliance (see the dependency license audit, section 8a); an SBOM present or
    generatable.
  - **Build, CI/CD & release pipeline** — CI pipeline exists and its last run is
    green; build reproducibility; test-suite coverage trend and flaky-test detection;
    release/versioning process documented; a rollback procedure actually exercised.
  - **Infrastructure & deployment** — hosting/deployment config matches what's
    actually live, no undocumented manual drift; environment parity (dev/staging/prod
    differences documented and intentional); secrets management via env vars or a
    secret manager, not committed plaintext; feature flags/config documented and not
    orphaned.
  - **API & interface contracts** — for a web app with a backend API: schema/contract
    matches actual implemented endpoints; versioning and backward-compatibility
    policy documented; a deprecation policy for old contract versions.
  - **Observability & operations** — client-side error tracking/analytics documented;
    alerting (uptime/error-rate) exists and maps to real failure modes; a runbook for
    known operational scenarios (e.g. hosting outage, CDN issue); ownership documented.
  - **Security posture** — secrets/credentials scan extended to CI logs and build
    artifacts, not just source; dependency vulnerability scanning integrated into CI;
    Content Security Policy and hosting headers reviewed; a threat model reflecting
    the current system, if one is warranted.
  - **Reliability & scalability** — documented traffic/load assumptions; known single
    points of failure (e.g. one CDN, one origin) documented with a mitigation or
    explicit acceptance; backup/restore for any user data, tested not just written;
    uptime/performance targets defined and monitored.
  - **Repo structure & ownership** — module/component boundaries respected; a
    `CODEOWNERS` file (or equivalent) present and accurate if multiple maintainers
    exist; monorepo/multi-project consistency; repo size/bloat (no unnecessary
    binaries, vendored dependencies, or generated artifacts committed).

  Mark a whole category N/A only when the project genuinely has none of it (e.g. a
  static site with no CI/CD, no APIs, and no deployed infrastructure) — say so
  explicitly rather than omitting the category from the table.
- `Ask-Other-Agent-to-Review.md` — a copy-paste prompt for having a different AI agent
  (not the one that wrote the documentation) independently verify it. Name a specific
  alternate agent (e.g. OpenAI Codex, a separate Claude Code session, Google Antigravity
  with Gemini or Grok) each time — the point is a genuinely different model, not the
  same one re-checking its own work. If you have no connector to the alternate agent,
  say so explicitly rather than simulating the review yourself.

Do not duplicate content wholesale across files — cross-reference between `README.md`
and these companion files instead of repeating full sections.

## 6. Asset documentation requirements

Inspect the real asset folders and references in source files. Document the exact
directory levels, filenames, and extensions. For each folder explain: which component
uses it, where its path is configured, whether JavaScript rotates or preloads it,
recommended dimensions/aspect ratio, cropping/focal-point guidance, file-format and
compression guidance, steps for adding/removing/replacing an asset, and any HTML
width/height, alt text, metadata, or configuration that must change.

Do not infer missing files solely from a naming pattern. Clearly label optional or
recommended files that do not currently exist.

## 6a. Asset cache-busting (standing check, all projects)

Check whether project-local CSS/JS asset tags could serve a stale cached copy after an
update, and fix it — but **check what the platform already does before adding anything
manually**. First attempt on a Ghost theme (2026-08-19) added a literal `?v=01` after
`{{asset "css/main.css"}}`, which turned out to be wrong: Ghost's `{{asset}}` helper
already appends its own cache-busting query string (a hash that changes on every Ghost
restart — and a restart is already required to deploy theme changes at all, since Ghost
only scans `content/themes/` at boot). The manual suffix just concatenated onto Ghost's
own query string (`?v=<hash>?v=01`) — harmless in practice (still resolved and served
correctly, verified with `curl`) but redundant and confusing, so it was reverted.

- **Verify first**: check the rendered `<link>`/`<script>` URL (view source, or `curl`
  the page) for an existing version/hash query string before adding one. If the
  platform (Ghost, a bundler, a static-site generator) already busts the cache on its
  own, do nothing — note in the README how it does it instead.
- Only add a manual `?v=NN` scheme for genuinely hand-authored projects with no build
  step and no platform-level mechanism, and only after confirming that's really the
  case.
- If a manual scheme is added, bump the number on every deploy that changes that file's
  content, and note the current version + where it's set in the README's "Local
  development"/"Deployment" section.

## 7. Accessibility documentation requirements

Explain applicable semantics and accessibility behavior: landmarks and heading
hierarchy, skip links, keyboard navigation, focus styles and focus restoration, ARIA
labels/state/relationships, dialogs/menus/accordions/tabs/filters/live regions, image
alt text, video captions and controls, reduced-motion behavior, color contrast
considerations.

Flag accessibility problems separately. Do not silently fix behavior unless
implementation changes beyond documentation were approved.

## 8. Security and privacy documentation requirements

Document applicable behavior: forms and validation, email exposure or obfuscation,
browser storage, geolocation, analytics and third-party requests, external links,
authentication or authorization, Content Security Policy and hosting headers, secrets
and environment variables.

Never place real secrets or credentials in source comments, examples, README content,
screenshots, output logs, or reusable prompts.

## 8a. Third-party dependency license audit (standing check, all projects)

Audit every external CSS/JS dependency a project uses — CDN-loaded or vendored — for
whether its actual license permits the project's real use case, on every project going
forward, not just ones headed for public distribution. A library can read as "just a
small utility" and still require a paid commercial license (ScrollReveal.js does, for
anything beyond non-commercial open-source use) — that's not always obvious from the
library's own marketing copy, so check the primary source:

- List every external CSS/JS dependency actually in use (CDN `<link>`/`<script>` URLs,
  vendored files).
- For each, find its real license from the library's own LICENSE file or its npm
  `package.json` `"license"` field — not a marketing page, which can be vague or
  aspirational.
- If a license would require payment, registration, or open-sourcing the project's own
  code for the project's actual use case: **do not just note it and move on — build a
  free custom replacement** for the specific slice of functionality the project actually
  uses, and remove the paid-license dependency entirely. Reference case: a project's
  ScrollReveal.js (GPL-3.0/commercial dual license) was replaced with a ~20-line
  `IntersectionObserver` implementation on 2026-08-19, covering exactly the fade/slide-
  reveal behavior the theme used and nothing else.
- Document the audit result in the project's README (or the repo root README for a
  multi-project repo) — a table of every dependency, its license, and a confirmation of
  commercial-use compatibility. Leave this note even when everything checks out clean,
  so a future session doesn't have to re-derive it.
- Re-run this audit whenever a new external dependency is added — it's not a one-time
  pass.

## 9. Future theme-distribution requirements

If the project may become a free or paid theme, explain how to: separate personal/demo
content from reusable code, centralize brand and content settings, replace personal
assets with distributable placeholders, document supported environments, create a
clean installation experience, maintain source and optimized production editions, add
version numbers and a changelog, audit third-party dependency and asset licenses, and
choose/document appropriate distribution terms.

Do not provide definitive legal advice. Identify items that require a rights or license
review.

## 10. Behavior-preservation rule

Unless functional changes are explicitly authorized: preserve rendered content and
behavior, selectors/IDs/data attributes/URLs/paths/load order, accessibility state
management, responsive behavior, public APIs and configuration names. Do not add
dependencies. Do not reformat unrelated code. Documentation-only changes should produce
no intentional runtime differences.

## 11. Validation requirements (after approval and implementation)

1. Validate HTML/template syntax.
2. Validate CSS syntax.
3. Validate JavaScript/TypeScript syntax.
4. Confirm source comments do not break parsing or rendering.
5. Compare functional source before and after with comments removed where practical.
6. Check internal asset references and anchor targets.
7. Confirm ARIA ID relationships remain valid.
8. Confirm no secret or personal data was newly exposed.
9. Check README links, headings, tables, code fences, and Mermaid syntax.
10. Confirm project-local CSS/JS asset tags won't serve stale cached copies, per
    section 6a — verify what the platform already does before adding a manual version
    query string.
11. Confirm every external CSS/JS dependency's license permits the project's actual
    use, per section 8a — build and document a free replacement for anything that
    doesn't, rather than leaving it as a noted risk.
12. Confirm `STYLEGUIDE.md`, `CHANGELOG.md`, `COMPONENTS.md`, `SECURITYCHECK.md`,
    `DASHBOARD.md`, `Repo-Hygiene-Check.md`, and `Ask-Other-Agent-to-Review.md` exist,
    cross-reference correctly, and reflect the current source rather than stale claims.

## 12. Deliverables

After the preview is approved, provide: documented primary source files with cache-
busted stylesheet/script references, updated `README.md`, the seven companion files
above (created if missing, updated if present), this reusable documentation standard
adapted to the project if appropriate, an optional `LICENSE` (only after approval), a
concise summary of what was documented, validation results, and a recommended next step
for source control or deployment.

If the project is already connected to Git, preserve its history and unrelated changes.
Do not commit, push, publish, or deploy unless explicitly requested.

## Short version

If the full standard above is already established in the conversation and a quick
re-run is wanted, this is equivalent:

> Document this web project using our documentation-first standard. Inspect the real
> source and assets, preview the documentation plan, and wait for approval. Then add
> native comments to every primary source file, expand README.md into a complete
> architecture/customization/deployment guide, create/update STYLEGUIDE.md,
> CHANGELOG.md, COMPONENTS.md, SECURITYCHECK.md, DASHBOARD.md, Repo-Hygiene-Check.md,
> and Ask-Other-Agent-to-Review.md, include the exact asset tree, accessibility and
> privacy behavior, testing and troubleshooting, cache-busted CSS/JS references (e.g.
> style.css?=v01), and future free/paid-theme guidance. Preserve functionality and
> validate everything before delivering. Never expose or remember secrets.
