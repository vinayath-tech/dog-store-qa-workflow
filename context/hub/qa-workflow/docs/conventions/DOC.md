---
name: qa-workflow/conventions
description: QA workflow conventions — output chaining, agent behavior, naming, and team standards
type: doc
tags: [qa, conventions, agents, workflow]
revision: 1
updated-on: 2026-03-29
source: local
---

# QA Workflow — Conventions

> Retrieved via `chub get qa-workflow/conventions`

## Output files

| Agent | Output file |
|---|---|
| Orchestrator | `qa-output/plan.md` |
| Functional Reviewer | `qa-output/functional-review.md` |
| Bug Reporter | `qa-output/bug-reports.md` |
| Test Scenario Designer | `qa-output/test-scenarios.md` |
| Automation Writer | `qa-output/automation/` |
| Manual Validator | `qa-output/validation-report.md` |

## Chaining rule

Never paste raw agent output into the next agent.
Always reference the file: `"Read from qa-output/test-scenarios.md"`.

## Naming conventions

See `context/CONTEXT.md → CI/CD` for test file naming and branch naming.

## Priority labels

| Label | Meaning |
|---|---|
| Must Test | Core AC, high risk — must pass before release |
| Should Test | Important but lower risk |
| Could Test | Edge case, cosmetic — run when time permits |

## Bug severity

See `context/CONTEXT.md → Project Management → Bug severity definitions`.

## Learning loop

After every agent task:
1. Did you learn something not in `context/CONTEXT.md`? → annotate it.
2. Is the learning permanent and team-wide? → commit it to `context/annotations/`.
3. Is it a stack change? → update `context/CONTEXT.md` directly.
