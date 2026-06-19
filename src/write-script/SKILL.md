---
name: write-script
description: >
  Write high-quality Bun scripts — workflow pipelines (run once, exit) and service scripts
  (long-running processes). Use when creating script/ files, root automation, dev tooling,
  deploy pipelines, process supervisors, local servers, or refactoring shell scripts to TypeScript.
  Triggers: "write a script", "bun script", "orchestration script", "dev script", "deploy script",
  "long-running script", "spawn", "script/ folder". Use when the user runs /write-script.
---

# Write Script

Bun scripts coordinate work. They are not where business logic lives.

Pick the right kind first — then read the matching reference.

## Two kinds

| Kind | Runs | Exits when | Shape | Reference |
|------|------|------------|-------|-----------|
| **Workflow** | Once | Pipeline completes or fails | Guard → stages → done | [workflow.md](references/workflow.md) |
| **Service** | Until stopped | SIGINT/SIGTERM or child crash | Spawn/serve → supervise → shutdown | [service.md](references/service.md) |

## Shared foundations

### Shebang and entry

```ts
#!/usr/bin/env bun
```

Wire in `package.json` as a one-liner. The script file is the source of truth:

```json
"deploy": "bun run script/deploy.ts"
```

When the script manages its own env loading, use `--no-env-file` on the entry so Bun does not auto-load the wrong file.

### Bun-native APIs

| Need | Use |
|------|-----|
| Shell commands | `$` from `bun` |
| Read/write files | `Bun.file` |
| Find files | `Bun.Glob` |
| CLI args | `Bun.argv` |
| Child processes | `Bun.spawn` |
| HTTP server | `Bun.serve` |

Avoid `child_process`, `execa`, and argparse libraries.

### Fail fast

- Validate preconditions before side effects
- `console.error` with an actionable message, then `process.exit(1)`
- Prefer letting shell/process errors propagate — no `try/catch` unless at an HTTP or I/O boundary where you must return a response

```ts
const arg = Bun.argv[2];
if (!arg) {
	console.error('Usage: bun run script/foo.ts <arg>');
	process.exit(1);
}
```

### Section comments are the structure

Mark phases with scannable headers. A reader should understand the script in 10 seconds.

```ts
// ---- guard ----
// ---- prepare ----
// ---- run ----
```

### Orchestrate, don't implement

If a tool has a CLI, call it. If logic belongs in a package, import it. The script sequences, guards, and wires — it does not contain domain logic.

### Configuration as data

Repeated work is a list, not copy-paste:

```ts
const targets = ['api', 'worker', 'web'];

await Promise.all(targets.map((target) => $`... ${target} ...`));
```

- `Promise.all` when steps are independent
- Sequential `await` when order matters

### Keep it short

Simple scripts: 20–50 lines. Extract functions only when reused within the file or when complexity is earned.

### Naming (when you need helpers)

- `ensure*` — preconditions that exit on failure
- Pure helpers — no I/O
- I/O helpers — one concern each
- `main()` — the only place that sequences (complex workflows)

## Anti-patterns

| Avoid | Do instead |
|-------|------------|
| `try/catch` everywhere | Let errors propagate; catch only at boundaries (HTTP response, optional file ops) |
| argparse libraries | `Bun.argv` + usage string |
| Config files for the script | The script is the config |
| Shared script frameworks | Repeat the shape; symmetry replaces abstraction |
| Verbose logging | Section comments + sparse output |
| Domain logic | Delegate to packages |
| `child_process` / `execa` | `Bun.spawn` / `$` |

## Writing a new script

1. **Pick the kind** — workflow or service
2. **Read the reference** — [workflow.md](references/workflow.md) or [service.md](references/service.md)
3. **Name the command** — what does the user run?
4. **List stages** — guards, prep, run, shutdown
5. **Check siblings** — mirror shape if a similar script exists
6. **Write top-to-bottom** — section comments, minimal helpers
7. **Wire package.json** — one-line entry
8. **Verify** — every stage is a real command or call, not a placeholder

## Checklist

- [ ] Shebang `#!/usr/bin/env bun`
- [ ] Guards before side effects
- [ ] Section comments mark phases
- [ ] Actionable error messages
- [ ] Bun-native APIs (`$`, `Bun.spawn`, `Bun.file`, `Bun.argv`)
- [ ] `package.json` one-line entry
- [ ] Kind-specific checklist in the matching reference