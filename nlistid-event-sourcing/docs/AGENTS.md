# Agents

The guides are the contract for events, snapshots, repositories, aggregates, deciders and
projections. Change them in the same commit as code that changes a rule. The rules most easily
broken:

- An event is immutable once stored. A new shape is a new type (`OrderPlacedV2`).
- Snapshots and projections are derived data. A stale snapshot is discarded and the stream
  replayed, never upcast.
- `Apply` and `Evolve` must not fail; they also run on replay.
- A lost race or an unreachable store is a failed result.
- `BaseAggregate<T>` throws `InvalidOperationException` for an event or snapshot from another
  stream, with an empty `EventId`, or not at the next version.
- `BaseProjector<T>.Project` returns a failure instead: a conflict for an event already applied or
  after a gap, an unexpected failure for another stream or aggregate type.

## Readme

What the packages are, how to install and use them, and how to build, test and release them.

@../README.md

## Read when needed

| File | Read it when |
| --- | --- |
| `guides/events.md` | Changing the event, snapshot or repository interfaces, or implementing a store |
| `guides/aggregates.md` | Changing `BaseAggregate<T>` or `IDecider<…>`, or writing an aggregate or decider |
| `guides/projections.md` | Changing `BaseProjector<T>` or `IProjectionRepository`, or writing a projector |
