# Prompt: Test Generation Pipeline

> **Use when**: You want to go from AC → test scenarios → automated tests in one flow.
> **Agents**: test-scenario-designer → automation-writer
> **Output**: `qa-output/test-scenarios.md` + `qa-output/automation/`

---

Read `context/CONTEXT.md` and `context/annotations/` first.

**Step 1** — Run the test-scenario-designer skill:

**Ticket ID**: [TICKET-ID]

**Acceptance Criteria**:
```
[Paste AC here]
```

**Existing test coverage** (optional — paste file paths or scenario IDs to avoid duplication):
```
[e.g., e2e/checkout.spec.ts already covers happy path for guest checkout]
```

Save scenarios to `qa-output/test-scenarios.md`.

---

**Step 2** — Once `qa-output/test-scenarios.md` is saved, run the automation-writer skill:

Read `.skills/automation-writer/SKILL.md` for full instructions.
Read `qa-output/test-scenarios.md` for the scenarios to automate.

Generate tests for all Must Test and Should Test scenarios.
Save draft output to `qa-output/automation/`.

Then copy the generated spec files and page objects into the live test repo:
- Spec files → `dog-store-e2e-tests/tests/ui-tests/specs/`
- Page objects → `dog-store-e2e-tests/tests/ui-tests/pages/`

Before writing, read any existing files in those directories to match the project's
existing patterns, imports, and conventions.

Framework override (leave blank to use context/CONTEXT.md):
```
[e.g., "Use Cypress + JavaScript" or leave blank]
```
