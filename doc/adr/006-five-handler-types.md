# ADR 006: Five Distinct Handler Types

## Status
Draft

## Context
Re-frame has separate registries for events, subscriptions, effects, and
coeffects. The event loop here needs to orchestrate async data loading, state
sampling, pure logic, state persistence, and side effects.

## Proposal
Define five handler types, all registered under a single `:handlers` map keyed
by id. Each handler id could provide any combination of these functions:

| Type         | Purpose                           | Phase        |
|--------------|-----------------------------------|--------------|
| `:preload`   | Fetch async data (returns Promise)| Before logic |
| `:input`     | Sample synchronous state          | Before logic |
| `:logic`     | Pure state transformation         | Core         |
| `:transition`| Persist state changes             | After logic  |
| `:effect`    | Execute side effects              | After logic  |

A single `reg` call can attach multiple types to one id:
```cljs
(reg {:id :db :input #(deref app-db) :transition #(reset! app-db %2)})
```

## Consequences

### Benefits
- Clear separation of concerns at each phase of the event loop.
- A single id (like `:db`) encapsulates both reading and writing state.
- The handler map is extensible; new types could be added without changing the
  registration API.

### Trade-offs
- Five types is more to learn than re-frame's model. The distinction between
  `:preload` and `:input`, or `:transition` and `:effect`, requires
  understanding the event loop phases.
- All types share the `:handlers` map, so `dispatch-fn` is used to look up
  handlers in contexts where "dispatch" isn't really happening (inputs,
  transitions). This overloads the concept.
- The `:log` key on handlers (used in `do-log`) adds a sixth implicit type
  that mixes logging concerns into the handler registry.
