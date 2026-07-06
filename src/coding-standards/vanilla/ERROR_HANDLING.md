# Error Handling

Expected failures live in typed return channels. Throws are reserved for defects (bugs, impossible states) and boundary translation.

Avoid `try`/`catch` blocks. Use `Result.tryPromise` instead.

```ts
// Good
const result = await Result.tryPromise(() => fetch(url));
if (result.isErr()) return result;

// Bad
try {
    const res = await fetch(url);
} catch (err) {
    // ...
}
```

- Return `Result` from functions that can fail in expected ways; let callers handle the error channel.
- Keep the failure type precise — a tagged union or error class per failure mode, not `Error`.
- Do not swallow errors. If a failure is ignored deliberately, say why in a comment.
- Do not log-and-rethrow at every layer; handle or propagate once.
- Secrets never enter error messages or logs.
