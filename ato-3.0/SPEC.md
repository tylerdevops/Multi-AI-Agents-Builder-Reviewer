# ATO 3.0 — Adaptive Task Orchestrator
## Plain-Text Markdown Specification

**Version:** 3.0  
**Purpose:** A text-only handoff document for Claude Code, OpenAI Codex, ChatGPT Chat, Grok, and other AI agents.

---

## 1. What ATO Means

**ATO = Adaptive Task Orchestrator**

ATO is a routing and reputation layer that decides which AI model should handle a task, whether another model should review it, and how future routing should change based on real outcomes.

The core idea is simple:

```text
Task
  -> classify
  -> check model reputation
  -> check token/budget state
  -> filter ineligible models
  -> Jev chooses among eligible models
  -> selected model performs work
  -> tests/review/human feedback
  -> reputation updated
  -> future routing changes
```

Jev is the fast decision layer.

ATO is the memory, policy, scoring, and audit layer.

---

## 2. Goals

ATO 3.0 should:

1. Reduce unnecessary Claude/Codex token usage.
2. Avoid sending every task to the most expensive model.
3. Prevent Jev from repeatedly recommending a model that performs poorly.
4. Track model performance by task category and role.
5. Keep human review as the strongest quality signal.
6. Use GitHub activity as supporting evidence.
7. Maintain transparent routing and reputation logs.
8. Adapt when weekly Claude/Codex usage is low.
9. Support independent cross-model review for higher-risk work.
10. Keep routing explainable.

---

## 3. Providers

ATO 3.0 uses:

- Jev
- Claude Code
- OpenAI Codex
- ChatGPT Chat
- Grok

Local Qwen/Ollama is intentionally excluded.

### Jev

Use Jev for:

- cheap/standard/deep classification
- model selection from an eligible list
- review-needed decisions
- skill selection
- escalation decisions

Jev should not:

- store model reputation
- read full repositories directly
- receive secrets
- receive full logs when a compact summary is enough

### Claude Code

Use Claude Code for:

- planning
- architecture
- implementation
- debugging
- refactoring
- review
- complex repository work

### OpenAI Codex

Use Codex for:

- implementation
- debugging
- independent code review
- repository analysis
- test-oriented fixes
- cross-review of Claude work

### ChatGPT Chat

Treat ChatGPT Chat as a manual lane.

ATO should generate a self-contained Markdown review packet that can be pasted or uploaded into ChatGPT Chat when:

- Claude/Codex usage is low
- a manual second opinion is desired
- architecture/review can be performed without direct repo automation

### Grok

Use Grok as:

- an automated provider if a supported CLI/API is configured
- a manual reviewer/researcher otherwise

Possible roles:

- browser QA
- research
- alternate code review
- bug reproduction
- extra coding capacity

---

## 4. Core Architecture

```text
                     USER TASK
                         |
                         v
                ATO TASK CLASSIFIER
                         |
           category / role / risk
                         |
                         v
              MODEL REPUTATION ENGINE
                         |
                  eligible models
                         |
                         v
                TOKEN / BUDGET STATE
                         |
                         v
                    JEV ROUTER
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
       CLAUDE          CODEX           GROK
          |              |              |
          +--------------+--------------+
                         |
                         v
                    WORK PRODUCT
                         |
                         v
               TESTS / LINT / BUILD
                         |
                         v
                INDEPENDENT REVIEW
                         |
                         v
                  HUMAN / GITHUB
                         |
                         v
                   OUTCOME ENGINE
                         |
             +-----------+-----------+
             |                       |
             v                       v
      MACHINE-READABLE LOGS     MARKDOWN LOGS
             |                       |
             +-----------+-----------+
                         |
                         v
                FUTURE ROUTING UPDATE
```

---

## 5. Important Routing Rule

ATO filters model eligibility before Jev makes a selection.

Wrong:

```text
Jev selects Model A
  -> ATO discovers Model A is quarantined
```

Correct:

```text
Task
  -> ATO checks category/role reputation
  -> ATO removes quarantined or disabled models
  -> Jev only sees eligible choices
  -> Jev selects from those choices
```

This prevents Jev from repeatedly choosing a model that has already lost trust.

---

## 6. Task Classification

Every task should have a structured record.

Example:

```json
{
  "task_id": "ATO-0184",
  "project": "example-project",
  "title": "Fix OAuth session handling",
  "category": "security",
  "subcategory": "authentication",
  "role": "builder",
  "risk": "high",
  "complexity": "high",
  "repo_required": true,
  "browser_required": false,
  "human_review_required": true
}
```

Suggested categories:

- frontend
- backend
- security
- architecture
- database
- devops
- testing
- debugging
- documentation
- research
- browser-qa
- design
- performance
- api
- mobile
- data

Suggested roles:

- planner
- builder
- reviewer
- debugger
- researcher
- qa
- architect

---

## 7. Model Reputation Engine

Each model should have:

- overall score
- role-specific score
- category-specific score
- human strikes
- automated failure history
- human failure history
- status
- model version
- sample count
- confidence

Example:

```json
{
  "provider": "openai",
  "model": "codex-example",
  "version": "2026-09",
  "overall_score": 927,
  "status": "active",
  "roles": {
    "builder": {
      "score": 945,
      "samples": 21
    },
    "reviewer": {
      "score": 960,
      "samples": 17
    }
  },
  "categories": {
    "frontend": {
      "score": 875,
      "human_strikes": 0
    },
    "backend": {
      "score": 952,
      "human_strikes": 0
    },
    "security": {
      "score": 966,
      "human_strikes": 0
    },
    "architecture": {
      "score": 910,
      "human_strikes": 1
    }
  }
}
```

---

## 8. Score Scale

Recommended scale:

```text
0 - 1000
```

Suggested new-model starting score:

```text
750
```

Do not initialize every unknown model at 1000.

ATO should consider:

- score
- sample count
- confidence
- human strikes
- recency
- model version

---

## 9. Human Review Weight

Human feedback should have more influence than automated AI feedback.

Suggested weights:

| Event | Score Change |
|---|---:|
| Human says excellent | +50 |
| Human approves | +25 |
| Independent AI approves | +10 |
| Automated tests pass | +5 |
| Minor AI finding | -25 |
| Implementation-caused test failure | -50 |
| Meaningful reviewer defect | -75 |
| PR changes requested for implementation defect | -100 |
| Regression traced to implementation | -100 |
| Human rejects delivery | -300 + strike |
| Human identifies critical/security failure | -500 + strike |

These values should be configurable.

---

## 10. Three-Strike Policy

ATO should support a role/category-specific strike policy.

```text
0 strikes
  -> ACTIVE

1 strike
  -> ACTIVE
  -> reduced priority

2 strikes
  -> DEGRADED
  -> prefer alternatives

3 strikes
  -> QUARANTINED
  -> remove from normal routing for that role/category
```

Example:

```text
Model A / Security Builder

1000
 -> human fail
700
 -> strike 1

700
 -> human fail
400
 -> strike 2
 -> DEGRADED

400
 -> human fail
100
 -> strike 3
 -> QUARANTINED
```

Three failures in one category should not automatically eliminate the model globally.

Example:

```text
Grok

Research:
  score 940
  strikes 0
  ACTIVE

Frontend:
  score 910
  strikes 0
  ACTIVE

Security:
  score 130
  strikes 3
  QUARANTINED
```

---

## 11. Model Statuses

Supported states:

```text
ACTIVE
DEGRADED
QUARANTINED
PROBATION
RETIRED
DISABLED
```

Lifecycle:

```text
ACTIVE
  |
  | strike 1
  v
ACTIVE / reduced priority
  |
  | strike 2
  v
DEGRADED
  |
  | strike 3
  v
QUARANTINED
  |
  | new version or manual retry
  v
PROBATION
  |
  | successful reviewed tasks
  v
ACTIVE
```

---

## 12. Probation

Recommended probation rule:

```text
Quarantined model/version
  -> new version or manual retry
  -> PROBATION
  -> only low/medium-risk tasks
  -> mandatory review
  -> no critical security work
  -> 5 successful human-approved tasks
  -> ACTIVE
```

The pass count should be configurable.

---

## 13. Separate Builder and Reviewer Reputation

Do not give one model a single universal score.

Example:

```text
Claude

Builder: 920
Reviewer: 860
Planner: 960
Architect: 970
```

If Claude builds a feature and Codex approves it, but a real regression is later found:

```text
Claude builder/backend score:
  penalty

Codex reviewer/backend score:
  penalty
```

This allows ATO to learn that a model may be better at one role than another.

---

## 14. GitHub Evidence

ATO should collect GitHub evidence before asking Jev to classify outcomes.

Possible inputs:

- issues
- pull requests
- PR review comments
- requested changes
- CI failures
- test failures
- bug-fix commits
- revert commits
- issue reopen events
- regression labels
- release failures
- linked task IDs
- commit history

Flow:

```text
GitHub
  -> ATO Evidence Collector
  -> normalize
  -> remove noise
  -> redact secrets
  -> link task IDs
  -> structured evidence
  -> outcome/reputation logic
```

---

## 15. GitHub Evidence Must Be Cautious

Do not subtract -300 merely because CI failed.

CI can fail because of:

- flaky tests
- infrastructure issues
- unrelated changes
- dependency outages
- bad fixtures
- external services

Recommended flow:

```text
GitHub event
  -> deterministic collection
  -> attribution/classification
  -> unrelated or uncertain: log only
  -> likely implementation issue: small/medium penalty
  -> human-confirmed failure: strong penalty + strike
```

Human review remains authoritative.

---

## 16. Task-to-Commit Linking

Each ATO task should record execution metadata.

Example:

```json
{
  "task_id": "ATO-0184",
  "provider": "anthropic",
  "model": "claude-example",
  "role": "builder",
  "category": "backend",
  "repo": "owner/repo",
  "branch": "ato/0184-oauth-fix",
  "commit": "abc123",
  "pull_request": 82
}
```

If a later bug is linked to PR #82, ATO can connect the regression back to:

- task
- builder
- reviewer
- category
- model version

---

## 17. Outcome Engine

Each task should end with a structured outcome.

Example:

```json
{
  "task_id": "ATO-0184",
  "outcome": "failed_human_review",
  "severity": "high",
  "attribution_confidence": 0.98,
  "builder": "claude-example",
  "reviewer": "codex-example",
  "tests_passed": false,
  "human_verdict": "fail",
  "notes": [
    "Session cookie not rotated",
    "OAuth failure redirect incorrect"
  ]
}
```

Supported outcomes:

```text
pass
pass_with_minor_changes
ai_review_failed
automated_test_failed
human_review_failed
regression
critical_failure
inconclusive
cancelled
```

---

## 18. Logging

ATO should keep both machine-readable and human-readable records.

Recommended structure:

```text
.at/
├── config/
│   ├── models.yaml
│   ├── scoring.yaml
│   ├── routing.yaml
│   └── budgets.yaml
│
├── intelligence/
│   ├── models.json
│   ├── task-history.jsonl
│   ├── reputation-events.jsonl
│   ├── routing-history.jsonl
│   └── github-events.jsonl
│
├── logs/
│   ├── MODEL_PERFORMANCE.md
│   ├── ROUTING_DECISIONS.md
│   ├── HUMAN_REVIEWS.md
│   └── ATO_MODEL_REPORT.md
│
├── handoffs/
│   ├── chatgpt/
│   ├── grok/
│   ├── claude/
│   └── codex/
│
└── state/
    ├── budgets.json
    └── active-task.json
```

---

## 19. MODEL_PERFORMANCE.md

Example:

```markdown
# Model Performance

| Model | Builder | Reviewer | Backend | Security | Strikes | Status |
|---|---:|---:|---:|---:|---:|---|
| Claude | 930 | 880 | 920 | 900 | 0 | ACTIVE |
| Codex | 955 | 950 | 970 | 960 | 0 | ACTIVE |
| Grok | 810 | 860 | 790 | 650 | 2 | DEGRADED |
```

---

## 20. ROUTING_DECISIONS.md

Record not just the choice, but the reason.

Example:

```markdown
## ATO-0184

Task:
Fix OAuth callback/session handling

Category:
Security / Backend

Risk:
High

Eligible:
- Claude
- Codex

Excluded:
- Grok — security category quarantined

Budget:
- Claude: HEALTHY
- Codex: LOW

Jev Choice:
Claude

Reason Inputs:
- Claude security score: 900
- Claude backend score: 920
- Claude usage: HEALTHY
- Codex usage: LOW
- Claude human strikes: 0

Review Policy:
Cross-review required
```

---

## 21. HUMAN_REVIEWS.md

Example:

```markdown
## ATO-0184 Human Review

Verdict:
FAIL

Severity:
HIGH

Findings:
- Session cookie not rotated
- OAuth failure redirect incorrect

Penalty:
-300

Strike:
1

Affected Reputation:
Claude / Builder / Security

Status After Update:
ACTIVE / reduced priority
```

---

## 22. ATO_MODEL_REPORT.md

Generate a periodic summary containing:

- model grades
- role grades
- category grades
- sample counts
- human strikes
- recent failures
- recoveries
- quarantined models
- probation models
- provider usage
- routing counts
- pass/fail rates
- human-review failure rate
- token state
- routing changes
- strongest categories
- weakest categories

---

## 23. Token/Budget Awareness

Suggested states:

```text
healthy
medium
low
critical
exhausted
disabled
unknown
```

Example:

```json
{
  "claude": "healthy",
  "codex": "low",
  "chatgpt_chat": "healthy",
  "grok": "healthy"
}
```

Jev can use this information when choosing among otherwise-qualified options.

High-risk quality rules should override token savings.

---

## 24. Operating Modes

### full-auto

```text
Jev
  -> builder
  -> tests
  -> independent review if needed
  -> outcome
  -> reputation update
```

### claude-first

```text
Jev
  -> Claude Code
  -> verification
  -> Codex/Grok review if needed
  -> manual ChatGPT fallback if needed
```

### codex-first

```text
Jev
  -> Codex
  -> verification
  -> Claude/Grok review if needed
  -> manual ChatGPT fallback
```

### cross-review

```text
Model A builds
  -> Model B reviews
  -> fixes
  -> fresh Model B inspection
```

For high-risk work:

```text
Claude builds -> Codex grades
Codex builds  -> Claude grades
```

### conserve-tokens

Use when Claude/Codex weekly allocation is low.

Possible routing:

```text
small/low-risk:
  cheaper eligible provider

research/review:
  Grok or manual ChatGPT

critical repo work:
  preserve strongest remaining coding allocation
```

### manual

```text
ATO evaluates
  -> generates handoff packet
  -> human sends to provider
  -> human returns result
  -> ATO logs outcome
```

---

## 25. Review Escalation

Do not use multiple models for every trivial task.

Low-risk example:

```text
Change button padding
  -> single qualified builder
  -> tests
  -> done
```

High-risk example:

```text
Rewrite authentication middleware
  -> classify security/high-risk
  -> reputation filtering
  -> Jev route
  -> builder
  -> tests
  -> independent reviewer
  -> fixes
  -> fresh final inspection
  -> human review
  -> reputation update
```

---

## 26. Retry Policy

Avoid infinite loops.

Recommended:

```text
Attempt 1:
  same model may fix a minor issue

Attempt 2:
  repeated/meaningful failure forces alternate strategy or reviewer

Attempt 3:
  route to another qualified provider

Three human failures:
  role/category quarantine
```

---

## 27. Fresh Inspection Rule

For important work:

```text
Builder
  -> tests
  -> reviewer
  -> fixes
  -> fresh final inspection
  -> human approval
```

Any material code change after final inspection invalidates that inspection for high-risk work.

---

## 28. Recommended Skills

```text
at-route
at-classify
at-budget
at-reputation
at-review
at-loop
at-outcome
at-github-evidence
at-human-review
at-report
at-chatgpt-handoff
at-grok-handoff
at-quarantine
at-probation
```

---

## 29. Jev Routing Contract

Jev should receive only eligible models.

Example:

```json
{
  "task": {
    "category": "security",
    "role": "builder",
    "risk": "high"
  },
  "eligible_models": [
    {
      "id": "claude",
      "category_score": 900,
      "role_score": 930,
      "status": "active",
      "budget": "healthy"
    },
    {
      "id": "codex",
      "category_score": 960,
      "role_score": 955,
      "status": "active",
      "budget": "low"
    }
  ]
}
```

Quarantined options should not be shown to Jev during normal routing.

---

## 30. Routing Factors

ATO/Jev can consider:

- task fit
- category score
- role score
- overall score
- human strikes
- status
- sample count
- confidence
- risk
- token availability
- cost
- speed
- tool requirements
- repo access
- browser requirement
- provider availability
- review requirement

High-risk work should favor quality and reliability over cost.

---

## 31. Sample Conceptual Routing Formula

```text
routing_score =
    category_fit       * 0.30
  + role_fit           * 0.20
  + historical_quality * 0.20
  + reliability        * 0.10
  + budget_health      * 0.08
  + cost_efficiency    * 0.05
  + tool_fit           * 0.05
  + speed              * 0.02
```

This is only a starting concept.

ATO should tune weights based on real results.

---

## 32. Confidence and Sample Size

Do not over-trust a model with one successful sample.

Example:

```text
Model A
Security score: 980
Samples: 1

Model B
Security score: 930
Samples: 42
```

ATO should account for sample count/confidence before assuming Model A is better.

---

## 33. Recency

Future versions may reduce the influence of old results.

Possible policy:

```text
0-30 days: 100% weight
31-90 days: 75%
91-180 days: 50%
older: historical reference
```

Do not overcomplicate the MVP with decay until basic scoring works reliably.

---

## 34. Model Versioning

Track versions separately.

Example:

```text
claude-sonnet-x@2026-09
claude-opus-x@2026-09
codex-x@2026-09
grok-x@2026-09
```

A major new version should enter as a new reputation record or probation candidate rather than silently inheriting every old score.

---

## 35. Manual ChatGPT Handoff

Generate:

```text
.at/handoffs/chatgpt/ATO-0184-review.md
```

Include:

- task
- goal
- acceptance criteria
- relevant files
- diff
- tests run
- test output
- builder identity
- known risks
- review rubric
- requested output format

Keep it self-contained.

---

## 36. Transparency Requirements

ATO should be able to answer:

```text
Why did ATO choose Claude?
Why was Grok excluded?
What caused Codex's score to drop?
Which tasks caused Grok's security quarantine?
Which reviewer approved a later regression?
What were the last 10 human failures?
```

No hidden reputation changes.

---

## 37. Example End-to-End Flow

Task:

```text
Fix OAuth callback/session handling
```

Flow:

```text
1. Category:
   security/backend

2. Role:
   builder

3. Risk:
   high

4. Reputation filter:
   Claude = eligible
   Codex = eligible
   Grok = quarantined for security

5. Budget:
   Claude = healthy
   Codex = low

6. Jev:
   selects Claude

7. Claude:
   builds

8. Tests:
   one failure

9. Codex:
   reviews and identifies session-rotation defect

10. Claude:
    fixes

11. Fresh inspection:
    passes

12. Human:
    PASS

13. Reputation:
    Claude security builder +25
    Codex security reviewer +25

14. Logs updated.
```

---

## 38. Three-Strikes Example

```text
Security Task 1
  -> Model A
  -> Human FAIL
  -> -300
  -> Strike 1

Security Task 2
  -> Model A
  -> Human FAIL
  -> -300
  -> Strike 2
  -> DEGRADED

Security Task 3
  -> Model A
  -> Human FAIL
  -> -300
  -> Strike 3
  -> QUARANTINED

Next Security Task
  -> ATO removes Model A before routing
  -> Jev never sees Model A
```

This is a central ATO 3.0 requirement.

---

## 39. Implementation Phases

### Phase 1 — Logging MVP

Build:

- task IDs
- task classification
- routing decision log
- structured task history
- Markdown audit files

### Phase 2 — Reputation Engine

Build:

- 0-1000 scores
- role scores
- category scores
- human strikes
- status lifecycle
- reputation events
- manual human review command

### Phase 3 — Jev Routing

Build:

- eligibility filtering
- Jev selection
- routing explanation
- budget-aware routing
- fallback behavior

### Phase 4 — Automated Review Loop

Build:

- builder/reviewer separation
- cross-provider review
- bounded retry loop
- fresh inspection
- test/proof commands

### Phase 5 — GitHub Intelligence

Build:

- issue collection
- PR collection
- CI evidence
- review comments
- commit mapping
- regression evidence
- attribution confidence

### Phase 6 — Analytics

Build:

- ATO_MODEL_REPORT.md
- pass rates
- failure rates
- strikes
- quarantines
- provider usage
- category strengths
- reviewer accuracy
- routing history

### Phase 7 — Advanced Learning

Possible later additions:

- recency decay
- confidence weighting
- sample-size normalization
- adaptive scoring
- automatic skill routing
- cost accounting
- latency tracking
- project-specific reputation

---

## 40. MVP Recommendation

Start simple:

```text
Task
  -> Category + Role
  -> Model Registry
  -> Reputation Scores
  -> Eligibility Filter
  -> Jev
  -> Work
  -> Human PASS / FAIL
  -> Score + Strike Update
  -> JSONL + Markdown Logs
```

Do not start with every GitHub and statistical feature at once.

---

## 41. Non-Negotiable Rules

1. ATO owns model memory; Jev does not.
2. Eligibility filtering happens before Jev routing.
3. Human review has the strongest reputation weight.
4. Three human strikes in the same role/category trigger quarantine.
5. Quarantine is role/category-specific unless explicitly made global.
6. Builder and reviewer reputation are separate.
7. CI failure alone is not a human strike.
8. Every routing decision is logged.
9. Every reputation change is logged.
10. Model versions are tracked independently.
11. High-risk work can require cross-review.
12. Review loops are bounded.
13. Manual ChatGPT/Grok handoffs must be self-contained.
14. Token state may influence routing but not override high-risk quality requirements.
15. No provider is permanently assumed to be best.

---

## 42. Proposed Repository Structure

```text
ato/
├── README.md
├── pyproject.toml
├── src/
│   └── ato/
│       ├── __init__.py
│       ├── router.py
│       ├── classifier.py
│       ├── reputation.py
│       ├── outcome.py
│       ├── budgets.py
│       ├── github_collector.py
│       ├── reports.py
│       ├── handoffs.py
│       ├── models.py
│       └── cli.py
│
├── .at/
│   ├── config/
│   ├── intelligence/
│   ├── logs/
│   ├── handoffs/
│   └── state/
│
├── skills/
│   ├── at-route/
│   ├── at-classify/
│   ├── at-review/
│   ├── at-budget/
│   ├── at-reputation/
│   ├── at-outcome/
│   ├── at-github-evidence/
│   ├── at-human-review/
│   ├── at-report/
│   ├── at-chatgpt-handoff/
│   └── at-grok-handoff/
│
└── tests/
    ├── test_router.py
    ├── test_reputation.py
    ├── test_strikes.py
    ├── test_quarantine.py
    ├── test_outcome.py
    └── test_reports.py
```

---

## 43. CLI Ideas

```bash
ato task create
ato classify ATO-0184
ato route ATO-0184
ato budget status
ato models
ato model show claude
ato review ATO-0184 --pass
ato review ATO-0184 --fail
ato quarantine model category
ato probation model category
ato report
ato github sync
ato handoff chatgpt ATO-0184
ato handoff grok ATO-0184
```

---

## 44. Minimum Tests

1. Quarantined model is removed before Jev receives choices.
2. Human fail subtracts configured penalty.
3. Human fail adds a strike.
4. Third human strike quarantines role/category.
5. Unrelated category remains eligible.
6. Builder failure does not automatically penalize reviewer.
7. Reviewer failure can be recorded independently.
8. CI failure alone does not create a human strike.
9. Routing event is appended to JSONL.
10. Reputation event is appended to JSONL.
11. Markdown audit log is generated.
12. Low token state changes routing preference.
13. High-risk policy can force cross-review.
14. Material edit invalidates prior final inspection.
15. New model version creates distinct reputation record.

---

## 45. Build Assignment for Claude Code or Codex

Use this exact assignment:

```text
Build the ATO 3.0 MVP foundations described in this document.

Do not attempt every advanced feature at once.

Implement:
1. task records
2. task category and role classification
3. model registry
4. 0-1000 role/category reputation scores
5. human review events
6. three-strike role/category quarantine
7. eligibility filtering before Jev
8. JSONL audit history
9. Markdown human-readable reports
10. Jev adapter interface
11. token/budget state
12. unit tests

Important:
- ATO owns reputation; Jev does not.
- Jev must only receive eligible models.
- A model with three human strikes in a category must be excluded before Jev receives the routing menu.
- CI failures alone must never create human strikes.
- Builder and reviewer reputation must be separate.
- Preserve transparent logs for every routing and reputation change.
- Keep provider adapters modular.
- Do not add local Qwen/Ollama.
- ChatGPT Chat remains a manual handoff lane.
- Add README, architecture notes, example config, sample data, and proof commands.

After implementation, run all tests and report:
- files created
- architecture decisions
- test results
- known limitations
- next recommended phase
```

---

## 46. Definition of Success

ATO 3.0 succeeds when:

```text
Human submits task
  -> ATO classifies it
  -> ATO checks reputation
  -> ATO removes disqualified models
  -> Jev chooses among valid models
  -> selected model performs task
  -> tests/review/GitHub evidence collected
  -> human approves or rejects
  -> reputation updated
  -> future routing changes
  -> human can inspect exactly why
```

ATO 3.0 is:

> **An AI orchestration system that remembers outcomes, learns which models earn trust for which jobs, and changes future routing without hiding the reasoning from the human.**
