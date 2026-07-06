# Error Handling (Effect)

Expected domain failures belong on the Effect error channel. Defects are for bugs, impossible states, and final unknown-boundary fallbacks.

## Modeling errors

- Use `Schema.TaggedErrorClass` for expected domain errors so callers can `catchTag` / `catchTags` precisely. `Data.TaggedError` is fine for purely internal failures that never cross a wire.
- Export a domain-level `Error` union from service modules.
- Use `Schema.Defect` for unknown cause fields.
- Keep the failure type precise — one tagged error per failure mode, not a generic `Error`.

```ts
export class SessionBusyError extends Schema.TaggedErrorClass<SessionBusyError>()("SessionBusyError", {
    sessionID: SessionID,
    message: Schema.String,
}) {}

export type Error = Storage.Error | SessionBusyError

export interface Interface {
    readonly get: (id: SessionID) => Effect.Effect<Info, Error>
}
```

Typed errors are part of the service contract, not something callers discover at runtime.

## Failing and translating

- In `Effect.gen` / `Effect.fn`, prefer `yield* new MyError(...)` for direct expected failures.
- Use `Effect.try(...)`, `Effect.tryPromise(...)`, `Effect.mapError`, `Effect.catchTag`, and `Effect.catchTags` to translate external failures into domain errors.
- When a Promise can reject with many unknown values but only one or two classes are expected domain failures, refine: map only the known rejection shapes to typed errors, let unknown rejections stay defects. Use `Effect.promise(...)` only when rejection means a defect outright.

## Defects

- Do not use `Effect.die(...)` for user, IO, validation, missing-resource, auth, provider, or busy-state failures — those are expected errors and belong on the error channel.
- Keep exactly one unknown-defect fallback at the outermost boundary: log `Cause.pretty(cause)`, return a safe generic body. Inner layers never pattern-match on defects to recover domain meaning.

## Hygiene

- Do not `try`/`catch` inside Effect code, and do not throw for expected failures — fail the effect.
- Do not swallow errors. If a failure is ignored deliberately, say why in a comment.
- Do not log-and-rethrow at every layer; handle or propagate once.
- Secrets never enter error messages or logs; use `Redacted` for secret values.
