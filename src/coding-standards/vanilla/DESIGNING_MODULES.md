# Designing Modules

Keep things in one function unless composable or reusable. Abstraction must be justified by hidden complexity, owned policy, or a real seam — not by line count or future flexibility.

- Default to a namespace as the abstraction to organise code.
- Use a class only when instances are identity-based, stateful, and must be reused.

```ts
// Good
export namespace Journal {
    export async function read(dir: string) {
        return Bun.file(path.join(dir, 'journal.json')).json();
    }
}

// Bad: class with no identity or state
export class JournalReader {
    async read(dir: string) { /* ... */ }
}
```

- Avoid re-exporting. Import from the module that owns the thing.
- Rely on inference for internal code; explicit types are for exports or clarity.
- A pass-through wrapper module earns its keep only when it hides complexity, owns policy, or translates across a real seam.
