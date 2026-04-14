# qa-output/

Agent outputs land here. Each file corresponds to one agent's output for the current ticket.

| File | Written by |
|---|---|
| `plan.md` | Orchestrator |
| `functional-review.md` | Functional Reviewer |
| `bug-reports.md` | Bug Reporter |
| `test-scenarios.md` | Test Scenario Designer |
| `automation/` | Automation Writer |
| `validation-report.md` | Manual Validator |

## Usage

Files here are read by downstream agents. Never paste content between agents — reference the file.

```bash
# Example: automation-writer reading from test-scenario-designer output
"Use the automation-writer skill. Read from qa-output/test-scenarios.md"
```

## Clearing outputs

Before starting a new ticket, clear this directory (or archive the previous outputs):

```bash
# Archive previous ticket outputs
mkdir -p qa-output/archive/TICKET-123
mv qa-output/*.md qa-output/archive/TICKET-123/

# Or simply delete
rm qa-output/*.md
rm -rf qa-output/automation/
```

> `qa-output/` is git-ignored by default. Add outputs to version control only if your team needs a persistent audit trail.
