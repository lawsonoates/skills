---
name: setup-monorepo
description: Scaffold a new Bun monorepo (turbo, tsgo, ultracite). Triggers: setup monorepo, init monorepo, bun workspace. Use with /setup-monorepo.
---

# Setup Monorepo

Bootstrap a new Bun + TypeScript monorepo. Copy templates from references — don't invent config.

| Reference | Contents |
|-----------|----------|
| [monorepo.md](references/monorepo.md) | workspaces, catalog, turbo |
| [tsconfig.md](references/tsconfig.md) | `tsconfig.json` |
| [gitignore.md](references/gitignore.md) | `.gitignore` |

## Steps

1. Get repo name, npm scope, and initial package names.
2. Create root files and `packages/<name>/src/` per package — subpath `exports` only, no barrel `index.ts`.
3. Wire workspaces, catalog, and turbo per [monorepo.md](references/monorepo.md).
4. Run `bun install && bun run check && bun run typecheck`.

## Rules

- Bun only. `private: true`, `type: "module"` everywhere.
- Internal deps: `workspace:*`. Shared versions: root `catalog` + `"catalog:"`.
- Same [tsconfig.md](references/tsconfig.md) at root and in each package.