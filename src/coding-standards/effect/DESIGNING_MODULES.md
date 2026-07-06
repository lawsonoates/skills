# Designing Modules (Effect)

Keep things in one function unless composable or reusable. Abstraction must be justified by hidden complexity, owned policy, or a real seam — not by line count or future flexibility.

- Do not use `export namespace Foo { ... }` to organise code. Use the service shape below instead.
- One service per real capability; do not create a service for a pure function — export the function directly.
- Rely on inference for internal code; explicit types are for exports or clarity.
- A pass-through wrapper module earns its keep only when it hides complexity, owns policy, or translates across a real seam.
- Avoid re-exporting, with one exception: the self-reexport at the bottom of a service module (below), which gives namespace-like ergonomics without `namespace`.

## Service shape

One module per service: flat top-level exports, traced Effect methods, explicit layers, and a self-reexport at the bottom.

```ts
export interface Interface {
    readonly get: (id: FooID) => Effect.Effect<FooInfo, FooError>
}

export class Service extends Context.Service<Service, Interface>()("Foo") {}

export const layer = Layer.effect(
    Service,
    Effect.gen(function* () {
        const get = Effect.fn("Foo.get")(function* (id: FooID) {
            return yield* load(id)
        })

        return Service.of({ get })
    }),
)

export const defaultLayer = layer.pipe(Layer.provide(FooDep.defaultLayer))

export * as Foo from "./foo"
```

Rules:

- Use `Effect.fn("Foo.method")` for public service methods so they carry a span.
- Use `Effect.fnUntraced` for small internal helpers that do not need a span.
- Keep helpers as non-exported top-level declarations in the same file.
- Export `layer` (undressed) and `defaultLayer` (dependencies provided). Compose with `defaultLayer`; use `layer` when tests or callers substitute dependencies.
- Self-reexport with `export * as Foo from "./foo"` (or `from "."` for `index.ts`).
- Typed errors appear in `Interface` method signatures — the error channel is part of the service contract (see [`ERROR_HANDLING.md`](ERROR_HANDLING.md)).

## Schemas

Use Effect Schema as the source of truth.

- Use `Schema.Class` for exported data objects with a clear identity.
- Use `Schema.Struct` for local shapes and simple nested objects.
- Use `Schema.brand` for single-value IDs.
- Reuse named refinements instead of re-spelling constraints.
- Prefer narrow boundary helpers over generic schema-bridge layers.

## Inside Effect code, prefer Effect APIs

Yield existing services instead of dropping to ad hoc platform APIs.

- Use `HttpClient.HttpClient` instead of raw `fetch`.
- Use `Path.Path`, `Config`, `Clock`, and `DateTime` when already inside Effect.
- Use `Effect.callback` for callback-based APIs.
- Use `Effect.void` instead of `Effect.succeed(undefined)`.
- Use `Effect.cached` when concurrent callers should share one in-flight computation.
- For background loops, use `Effect.repeat` / `Effect.schedule` with `Effect.forkScoped` in the owning layer or state scope.

## One error, many seams

A domain error is only a domain error. It should never also be the HTTP body, the CLI formatter, and the event/wire payload — each seam catches the typed error and adapts it into the shape that seam owns. Service modules stay HTTP-agnostic: no HTTP status codes, `HttpApiError`, `HttpServerResponse`, or route-specific error schemas in service code. A service error class must not grow `toObject()`, an HTTP status, or formatting methods to serve its consumers.

```text
CLI
└─ format helper sees _tag ModelNotFoundError -> nice text

HTTP route
└─ catch ModelNotFoundError -> declared BadRequest public API error

Event/wire output
└─ catch ModelNotFoundError -> publish the wire shape that seam owns

HTTP middleware
└─ no ModelNotFoundError knowledge
```

## Boundary helpers

- HTTP handlers translate service errors into endpoint-declared public error schemas. Keep mappings inline when they are one-off; extract tiny shared helpers only when the same translation repeats.
- Shared helpers are narrow named constructors — `notFound(message)`, `badRequest(message)`.
- Anti-pattern: generic middleware as a registry of domain errors (`mapAnyDomainError(error)`). Middleware handles cross-cutting concerns and the final unknown-defect fallback, nothing domain-specific.
