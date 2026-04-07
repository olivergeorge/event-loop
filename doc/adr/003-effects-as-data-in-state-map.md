# ADR 003: Effects as Data in the State Map

## Status
Draft

## Context
Re-frame separates coeffects (inputs) from effects (outputs) into distinct maps
that flow through the interceptor context. This creates a clear boundary but
requires understanding both `:coeffects` and `:effects` maps and the context
object that carries them.

## Proposal
Accumulate effects in the same state map that logic functions transform, under
the `:fx` key. Logic functions would append effect descriptions to `:fx` using
ordinary map operations:

```cljs
(defn get-data [s]
  (update s :fx conj {::GET {:url "/endpoint/data" :cb #(dispatch [::get-resp %])}}))
```

A `:fx` transition handler would process this list after logic completes,
calling `do-effects` to dispatch each effect map entry to its registered
`:effect` handler.

## Consequences

### Benefits
- Single data shape: logic functions read and write one map, reducing cognitive
  load.
- Effects are visible in the state map during composition, making it easy to
  inspect or modify accumulated effects in later composed functions.
- Testing is straightforward: assert that the output map's `:fx` key contains
  expected effect descriptions.

### Trade-offs
- Effects and state share the same namespace, so `:fx` is a reserved key that
  logic must not accidentally clobber.
- The effect format (a list of maps where each map key is an effect handler id)
  is less structured than re-frame's explicit effect map. A typo in an effect
  key silently does nothing rather than failing loudly.
- The `do-effects` / `do-effect` chain iterates MapEntries from effect maps,
  which requires care around destructuring: effect handlers receive `[id payload]`
  pairs, not just the payload.
