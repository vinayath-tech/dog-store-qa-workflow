---
name: qa-workflow/stack
description: Living project context for the QA workflow — stack, conventions, tools, and accumulated learnings
type: doc
tags: [qa, stack, context, living]
revision: 1
updated-on: 2026-03-29
source: local
---

# QA Workflow — Stack Context

> This document is the context-hub version of `context/CONTEXT.md`.
> It is retrieved by agents via `chub get qa-workflow/stack`.
> Update this file when your stack changes. Run `chub build context/hub/` to rebuild the registry.

## Quick reference

```bash
# Retrieve this document in a session
chub get qa-workflow/stack

# Add a project-specific annotation
chub annotate qa-workflow/stack "Discount is applied before tax — verified cart-service.ts:142"

# Search for related context
chub search "playwright typescript"
chub search "jira bug report"
```

## Stack

See `context/CONTEXT.md` for the full, editable stack definition.
This file mirrors that content for context-hub retrieval.

> Sync this file with `context/CONTEXT.md` whenever the stack section changes.

## Accumulated annotations

Annotations added via `chub annotate qa-workflow/stack "..."` are stored in `~/.chub/`
and appear automatically on the next `chub get qa-workflow/stack`.

To make annotations permanent and team-wide, paste them into `context/annotations/` and commit.
