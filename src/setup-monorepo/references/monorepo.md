# Monorepo

## Root `package.json`

```json
{
	"name": "<name>",
	"private": true,
	"workspaces": ["packages/*"],
	"type": "module",
	"scripts": {
		"typecheck": "bun turbo run typecheck"
	},
	"devDependencies": {
		"turbo": "^2.9.3"
	},
	"packageManager": "bun@<bun-version>",
	"catalog": {
		"typescript": "6.0.3"
	}
}
```

Packages: `workspace:*` for internal deps, `"catalog:"` for shared versions.

## `turbo.json`

```json
{
	"$schema": "https://turbo.build/schema.json",
	"tasks": {
		"typecheck": {}
	}
}
```