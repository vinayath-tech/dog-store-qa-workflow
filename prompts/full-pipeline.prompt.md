# Prompt: Full QA Pipeline

> **Use when**: You want end-to-end QA on a PR — checkout the branch, run the app, validate in browser, review the diff, generate tests.
> **Agents**: orchestrator → environment-manager → functional-reviewer ∥ test-scenario-designer → browser-validation → bug-reporter → automation-writer
> **Output**: all files in `qa-output/`

---

Read `context/CONTEXT.md` and `context/annotations/` first.

Use the orchestrator skill to plan and then execute the full pipeline for this ticket.

**Ticket ID**: [TICKET-ID]

**PRs to test**:
```
[List PR numbers and repos, e.g.:
- Anasss/medusa-backend#2
- Anasss/medusa-storefront#2
]
```

**Acceptance Criteria**:
```
[Paste AC here, or reference the GitHub Issue number to read from]
```

**Additional context**:
```
[Epic, linked design doc, impacted services, anything relevant]
```

---

## Execution instructions for the orchestrator:

### Phase 1 — Environment Setup
1. Run `environment-manager`:
   - Checkout the PR branch(es) on the relevant repos
   - Run migrations and seed if backend changed
   - Start backend and storefront from the feature branch
   - Verify health checks pass
   - Save status to `qa-output/environment-status.md`

### Phase 2 — Analysis (parallel)
2. Run `functional-reviewer` and `test-scenario-designer` in parallel:
   - `functional-reviewer`: Read the git diff from the checked-out branches + AC → `qa-output/functional-review.md`
   - `test-scenario-designer`: Read AC → `qa-output/test-scenarios.md`

### Phase 3 — Live Browser Validation
3. Run `browser-validation` (Chrome MCP):
   - Read Must Test scenarios from `qa-output/test-scenarios.md`
   - Navigate the running app, execute steps, verify expected results
   - Take screenshots on failures
   - Save results to `qa-output/browser-validation.md`

### Phase 4 — Reporting (parallel)
4. Run `bug-reporter` and `automation-writer`:
   - `bug-reporter`: If gaps found in `functional-review.md` OR `browser-validation.md`, generate bug reports → `qa-output/bug-reports.md`
   - `automation-writer`: Read `qa-output/test-scenarios.md` → generate Playwright tests → `qa-output/automation/`

### Phase 5 — Summary
5. Print a final summary:
   - What agents ran
   - Key findings (gaps, bugs, pass/fail counts)
   - Files created in `qa-output/`
   - Overall verdict: APPROVE / APPROVE WITH CONDITIONS / REJECT
