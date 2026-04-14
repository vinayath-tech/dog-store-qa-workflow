# Prompt: Diff-First Functional Review

> **Use when**: You have a diff and AC and want to run the functional review directly without the orchestrator.
> **Agent**: functional-reviewer
> **Output**: `qa-output/functional-review.md`

---

Read `context/CONTEXT.md` and `context/annotations/` first.

Then use the functional-reviewer skill with the following inputs:

**Ticket ID**: [TICKET-ID]

**Acceptance Criteria**:
```
[Paste AC here — Given/When/Then, bullet points, or free-form]
```

**Code Diff** (unified format):
```diff
[Paste git diff output here]
```

**Additional context** (optional):
```
[Linked specs, design doc URL, notes about the change]
```

Save the output to `qa-output/functional-review.md`.

If gaps are found, automatically run the bug-reporter skill against the output.
