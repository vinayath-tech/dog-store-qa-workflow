---
name: environment-manager
description: Checks out PR branches, runs migrations, and starts the application locally for live testing
---

# Skill: Environment Manager

> **Trigger**: A PR or feature branch needs to be tested locally before QA agents can validate.
> **Reads**: PR number or branch name from the execution plan
> **Writes**: `qa-output/environment-status.md`

## Role

You are the environment manager. Your job is to prepare the local workspace so the feature under test is running and accessible. You check out branches, run migrations, seed data, start services, and verify the application is healthy before handing off to other agents.

## Prerequisites

Before running, verify:
1. Docker is running (`docker ps` should work) — needed for PostgreSQL
2. The workspace repos exist at the paths defined in `context/CONTEXT.md`
3. Ports 8000 and 9000 are available (kill existing processes if needed)

## Step 1 — Identify repos and branches

From the execution plan or user input, determine:

| Repo | Branch to checkout | PR number |
|---|---|---|
| medusa-backend | `feature/ISSUE-N-...` | #N |
| medusa-storefront | `feature/ISSUE-N-...` | #N |

Not all PRs touch both repos. Only checkout repos that have changes.

## Step 2 — Checkout branches

For each repo with a feature branch:

```bash
cd <repo-directory>
git fetch origin
git checkout <branch-name>
git pull origin <branch-name>
```

**Safety rules:**
- Always `git stash` uncommitted changes before checkout
- Record the previous branch so it can be restored later
- If checkout fails, report the error and stop — do not continue with wrong branch

## Step 3 — Prepare the backend

If the backend branch changed, run these steps in order:

```bash
cd medusa-backend

# 1. Install dependencies (if package.json changed)
npm install

# 2. Check if new modules were added (check medusa-config.ts diff)
#    If yes, generate and run migrations:
npx medusa db:generate <module-name>
npx medusa db:migrate

# 3. If only existing modules changed, just run migrations:
npx medusa db:migrate

# 4. If seed script changed, re-seed:
npm run seed

# 5. Create admin user (if fresh database):
npx medusa user -e admin@medusa-test.com -p supersecret
```

**Database reset** (only if migrations fail or schema conflicts):

```bash
# Stop backend first
# Drop and recreate database via Docker:
docker exec medusa-postgres psql -U medusa -d postgres -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'medusa-db' AND pid <> pg_backend_pid();"
docker exec medusa-postgres psql -U medusa -d postgres -c "DROP DATABASE IF EXISTS \"medusa-db\";"
docker exec medusa-postgres psql -U medusa -d postgres -c "CREATE DATABASE \"medusa-db\";"

# Then re-run migrations + seed + create user
npx medusa db:migrate
npm run seed
npx medusa user -e admin@medusa-test.com -p supersecret
```

## Step 4 — Start the backend

```bash
cd medusa-backend
npm run dev  # runs in background
```

Wait for: `Server is ready on port: 9000`

## Step 5 — Update storefront API key

After backend starts with a fresh database, the publishable API key changes:

```bash
# Get the new key
TOKEN=$(curl -s http://localhost:9000/auth/user/emailpass -X POST \
  -H 'Content-Type: application/json' \
  -d '{"email":"admin@medusa-test.com","password":"supersecret"}' \
  | sed 's/.*"token":"\([^"]*\)".*/\1/')

NEW_KEY=$(curl -s http://localhost:9000/admin/api-keys \
  -H "Authorization: Bearer $TOKEN" \
  | sed 's/.*"token":"\([^"]*\)".*/\1/')

# Update storefront .env.local
sed -i "s/NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=.*/NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=$NEW_KEY/" \
  medusa-storefront/.env.local
```

## Step 6 — Start the storefront

```bash
cd medusa-storefront

# Clear Next.js cache (important after branch switch)
rm -rf .next

npm run dev  # runs in background
```

Wait for: `Ready in` message.

## Step 7 — Health check

Verify everything is running:

| Check | Command | Expected |
|---|---|---|
| Backend API | `curl -s http://localhost:9000/health` | 200 OK |
| Store products | `curl -s http://localhost:9000/store/products -H "x-publishable-api-key: $NEW_KEY"` | JSON with products |
| Storefront | `curl -s http://localhost:8000 -o /dev/null -w '%{http_code}'` | 200 or 307 (redirect) |
| Feature endpoint (if applicable) | `curl -s http://localhost:9000/store/<new-endpoint>` | Expected response |

## Output format

Save to `qa-output/environment-status.md`.

```markdown
## Environment Status

**Date**: [date]
**Ticket**: [ID]

### Branches Checked Out

| Repo | Branch | Previous Branch | Status |
|---|---|---|---|
| medusa-backend | feature/ISSUE-1-... | main | ✅ Checked out |
| medusa-storefront | feature/ISSUE-1-... | main | ✅ Checked out |

### Services Running

| Service | URL | Status |
|---|---|---|
| Backend API | http://localhost:9000 | ✅ Running |
| Storefront | http://localhost:8000 | ✅ Running |
| PostgreSQL | localhost:5432 | ✅ Running |

### Migrations & Seed

- [x] Migrations ran successfully
- [x] Seed data applied
- [x] Admin user created

### Health Checks

| Endpoint | Status | Response |
|---|---|---|
| /health | ✅ 200 | OK |
| /store/products | ✅ 200 | 12 products returned |
| Feature endpoint | ✅ 200 | [describe response] |

### Publishable API Key
`pk_xxxxx...xxxxx`

### Ready for Testing
**YES** — All services running, feature branch code deployed locally.
```

## Cleanup (after testing)

To restore the workspace to main after testing:

```bash
cd medusa-backend && git checkout main
cd medusa-storefront && git checkout main
# Optionally re-seed with main branch data
```

## Rules

- Always verify services are healthy before reporting "ready"
- If any step fails, stop and report the failure — do not continue with a broken environment
- Record the publishable API key — other agents may need it
- If the database needs a full reset, warn the user before dropping data
- Keep the previous branch name so it can be restored
