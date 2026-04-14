# Prompt: Multi-Repo Release Analysis

> **Use when**: A release is shipping and you need to understand the combined functional impact across all repos before testing.
> **This is the Diff-First Method applied at release scale.**
> **Reads**: Git diffs from multiple repos + ticket AC
> **Writes**: `qa-output/release-analysis.md`

---

## Instructions

Read `context/CONTEXT.md` for repo locations, environments, and AC format.

### Step 1 — Gather diffs

For each repo in the release, get the diff between the last release tag and the current branch:

```bash
# In each repo directory:
cd <fe-repo>
git diff <last-release-tag>..HEAD --stat
git diff <last-release-tag>..HEAD

cd <backend-repo>
git diff <last-release-tag>..HEAD --stat
git diff <last-release-tag>..HEAD

cd <qa-repo>
git diff <last-release-tag>..HEAD --stat
```

Or provide the diff ranges manually:
- **Frontend diff**: `[paste or describe]`
- **Backend diff**: `[paste or describe]`
- **Release tickets**: `[list ticket IDs with their AC]`

### Step 2 — Analyse each change

For every meaningful change in the diff, classify it:

| Symbol | Meaning |
|---|---|
| ⚠️ | Functional change with risk — needs targeted testing |
| ✅ | Low-risk change — display-only, config, cosmetic |
| ❌ | AC gap — an acceptance criterion has no corresponding code change |
| 🔗 | Cross-repo dependency — change in one repo affects behaviour in another |

### Step 3 — Map cross-repo impact

This is the step that single-repo analysis misses:
- Does a backend API change affect frontend behaviour?
- Does a frontend route change break existing E2E tests?
- Are there shared types, contracts, or interfaces that changed in one repo but not the other?

## Output format

Save to `qa-output/release-analysis.md`.

```markdown
## Release Analysis

**Release**: [version or tag]
**Date**: [date]
**Repos analysed**: [list]
**Total changes**: [N files across M repos]

### Impact Summary

#### Frontend — [repo name]
⚠️ `cart.component.ts` → Discount display now reads from new `discountPreTax` field.
  Risk: HIGH — depends on backend field that may not be deployed yet.
  Test: cart with active discount, verify displayed value matches API response.

✅ `profile.component.ts` → Avatar border radius changed from 4px to 50%.
  Risk: LOW — cosmetic only.

#### Backend — [repo name]
⚠️ `cart-service.ts:142` → Discount calculation now applies BEFORE tax instead of after.
  Risk: HIGH — impacts all checkout flows.
  Test: multi-item cart with % discount, fixed discount, stacked coupons.
  🔗 **Cross-repo**: Frontend `cart.component.ts` reads `discountPreTax` — verify field exists.

✅ `user-service.ts` → Added logging to login endpoint.
  Risk: LOW — no functional change.

#### QA — [repo name]
⚠️ `checkout.spec.ts` → Existing discount test asserts post-tax value.
  Risk: HIGH — this test WILL FAIL after backend change. Must update expected value.

### AC Compliance

| Ticket | AC | Status | Code change | Notes |
|---|---|---|---|---|
| PROJ-456 | AC-1: Discount applies before tax | ✅ Covered | cart-service.ts:142 | — |
| PROJ-456 | AC-2: User sees updated total | ⚠️ Partial | cart.component.ts | Reads new field, but no loading state |
| PROJ-456 | AC-3: Confirmation email sent | ❌ No code | — | No email logic in this release |

### Cross-Repo Dependencies

| Source change | Affected repo | Risk | Action needed |
|---|---|---|---|
| BE: `discountPreTax` field added | FE: `cart.component.ts` reads it | HIGH | Deploy BE before FE |
| BE: discount calc changed | QA: `checkout.spec.ts` | HIGH | Update test expected value |

### Recommended Test Focus

1. **Must test**: [ordered list of highest-risk flows with specific scenarios]
2. **Must update**: [existing tests that will break]
3. **Can skip**: [changes with no functional risk]

### Deployment Order Recommendation
[If cross-repo dependencies require specific deploy sequencing, state it here.]

### Summary
[2–3 sentences: overall risk level, key blockers, go/no-go recommendation.]
```

## Rules

- Analyse every file in the diff, not just the ones that look important.
- Always check for cross-repo impact — this is the unique value of multi-repo analysis.
- If AC is provided, verify every criterion has a corresponding code change. Flag gaps with ❌.
- Output must be readable by a QA lead who hasn't seen the code — write in tester language, not developer language.
- If diffs are too large, summarise low-risk changes and focus detail on high-risk items.
