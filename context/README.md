# Context — Setup & Usage

## What's in this folder

```
context/
├── CONTEXT.md           ← Fill this in. Every agent reads it.
├── annotations/         ← Learnings agents accumulate over time
│   ├── services.md
│   ├── environments.md
│   ├── test-patterns.md
│   └── domain.md
└── hub/                 ← context-hub BYOD registry (optional)
    └── qa-workflow/
        └── docs/
            ├── stack/DOC.md
            └── conventions/DOC.md
```

## Step 1 — Fill in CONTEXT.md

Open `context/CONTEXT.md` and fill in your project details.
This is the only required setup step. Everything else is optional.

## Step 2 (optional) — Set up context-hub

context-hub gives agents a searchable, versioned registry of your project context.
Annotations persist across sessions. Team members can share learnings.

### Install

```bash
npm install -g @aisuite/chub
```

### Build the local registry

```bash
chub build context/hub/
```

This generates `context/hub/.chub-local/` with a `registry.json`.

### Configure chub to use it

Add to `~/.chub/config.yaml`:

```yaml
sources:
  - local: ./context/hub/.chub-local/registry.json
```

### Use in sessions

```bash
# Search your project context
chub search "stack"
chub search "conventions"

# Retrieve a document
chub get qa-workflow/stack
chub get qa-workflow/conventions

# Add a persistent annotation (stays in ~/.chub/, shown on next retrieval)
chub annotate qa-workflow/stack "auth callback has 2s delay on staging"

# Submit feedback on public context-hub docs
chub feedback openai/chat-api 5 "Works well for structured outputs"
```

## Step 3 — Agents maintain annotations automatically

Every agent skill is instructed to annotate learnings after completing its task.
Over time, `context/annotations/*.md` grows with project-specific knowledge.

When an annotation is stable and team-wide, commit it to the annotations file.
When it changes the stack, update `context/CONTEXT.md` directly.
