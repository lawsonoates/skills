---
name: coding-standards
description: TypeScript coding standards and design taste. Use when working on TypeScript code; when adding or changing domain models, modules, errors, async workflows, tests; or when another engineering skill needs the user's coding standards.
---

# Coding Standards

Use these standards while designing and editing TypeScript. They encode the user's taste: correctness first, typed failures, minimal abstraction, short names, Bun-native tooling.

## Effect vs non-Effect

Some rules differ depending on whether the codebase uses Effect. Determine this first: check `package.json` for `effect`. The style rules below apply either way; topic files are split per variant — load only the variant that matches the repo, never both.

## Core tenets

- Correctness, safety, and debuggability come before convenience.
- Keep things in one function unless composable or reusable.
- Model expected failures in typed channels; reserve throws for defects and boundary translation.
- Rely on type inference; avoid explicit type annotations or interfaces unless necessary for exports or clarity.
- Prefer functional array methods (`flatMap`, `filter`, `map`) over `for` loops; use type guards on `filter` to maintain type inference downstream.
- Avoid re-exporting.
- Use Bun APIs when possible. Read Bun API docs at `node_modules/bun-types/docs/**.md`.
- Improve changed paths without forcing broad migrations unless explicitly requested.

## Style

### Naming (mandatory for agent-written code)

Prefer single word names for variables and functions. Only use multiple words if necessary.

- Use single word names by default for new locals, params, and helper functions.
- Multi-word names are allowed only when a single word would be unclear or ambiguous.
- Do not introduce new camelCase compounds when a short single-word alternative is clear.
- Before finishing edits, review touched lines and shorten newly introduced identifiers where possible.
- Good short names to prefer: `pid`, `cfg`, `err`, `opts`, `dir`, `root`, `child`, `state`, `timeout`.
- Examples to avoid unless truly required: `inputPID`, `existingClient`, `connectTimeout`, `workerPath`.

```ts
// Good
const foo = 1;
function journal(dir: string) {}

// Bad
const fooBar = 1;
function prepareJournal(dir: string) {}
```

Reduce total variable count by inlining when a value is only used once.

```ts
// Good
const journal = await Bun.file(path.join(dir, 'journal.json')).json();

// Bad
const journalPath = path.join(dir, 'journal.json');
const journal = await Bun.file(journalPath).json();
```

### Destructuring

Avoid unnecessary destructuring. Use dot notation to preserve context.

```ts
// Good
obj.a;
obj.b;

// Bad
const { a, b } = obj;
```

### Variables

Prefer `const` over `let`. Use ternaries or early returns instead of reassignment.

```ts
// Good
const foo = condition ? 1 : 2;

// Bad
let foo;
if (condition) foo = 1;
else foo = 2;
```

### Control Flow

Avoid `else` statements. Prefer early returns.

```ts
// Good
function foo() {
    if (condition) return 1;
    return 2;
}

// Bad
function foo() {
    if (condition) return 1;
    else return 2;
}
```

## How to apply the standards

1. **Audit the local codebase.** Check whether the repo uses Effect and follow the matching variant in each topic file.
2. **Load the topic files that match the change.** The style rules above always apply.
3. **Prefer the smallest coherent improvement.** Do not add abstractions, modules, libraries, or config layers without a concrete reason.
4. **Before finishing, review touched lines** and shorten newly introduced identifiers where possible.

## Topic routing

Pick the column matching the repo (see "Effect vs non-Effect" above).

| If the change touches...                        | Effect repo                                                  | Otherwise                                                      |
| ------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------------------------- |
| Expected failures, errors, `try`/`catch`         | [`effect/ERROR_HANDLING.md`](effect/ERROR_HANDLING.md)       | [`vanilla/ERROR_HANDLING.md`](vanilla/ERROR_HANDLING.md)       |
| Code organisation, modules, exports              | [`effect/DESIGNING_MODULES.md`](effect/DESIGNING_MODULES.md) | [`vanilla/DESIGNING_MODULES.md`](vanilla/DESIGNING_MODULES.md) |

## Rejected framings

- **"The existing code throws, so new expected failures can throw."** Preserve compatibility at boundaries; do not copy unsafe failure contracts into new local logic.
- **"A wrapper is architecture."** A pass-through module earns its keep only when it hides complexity, owns policy, or translates across a real seam.
- **"Future flexibility justifies an interface."** A seam is real when behavior varies, a boundary translates, or tests substitute through an intentional seam.
- **"A helper function makes it cleaner."** Splitting a function is justified by composability or reuse, not line count.
