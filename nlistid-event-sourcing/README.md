# NlistId.EventSourcing

Event-sourced aggregates, in two styles: a mutable aggregate or a pure decider. With them come
snapshots, projections and the repository interfaces a store implements.

## Packages

| Project | Holds | References |
|---|---|---|
| `NlistId.EventSourcing.Abstractions` | `IEvent`, `IEventIdentity`, `ISnapshot<T>`, `IEntity`, `IEventContext`, `IEventRepository`, `ISnapshotRepository`, `IAggregate<T>`, `IDecider<TCommand, TEvent, TState>`, `IProjector<T>`, `IProjectionRepository` | the `NlistId.Results.Abstractions` package |
| `NlistId.EventSourcing` | `BaseAggregate<T>`, `BaseProjector<T>` | Abstractions, the `NlistId.Results` package |
| `NlistId.EventSourcing.Test` | MSTest BDD tests for `BaseAggregate<T>` and `BaseProjector<T>`. Not packed. | `NlistId.EventSourcing`, Abstractions |

## Install

Add the GitHub Packages source first, as in the root [Install](../README.md#install). Then:

```bash
dotnet add package NlistId.EventSourcing   # brings in .Abstractions and NlistId.Results
```

Up to 2.1.1 the packages were `Nlist.EventSourcing` and `Nlist.EventSourcing.Abstractions`, on
`Nlist.Results`; 2.2.0 is 2.1.1 under the new ids, on `NlistId.Results`. To move, change the
`PackageReference` and `using` to `NlistId.EventSourcing`, together with `NlistId.Results`. The old
versions stay on GitHub Packages until every consumer has moved.

## Use

Derive an aggregate from `BaseAggregate<T>`, then load it, call its business methods and append
the events they raised:

```csharp
var aggregate = new EnquiryAggregate(id);

var snapshot = await snapshots.GetAsync<Enquiry>(id, ct);
if (snapshot.Value is { } storedSnapshot)
{
    aggregate.TryRestore(storedSnapshot);   // false = stale snapshot, ignored
}

var history = await events.ReadStreamAsync(id, aggregate.Version, ct);
aggregate.Load(history.Value!);

// ... call business methods that Raise events ...

await events.AppendAsync(id, aggregate.OriginalVersion, aggregate.UncommittedChanges, ct);
aggregate.ClearUncommittedChanges();
```

The decider style and projections are in the [docs](#docs).

## Maintain it

From this folder:

- Build: `cd src/NlistId.EventSourcing && dotnet build`, which builds
  `NlistId.EventSourcing.Abstractions` too. It restores `NlistId.Results` from GitHub Packages, so
  it needs the source from the root [Install](../README.md#install).
- Test: `cd src/NlistId.EventSourcing.Test && dotnet test`.
- Move to a new `NlistId.Results`: release it first, then bump both `PackageReference`s to the same
  version, `NlistId.Results` in `NlistId.EventSourcing` and `NlistId.Results.Abstractions` in
  `NlistId.EventSourcing.Abstractions`.
- Release: `/release-package event-sourcing <version>`, which bumps `VERSION` in
  [`nlistid-event-sourcing.yml`](../.github/workflows/nlistid-event-sourcing.yml).

## Docs

- [Events, snapshots and repositories](docs/guides/events.md): the types a stream is made of, and
  what a store's repositories must do
- [Aggregates and deciders](docs/guides/aggregates.md): `BaseAggregate<T>` and
  `IDecider<TCommand, TEvent, TState>`, with a worked decider
- [Projections](docs/guides/projections.md): read models built with `BaseProjector<T>`
