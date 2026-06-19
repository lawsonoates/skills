# Workflow scripts

Run a pipeline once and exit.

## Anatomy

1. **Guard** — argv, branch, clean tree, environment
2. **Stages** — sequential steps, each a section comment
3. **Done** — exit naturally; last stage may delegate to monorepo tooling

```ts
#!/usr/bin/env bun

import { $ } from 'bun';

// ---- guard ----
if (process.env.NODE_ENV !== 'production') {
	console.error('Refusing to run outside production');
	process.exit(1);
}

// ---- prepare ----
await $`some-cli sync`;

// ---- validate ----
await $`bun run script/validate.ts`;

// ---- run ----
await $`some-tool run deploy`;
```

Top-level `await` for simple pipelines. Use `main()` when the script has multiple helpers.

## Complex workflows

When a script grows, keep the pipeline in `main()` and push details into small functions:

```ts
async function ensureClean() {
	const status = await $`git status --porcelain`.text();
	if (status.trim()) {
		console.error('Working tree not clean');
		process.exit(1);
	}
}

async function main() {
	console.log('\n=== release ===\n');

	await ensureClean();
	// ---- stage ----
	// ---- stage ----

	console.log('\n✓ done\n');
}

await main();
```

## Symmetric siblings

When two scripts do the same thing for different environments (dev/prod, staging/prod), mirror their shape. Only names and values should differ — not structure. Symmetry prevents mistakes.

## Env discipline (when scripts manage env)

1. Entry uses `--no-env-file` if the script selects its own env
2. Script names or pulls the correct env file explicitly
3. Validation is a separate script or import — fail before side effects
4. Pass env to child commands explicitly (`--env-file=`, `NODE_ENV=`, etc.)

## Delegation

Root scripts handle cross-cutting setup (env, auth, guards). Per-package or per-app work belongs in existing tooling (`turbo`, `npm run`, framework CLI). The script should not enumerate every package.

## Checklist

- [ ] Pipeline reads top-to-bottom
- [ ] Independent steps parallelized, dependent steps sequential
- [ ] Delegates domain work to packages or existing tooling
- [ ] Short unless complexity is earned