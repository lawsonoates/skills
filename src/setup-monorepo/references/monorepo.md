# Monorepo

## Root `package.json`

```json
{
	"$schema": "https://json.schemastore.org/package.json",
	"name": "<name>",
	"private": true,
	"workspaces": ["packages/*"],
	"type": "module",
	"scripts": {
		"typecheck": "bun turbo run typecheck"
	},
	"devDependencies": {
		"@tsconfig/bun": "catalog:",
		"@types/bun": "catalog:",
		"@typescript/native-preview": "catalog:",
		"turbo": "^2.9.3",
		"typescript": "catalog:"
	},
	"packageManager": "bun@<bun-version>",
	"catalog": {
		"@tsconfig/bun": "1.0.10",
		"@types/bun": "1.3.13",
		"@typescript/native-preview": "7.0.0-dev.20251207.1",
		"typescript": "6.0.3"
	}
}
```

Packages: `workspace:*` for internal deps, `"catalog:"` for shared versions.

## Package `package.json`

```json
{
	"$schema": "https://json.schemastore.org/package.json",
	"name": "@<scope>/<package>",
	"private": true,
	"type": "module",
	"exports": {
		"./<subpath>": "./src/<subpath>.ts"
	},
	"scripts": {
		"typecheck": "tsgo -b --noEmit"
	},
	"dependencies": {},
	"devDependencies": {
		"@tsconfig/bun": "catalog:",
		"@types/bun": "catalog:",
		"@typescript/native-preview": "catalog:",
		"typescript": "catalog:"
	}
}
```

`tsgo` comes from `@typescript/native-preview`. Use `tsgo -b --noEmit` for every package `typecheck` script.

## `turbo.json`

```json
{
	"$schema": "https://turbo.build/schema.json",
	"tasks": {
		"typecheck": {}
	}
}
```
