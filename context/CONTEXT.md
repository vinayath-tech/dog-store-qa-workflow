# Project Context — Pet dog store E-Commerce (Example)

> This is a completed example of `context/CONTEXT.md` for the Medusa demo app.
> Copy this to `context/CONTEXT.md` and adapt it to your own project.
> Last updated: 2026-03-31

---

## Automation Framework

- **Language**: TypeScript
- **Framework**: Playwright
- **Test runner**: @playwright/test
- **BDD format**: plain describe/it (no Gherkin)
- **Test file naming**: `*.spec.ts`
- **Test directory**: `dog-store-e2e-tests/tests/ui-tests/specs`
- **Run command**: `npx playwright test`
- **Tags used**: `@smoke`, `@regression`, `@must-test`, `@checkout`, `@cart`

## Application Under Test

- **Frontend**: Next.js 15 / React 19 / TypeScript (Medusa Storefront)
- **Backend**: Node.js 20 / Medusa v2 / TypeScript
- **API layer**: REST — base URL: `http://localhost:9000/store`
- **Admin API**: REST — base URL: `http://localhost:9000/admin`
- **Frontend URL (local)**: `http://localhost:8000`
- **Key repos**:
  - FE: `https://github.com/vinayath-tech/dog-store-frontend` (storefront)
  - BE: `https://github.com/vinayath-tech/dog-store-backend` (API + services)
  - QA: `https://github.com/vinayath-tech/dog-store-e2e-tests` (Playwright tests — created for this workflow)

## Project Management

- **Ticket system**: GitHub Issues
- **AC format**: Bullet points with expected behavior
- **Bug severity definitions**: Critical / Major / Minor / Trivial (see Bug Reporter skill)
- **Bug report format**: GitHub Issue with Steps to Reproduce, Expected, Actual, Severity

## CI/CD

- **Pipeline**: GitHub Actions
- **Branch naming**: `feature/ISSUE-123-description`
- **Test execution in CI**: `npx playwright test --project=chromium`
- **Environments**:
  - Local: `http://localhost:8000` (FE) / `http://localhost:9000` (BE)
  - Staging: N/A (demo app)
  - Production: N/A (demo app)

## Quality Standards

- **Definition of Done**:
  - All Must Test scenarios pass
  - No Critical or Major bugs open
  - Functional review shows no ❌ AC gaps
- **Regression scope**: Full suite on main branch, smoke only on feature PRs

## Preferences

- **Output language**: English
- **Tone**: Concise, developer-friendly
- **Terminology**:
  - "bug" not "defect"
  - "test scenario" not "test case"
  - "ticket" not "issue" or "story"
