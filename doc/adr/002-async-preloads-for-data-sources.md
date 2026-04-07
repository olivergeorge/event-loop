# ADR 002: Async Preloads for Data Sources

## Status
Draft

## Context
Modern mobile and web apps increasingly depend on asynchronous data sources:
SQLite on React Native, IndexedDB, async storage APIs. In re-frame, accessing
these from event handlers pushes developers toward callback hell or requires
splitting logic across multiple events chained by side effects.

We want logic handlers to remain pure functions of their inputs, even when
some inputs come from async sources.

## Proposal
Introduce a `:preload` handler type. Before the synchronous logic phase runs,
all preloads for the event's registered inputs would be resolved via
`Promise.all`. The resolved values would be merged into the state map alongside
synchronous `:input` values.

```
dispatch(event)
  -> do-preloads (async, via Promise.all)
  -> do-event (synchronous: merge preloads + inputs, apply logic)
```

The `dispatch` function would always return a Promise. A separate
`dispatch-sync` would bypass preloads for cases where synchronous execution is
required.

## Consequences

### Benefits
- Logic handlers stay pure: they see preloaded data as ordinary map entries.
- Async sources (SQLite, IndexedDB) become transparent inputs.
- Promise.all ensures all data is ready before logic runs, avoiding partial
  state.

### Trade-offs
- Every `dispatch` call is async, even when no preloads are registered. This
  introduces microtask delays that can cause React input cursor jank. A fast
  synchronous path for events without preloads would mitigate this.
- Preloads are "reference data" only: they must resolve quickly and not block.
  Slow or failing async sources will stall the entire event loop for that
  dispatch. There is no timeout or cancellation mechanism.
- This feature is explicitly marked as unproven by the author; real-world
  usage patterns are still being discovered.

### Open Questions
- Should `do-dispatch` detect the absence of preloads and take a synchronous
  fast path to avoid unnecessary Promise overhead?
- How should preload errors be surfaced? Currently they propagate through the
  Promise chain to the `.catch` in `do-dispatch`.
