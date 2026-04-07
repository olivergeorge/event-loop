# ADR 001: State-In, State-Out Over Interceptors

## Status
Draft

## Context
Re-frame uses an interceptor chain model for event handling. Interceptors are
powerful but add cognitive overhead: each interceptor carries `:before` and
`:after` fns, coeffect/effect maps flow through a context object, and the
ordering of interceptors matters in non-obvious ways. Developers frequently
struggle with the mental model of "what state is available where" inside the
chain.

## Proposal
Replace the interceptor chain with plain functions that take a state map and
return a state map. Logic handlers would have the signature
`(fn [state event] state')`. State keys like `:db` and `:fx` would be
first-class entries in this map, read and written directly.

Composition would be achieved via `comp` on these functions:
```cljs
(reg {:id :app/bootstrap :logic (comp set-loading get-data log-state)})
```

## Consequences

### Benefits
- Logic functions are trivially testable: pass a map in, assert on the map out.
- Composition via `comp` is a familiar, well-understood primitive.
- No implicit ordering concerns; composition order is explicit and left-to-right
  in execution (right-to-left in `comp` declaration).
- The mental model is drastically simpler: one data shape flows through.

### Trade-offs
- Interceptors supported cross-cutting concerns (tracing, validation) that could
  be injected without changing handler registration. This model requires
  wrapping or composing such concerns explicitly.
- There is no `:before`/`:after` distinction, so "undo" style interceptors
  (that clean up on the way back out) need a different pattern.
- All composed functions must agree on the shape of the state map, coupling them
  to the `:std-ins` configuration.
