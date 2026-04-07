# ADR 004: Pluggable dispatch-fn for Event Format Flexibility

## Status
Draft

## Context
Re-frame requires events to be vectors where the first element is the event id:
`[:event-id arg1 arg2]`. This is simple and effective but constraining. Some
teams prefer map-based events (`{:id :event-id :data arg}`) for self-documenting
payloads or integration with other systems.

## Proposal
Rather than hardcoding the event format, provide a configurable `:dispatch-fn`
that extracts the handler id from event data. It would default to `first`
(re-frame compatible) but could be set to any function:

```cljs
(cfg :dispatch-fn :id)  ;; for map-based events
```

This function would be used throughout the event loop to resolve handler ids
from events, inputs, and effects.

## Consequences

### Benefits
- Users can choose vector, map, or custom event formats.
- Migration from re-frame is straightforward with the default `first`.

### Trade-offs
- `dispatch-fn` is used in multiple contexts (events, inputs, effects, preloads)
  where the data shape varies. This overloading means the function must work
  correctly for all these shapes, which is fragile. For example, `first` on
  `[:db]` and `[:db [:db]]` both happen to return `:db`, but this is
  coincidental and breaks with non-default dispatch functions.
- The specs (`::event` defined as `s/cat`) assume sequential events, conflicting
  with the stated goal of supporting map events.
- Changing `dispatch-fn` has far-reaching effects across the system that may not
  be immediately obvious.
