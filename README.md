# AI-First QA Workflow

A multi-agent QA workflow for Claude Code. AI reads your diffs, maps functional impact, designs test scenarios, and generates automation — all from a single workspace across multiple repos.

## Quick Start

```bash
# 1. Clone this repo into your workspace alongside your project repos
git clone https://github.com/your-org/qa-workflow.git

# 2. Fill in your project context
#    Open context/CONTEXT.md and fill in your stack details
#    (see examples/CONTEXT.example.md for a completed reference)

# 3. Configure MCP servers
#    Copy .mcp.json, fill in your tokens for GitHub/GitLab/Jira

# 4. Run your first agent
claude "Use the functional-reviewer skill on this AC and diff: ..."
```

## Architecture

```
                    ┌──────────────────────┐
                    │     ORCHESTRATOR     │
                    │    (Sonnet 4.6)      │
                    └─────────┬────────────┘
                              │ routes
            ┌─────────────────┼─────────────────┐
            ▼                 ▼                 ▼
   ┌────────────────┐ ┌──────────────┐  ┌──────────────┐
   │  FUNCTIONAL    │ │    TEST      │  │   BROWSER    │
   │  REVIEWER      │ │  SCENARIO    │  │  VALIDATOR   │
   │  (Opus 4.6)    │ │  DESIGNER    │  │ (Chrome MCP) │
   └───────┬────────┘ │ (Sonnet 4.6) │  └──────────────┘
           │          └──────┬───────┘
           ▼                 ├──────────────┐
   ┌────────────────┐        ▼              ▼
   │  BUG REPORTER  │ ┌──────────────┐ ┌──────────────┐
   │  (Sonnet 4.6)  │ │  AUTOMATION  │ │   MANUAL     │
   └────────────────┘ │  WRITER      │ │  VALIDATOR   │
                      │ (Sonnet 4.6) │ │ (Sonnet 4.6) │
                      └──────────────┘ └──────────────┘
```

## The Four Pillars

### 1. Multi-Repo AI Workspace

Your workspace connects AI to your full project context — all repos, all tools.

```
workspace/
├── storefront/          ← Frontend repo
├── api/                 ← Backend repo
├── qa-automation/       ← Test repo (Playwright/Cypress)
├── qa-workflow/         ← This repo (agents + prompts)
├── .mcp.json            ← Jira + GitHub/GitLab + Chrome MCP
└── AGENTS.md            ← AI behavioral instructions
```

### 2. Chrome MCP: AI Meets the Browser

AI navigates your app, checks elements, performs functional validations — the same work a manual tester does, but structured and repeatable.

```bash
# Use the browser validation prompt
"Use the browser-validation prompt on qa-output/test-scenarios.md"
# AI opens Chrome, navigates, validates, produces pass/fail report
```

### 3. Diff-First Functional Review

Feed a release diff and acceptance criteria to AI. Get back a prioritised QA brief.

```bash
# Single ticket review
"Use the diff-review prompt for PROJ-456"

# Multi-repo release analysis
"Use the release-analysis prompt for release v2.4.0"
```

### 4. AI-Assisted Test Generation & Selection

From analysis to action. Generate new tests, or select existing tests impacted by a change.

```bash
# Generate test scenarios + automation code
"Use the test-generation prompt for these ACs: ..."

# Select existing tests affected by a diff
"Use the smart-test-selector prompt for this diff: ..."
```

## Agents

| Agent | Model | What it does |
|---|---|---|
| [Orchestrator](.skills/orchestrator/SKILL.md) | Sonnet 4.6 | Routes tickets to the right agents in the right order |
| [Functional Reviewer](.skills/functional-reviewer/SKILL.md) | Opus 4.6 | Compares diff vs AC — finds gaps, risks, missing coverage |
| [Bug Reporter](.skills/bug-reporter/SKILL.md) | Sonnet 4.6 | Turns findings into developer-ready bug reports |
| [Test Scenario Designer](.skills/test-scenario-designer/SKILL.md) | Sonnet 4.6 | Generates 10–20 test scenarios per ticket (happy, negative, boundary, edge) |
| [Automation Writer](.skills/automation-writer/SKILL.md) | Sonnet 4.6 | Converts scenarios to Playwright/Cypress/Gherkin code |
| [Manual Validator](.skills/manual-validator/SKILL.md) | Sonnet 4.6 | Guides manual execution, tracks pass/fail, produces validation report |
| [Environment Manager](.skills/environment-manager/SKILL.md) | Sonnet 4.6 | Checks out PR branches, runs migrations, and starts the app locally for live testing |

## Prompts

Ready-to-use prompt templates in `prompts/`:

| Prompt | Use when |
|---|---|
| [diff-review.prompt.md](prompts/diff-review.prompt.md) | You have a diff + AC and want a functional review |
| [release-analysis.prompt.md](prompts/release-analysis.prompt.md) | You want to analyze a full release across multiple repos |
| [test-generation.prompt.md](prompts/test-generation.prompt.md) | You want AC → scenarios → automation code |
| [smart-test-selector.prompt.md](prompts/smart-test-selector.prompt.md) | You want to find existing tests affected by a change |
| [browser-validation.prompt.md](prompts/browser-validation.prompt.md) | You want AI to validate scenarios in a real browser via Chrome MCP |
| [full-pipeline.prompt.md](prompts/full-pipeline.prompt.md) | You want the full end-to-end QA pipeline |

## Output Chaining

Each agent writes to `qa-output/`. The next agent reads from there. Never paste between agents.

```
qa-output/
├── plan.md                 ← Orchestrator
├── environment-status.md   ← Environment Manager
├── functional-review.md    ← Functional Reviewer
├── bug-reports.md          ← Bug Reporter
├── test-scenarios.md       ← Test Scenario Designer
├── validation-report.md    ← Manual Validator
└── automation/             ← Automation Writer
```

## Living Context

`context/CONTEXT.md` is the single source of truth for your stack. Every agent reads it.
`context/annotations/` accumulates project-specific learnings across sessions.

Optional: set up [context-hub](https://github.com/andrewyng/context-hub) for a searchable registry with persistent annotations. See `context/README.md`.

## License

MIT
