# Multi-Repo Workspace Setup — Medusa Demo App

This guide sets up the multi-repo AI workspace used in the training.
By the end, you'll have three repos in a single workspace with AI context wired across all of them.

---

## What you're building

```
medusa-qa-workspace/
├── medusa-backend/              ← Medusa v2 API + Admin (Node.js / TypeScript)
├── medusa-storefront/           ← Next.js 15 storefront (React / TypeScript)
├── qa-automation/               ← Playwright test suite (TypeScript)
├── qa-workflow/                 ← This repo (agents + prompts)
├── .mcp.json                    ← Chrome MCP + GitHub connected
└── AGENTS.md                    ← (symlink or copy from qa-workflow/)
```

| Component | URL | Port |
|---|---|---|
| Backend API | `http://localhost:9000` | 9000 |
| Admin Dashboard | `http://localhost:9000/app` | 9000 |
| Storefront | `http://localhost:8000` | 8000 |

---

## Prerequisites

- **Node.js v20+** (below v25)
- **Git**
- **PostgreSQL** installed and running
- **Yarn** (recommended) or npm

Verify:
```bash
node -v    # v20.x or higher
git --version
psql --version
```

---

## Step 1 — Create the workspace directory

```bash
mkdir medusa-qa-workspace
cd medusa-qa-workspace
```

---

## Step 2 — Install Medusa Backend

```bash
# Create the Medusa app (includes backend + admin)
npx create-medusa-app@latest medusa-backend

# Follow the prompts:
#   - Database: let it create a new PostgreSQL database
#   - Admin email: admin@test.com (for demo)
#   - Admin password: supersecret (for demo)

cd medusa-backend
```

**Seed demo data** (products, categories, etc.):

```bash
# Medusa seeds some data during setup. If you need more:
npx medusa exec ./src/scripts/seed.ts
# Or use the admin dashboard to create products manually
```

**Start the backend**:

```bash
yarn dev
# API: http://localhost:9000
# Admin: http://localhost:9000/app
```

Leave this running. Open a new terminal.

---

## Step 3 — Install the Next.js Storefront

```bash
cd medusa-qa-workspace

git clone https://github.com/medusajs/nextjs-starter-medusa.git medusa-storefront
cd medusa-storefront
```

**Configure environment**:

```bash
cp .env.template .env.local
```

Edit `.env.local`:

```env
# Backend URL
MEDUSA_BACKEND_URL=http://localhost:9000

# Get this from Admin → Settings → Publishable API Keys
NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=pk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**How to get the publishable key**:
1. Open `http://localhost:9000/app` in your browser
2. Log in with the admin credentials you set during setup
3. Go to **Settings → Publishable API Keys**
4. Copy the key and paste it into `.env.local`

**Install and start**:

```bash
yarn install
yarn dev
# Storefront: http://localhost:8000
```

Leave this running. Open a new terminal.

---

## Step 4 — Create the QA Automation repo

```bash
cd medusa-qa-workspace

mkdir qa-automation
cd qa-automation
npm init -y
```

**Install Playwright**:

```bash
npm init playwright@latest
# Choose: TypeScript, tests folder: e2e, install browsers: yes
```

**Configure Playwright** — edit `playwright.config.ts`:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  timeout: 30000,
  retries: 0,
  use: {
    baseURL: 'http://localhost:8000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'chromium', use: { browserName: 'chromium' } },
  ],
});
```

**Create your first smoke test** — `e2e/smoke.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Storefront Smoke Tests', () => {
  test('homepage loads and shows products', async ({ page }) => {
    await page.goto('/');
    await expect(page).toHaveTitle(/Medusa/i);
    await expect(page.getByRole('main')).toBeVisible();
  });

  test('product listing page loads', async ({ page }) => {
    await page.goto('/store');
    await expect(page.locator('[data-testid="product-wrapper"]').first()).toBeVisible();
  });

  test('can view a product detail page', async ({ page }) => {
    await page.goto('/store');
    await page.locator('[data-testid="product-wrapper"]').first().click();
    await expect(page.getByRole('button', { name: /add to cart/i })).toBeVisible();
  });
});
```

**Verify it works**:

```bash
npx playwright test
```

---

## Step 5 — Clone the QA Workflow repo

```bash
cd medusa-qa-workspace

git clone https://github.com/your-org/qa-workflow.git
```

---

## Step 6 — Wire the workspace with .mcp.json

Create `.mcp.json` at the **workspace root** (not inside any repo):

```bash
cd medusa-qa-workspace
```

Create `.mcp.json`:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/chrome-devtools-mcp@latest"],
      "env": {}
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<your-github-token>"
      }
    }
  }
}
```

---

## Step 7 — Fill in the project context

Copy the example and adapt:

```bash
cp qa-workflow/examples/CONTEXT.example.md qa-workflow/context/CONTEXT.md
```

Or edit `qa-workflow/context/CONTEXT.md` directly with your workspace details.

---

## Step 8 — Verify the workspace

With all three services running, verify everything works:

```bash
# Backend health check
curl http://localhost:9000/health

# Store API — list products
curl http://localhost:9000/store/products

# Storefront — should return HTML
curl -s http://localhost:8000 | head -5

# Playwright tests
cd qa-automation && npx playwright test

# Chrome MCP — open Claude Code in the workspace root
cd medusa-qa-workspace
claude
# Then: "Navigate to http://localhost:8000 and take a screenshot"
```

---

## Step 9 — Run your first QA workflow

Now that the workspace is ready, try the full pipeline:

```bash
# Open Claude Code from the workspace root
cd medusa-qa-workspace
claude
```

### Example 1 — Browser validation

```
Navigate to http://localhost:8000/store, click on the first product,
add it to cart, and verify the cart count updates.
```

### Example 2 — Diff-first functional review

```
Use the functional-reviewer skill.

AC:
- User can add a product to cart from the product detail page
- Cart icon shows the number of items
- User can view cart contents

Diff:
[paste a git diff from the storefront repo]
```

### Example 3 — Test generation

```
Use the test-scenario-designer skill for these ACs:

AC-1: User can browse the product catalog with pagination
AC-2: User can filter products by category
AC-3: User can search for products by name
```

---

## Useful commands reference

| Action | Command | Directory |
|---|---|---|
| Start backend | `yarn dev` | `medusa-backend/` |
| Start storefront | `yarn dev` | `medusa-storefront/` |
| Run all tests | `npx playwright test` | `qa-automation/` |
| Run smoke tests | `npx playwright test --grep @smoke` | `qa-automation/` |
| Open Playwright UI | `npx playwright test --ui` | `qa-automation/` |
| Get release diff | `git diff v2.13.0..HEAD --stat` | any repo |
| Open Claude Code | `claude` | workspace root |

---

## Key API endpoints for testing

| Endpoint | Method | Description |
|---|---|---|
| `/store/products` | GET | List products (paginated) |
| `/store/products/{id}` | GET | Get product details |
| `/store/carts` | POST | Create a new cart |
| `/store/carts/{id}` | GET | Get cart contents |
| `/store/carts/{id}/line-items` | POST | Add item to cart |
| `/store/carts/{id}/line-items/{id}` | DELETE | Remove item from cart |
| `/store/carts/{id}/complete` | POST | Complete checkout |
| `/store/customers` | POST | Register a customer |
| `/store/shipping-options` | GET | List shipping methods |
| `/health` | GET | Backend health check |

---

## Troubleshooting

**PostgreSQL not running**
```bash
# macOS
brew services start postgresql@16

# Linux
sudo systemctl start postgresql

# Windows
net start postgresql-x64-16
```

**Publishable key not found**
Log into admin at `http://localhost:9000/app` → Settings → Publishable API Keys.
If no key exists, create one.

**Storefront shows blank page**
Check `.env.local` has the correct `MEDUSA_BACKEND_URL` and `NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY`.
Restart the storefront with `yarn dev`.

**Playwright tests fail with "page not found"**
Ensure both backend (port 9000) and storefront (port 8000) are running before running tests.

**Port already in use**
```bash
# Find what's using the port
lsof -i :9000   # macOS/Linux
netstat -ano | findstr :9000   # Windows
```
