---
name: git-commit-helper
description: >
  Writes conventional commit messages following the Conventional Commits spec.
  Use when user asks to commit changes, write commit messages, or mentions
  "conventional commits". Analyzes staged changes to generate appropriate
  type, scope, and description.
---

# Git Commit Helper

## Commit Message Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

## Types

| Type | When to use |
|------|-------------|
| feat | New feature or capability |
| fix | Corrects a bug or unintended behaviour |
| docs | Documentation only (comments, README, etc.) |
| style | Formatting only, zero logic change |
| refactor | Restructuring without fixing a bug or adding a feature |
| perf | Performance improvement with no behaviour change |
| test | Adding or updating tests |
| chore | Build process, tooling, or dependency updates |

### Type Decision Guide

When in doubt, ask these questions in order:

1. **Does it add new functionality for the user?** → `feat`
2. **Does it fix something that was broken or wrong?** → `fix`
   - Includes: fixing incorrect logic, wrong config values, bad regex, inaccurate comments that affect behaviour
   - Also includes: improving AI prompt rules that were causing wrong outputs
3. **Does it only restructure/rewrite without changing behaviour?** → `refactor`
4. **Is it only updating comments, README, or docs?** → `docs`
5. **Is it only faster with no logic change?** → `perf`
6. **Is it tooling, CI, or dependency related?** → `chore`

> **`fix` vs `chore` for config files**: If the config change corrects a bug or wrong behaviour → `fix`. If it's routine maintenance (updating a URL, bumping a version) → `chore`.

---

## Breaking Changes

Use when a change is **not backwards-compatible** (API removal, behaviour change that breaks callers).

Two ways to mark a breaking change — use either or both:

1. **Append `!` after the type/scope:**
   ```
   feat(api)!: remove deprecated v1 endpoints
   ```

2. **Add `BREAKING CHANGE:` footer** (required if you need to explain what breaks):
   ```
   feat(api)!: remove deprecated v1 endpoints

   BREAKING CHANGE: /api/v1/* routes have been removed. Migrate to /api/v2/.
   ```

> Use `!` alone for obvious removals. Add the `BREAKING CHANGE` footer when callers need migration instructions.

---

## Scope

Scope narrows the commit to a **functional area or module**, not a directory name.

### Rules

- **Use a functional name**, not a folder path.
  - ✅ `fix(ai-analysis): ...`
  - ❌ `fix(config): ...` ← "config" is a directory, not a module
- **Omit scope** when the change spans multiple unrelated modules, or when scope adds no useful information.
  - ✅ `fix: correct keyword matching and prompt anti-hallucination rules`
- **Use the subsystem name** that a reviewer would care about:
  - Feature areas: `auth`, `api`, `crawler`, `notification`, `ai-analysis`
  - Single files (when obvious): `readme`, `dockerfile`

---

## Body & Footer Format

- **Blank line required** between header and body, and between body and footer.
- **Wrap at 72 characters** per line in the body.
- Body answers *why*, not *what* — the diff already shows what.
- Multiple footers are allowed; each on its own line (`Token: value`).

```
feat(auth): implement JWT-based login

Previous session tokens were stateless cookies; this switches to signed
JWTs so mobile clients can share the same auth flow without a separate
endpoint.

Closes #142
```

---

## Workflow

1. Run `git diff --staged` to see what changed
2. Identify the primary type using the **Type Decision Guide** above
3. Determine the scope: functional module name, or omit if multi-module
4. Write a concise description (imperative mood, no period, ≤72 chars)
5. Add body only if the "why" isn't obvious from the description
6. Add `BREAKING CHANGE:` footer if this breaks any existing callers

---

## Examples

- `feat(auth): add OAuth2 login flow`
- `fix(api): handle null response from payment gateway`
- `fix(ai-analysis): add anti-hallucination rules and cap output length`
- `fix: correct keyword dedup and prompt output format`
- `perf(crawler): cache DNS lookups to reduce latency by 40%`
- `docs(readme): update installation instructions`
- `chore: bump litellm to 2.0.0`
- `feat(api)!: remove deprecated v1 endpoints` ← breaking change
