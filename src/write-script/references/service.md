# Service scripts

Run until interrupted or a child exits. Supervise processes, serve HTTP, or watch files.

## Anatomy

1. **Guard** — environment, port availability, required binaries
2. **Start** — spawn children or start a server
3. **Supervise** — handle signals, propagate child exit
4. **Shutdown** — kill children, flush I/O, exit cleanly

## Process supervisor

Spawn children with inherited stdio. When any child exits, stop the rest.

```ts
#!/usr/bin/env bun

// ---- guard ----
if (process.env.NODE_ENV !== 'development') {
	console.error('Dev only');
	process.exit(1);
}

// ---- start ----
const a = Bun.spawn(['bun', 'run', 'script/a.ts'], {
	stderr: 'inherit',
	stdin: 'inherit',
	stdout: 'inherit',
});

const b = Bun.spawn(['some-cli', 'watch'], {
	stderr: 'inherit',
	stdin: 'inherit',
	stdout: 'inherit',
});

// ---- supervise ----
let stopping = false;

async function stop(code = 0) {
	if (stopping) return;
	stopping = true;

	a.kill('SIGTERM');
	b.kill('SIGTERM');

	await Promise.allSettled([a.exited, b.exited]);
	process.exit(code);
}

process.on('SIGINT', () => stop(0));
process.on('SIGTERM', () => stop(0));

const [aCode, bCode] = await Promise.all([a.exited, b.exited]);
await stop(aCode !== 0 || bCode !== 0 ? 1 : 0);
```

Key points:

- `stopping` flag prevents double shutdown
- `Promise.allSettled` on `exited` — wait for children to actually die
- If either child exits unexpectedly, tear down the other and exit non-zero
- Inherit stdio so output is visible

## HTTP service

Use `Bun.serve` for local tooling servers. Register signal handlers to flush and close resources.

```ts
#!/usr/bin/env bun

const port = 4318;

// ---- start ----
Bun.serve({
	port,
	routes: {
		'/': async (req) => {
			const body = await req.text();
			// handle request
			return new Response(null, { status: 200 });
		},
	},
});

console.log(`listening on ${port}`);

// ---- supervise ----
const stop = async () => {
	// flush writers, close connections
	process.exit(0);
};

process.on('SIGINT', stop);
process.on('SIGTERM', stop);
```

## Piped children

When piping stdin to a child (`stdin: 'pipe'`), end the pipe and kill the child on shutdown:

```ts
const child = Bun.spawn(['processor'], { stdin: 'pipe', stderr: 'inherit', stdout: 'inherit' });

const stop = async () => {
	child.stdin.end();
	child.kill();
	process.exit(0);
};
```

## Env guards

Service scripts often guard on `NODE_ENV` or a dev-only flag at the top. Workflow scripts guard on git state, argv, or deployment target. Same pattern, different preconditions.

## Checklist

- [ ] Children spawned with appropriate stdio (`inherit` or `pipe`)
- [ ] SIGINT and SIGTERM handled
- [ ] Shutdown flag prevents double teardown
- [ ] Child exit propagates — one crash stops the rest
- [ ] Resources flushed/closed on shutdown