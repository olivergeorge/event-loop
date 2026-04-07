# ADR 005: Global Mutable Registry

## Status
Draft

## Context
Re-frame uses a global registry for subscriptions, events, effects, and
coeffects. This simplifies the API surface: `reg-event-fx`, `dispatch`, etc.
are all top-level calls against an implicit global. However, global mutable
state complicates testing, makes multiple independent instances impossible,
and couples all consumers.

## Proposal
Follow the re-frame convention: a single global `registry-ref` atom would hold
all handler registrations and configuration. `cfg` and `reg` would mutate this
atom. `dispatch` would read from it via `init-ctx`.

```cljs
(def registry-ref
  (atom {:std-ins {} :dispatch-fn first :error (partial js/console.error)}))
```

## Consequences

### Benefits
- Simple, familiar API matching re-frame conventions.
- No need to thread a registry through the component tree.
- Top-level `reg` calls work naturally at namespace load time.

### Trade-offs
- Cannot run multiple independent event loops (e.g. for isolated testing or
  micro-frontends).
- Top-level side-effecting code (including the `do` block at the bottom of the
  core namespace) executes on require, which can surprise consumers.
- Test isolation requires careful setup/teardown of the registry between tests.

### Future Consideration
An alternative would be to make the registry a first-class value passed to
`dispatch`, allowing multiple independent instances. The global registry could
then be sugar on top of this.
