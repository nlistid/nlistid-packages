# Events, snapshots and repositories

The types a stream is made of, and the repositories a store implements. All are in
`NlistId.EventSourcing.Abstractions`.

## Events and state

```csharp
public interface IEventIdentity
{
    Guid AggregateId { get; }     // the stream
    string AggregateType { get; } // e.g. "Enquiry"
    long Version { get; }         // starts at 1, no gaps
}

public interface IEvent : IEventIdentity
{
    Guid EventId { get; }
    string EventType { get; }     // e.g. "EnquiryReceived"
    IEventContext EventContext { get; }
}

public interface IEventContext
{
    string UserId { get; }
    string CorrelationId { get; }
    DateTimeOffset OccurredAt { get; }
    Guid? CausationId { get; }    // EventId of the event that caused this one
}

public interface IEntity
{
    Guid Id { get; set; }
    long Version { get; set; }    // 0 = no events yet
}

public interface ISnapshot<T> : IEventIdentity
    where T : IEntity
{
    Guid EventId { get; }
    string AggregateDataType { get; } // e.g. "EnquiryV1"
    T AggregateData { get; }
}
```

- An event is immutable once stored: a new shape is a new type (`OrderPlacedV2`).
- The stream is the source of truth. A snapshot is derived data; losing one only makes the next load slower.
- A snapshot whose `AggregateDataType` isn't the aggregate's current one (e.g. `EnquiryV1` after
  moving to `EnquiryV2`) is discarded and the stream replayed. Snapshots are never upcast.
- `IEntity.Version` is the version of the last event applied; projection and snapshot writes are guarded on it.

## Repositories

```csharp
public interface IEventRepository
{
    Task<IResult<IReadOnlyList<IEvent>>> ReadStreamAsync(
        Guid aggregateId,
        long afterVersion,
        CancellationToken cancellationToken);

    Task<IResult<long>> AppendAsync(
        Guid aggregateId,
        long expectedVersion,
        IReadOnlyList<IEvent> events,
        CancellationToken cancellationToken);
}

public interface ISnapshotRepository
{
    Task<IResult<ISnapshot<T>?>> GetAsync<T>(Guid aggregateId, CancellationToken cancellationToken)
        where T : IEntity;

    Task<IResult<bool>> SaveAsync<T>(ISnapshot<T> snapshot, CancellationToken cancellationToken)
        where T : IEntity;
}
```

- `ReadStreamAsync` — pass `0` for the whole stream, or a snapshot's version. An unknown stream is an empty list.
- `AppendAsync` — all or nothing. Returns a conflict failure if the stream moved past `expectedVersion`.
- `SaveAsync` — ignores a snapshot at an equal or lower version and returns `false`.
- A lost race or an unreachable store is a failed result. Broken input (out-of-order events) throws.
