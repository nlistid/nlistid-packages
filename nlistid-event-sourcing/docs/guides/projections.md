# Projections

A projector builds a projection — a queryable read model — per stream, for views across streams (lists, counts,
search). Like a snapshot it is derived data: rebuild it by replaying the streams.

```csharp
public interface IProjector<T>
    where T : IEntity
{
    string AggregateType { get; }          // events of other aggregate types are a failure

    T Apply(T model, IEvent @event);       // must not fail; the projector sets Id and Version
}

public interface IProjectionRepository
{
    Task<IResult<T?>> GetAsync<T>(Guid aggregateId, CancellationToken cancellationToken)
        where T : IEntity;

    Task<IResult<long>> SaveAsync<T>(T model, long expectedVersion, CancellationToken cancellationToken)
        where T : IEntity;
}
```

- `SaveAsync` — writes only if the stored model is still at `expectedVersion` (0 = none stored);
  otherwise a conflict failure. Unlike a snapshot, a lost race is reported, not ignored.
- Querying across read models is store-specific; services read the store directly.

Derive from `BaseProjector<T>`, one per stream. Like `BaseAggregate<T>` it does no I/O: the caller
loads, projects and saves. The read model is stored as-is, so it can be queried directly.

```csharp
public abstract class BaseProjector<T> : IProjector<T>
    where T : class, IEntity, new()
{
    protected BaseProjector(Guid aggregateId, T? model = null);   // null: starts from new T { Id = aggregateId }

    public abstract string AggregateType { get; }
    public abstract T Apply(T model, IEvent @event);

    public T Model { get; }
    public long Version { get; }
    public long OriginalVersion { get; }                    // expected version for the next save
    public bool HasChanges { get; }

    public IResult<bool> Project(IEnumerable<IEvent> events);  // true = something applied
}
```

```csharp
public sealed class EnquiryRowProjector(Guid id, EnquiryRow? model)
    : BaseProjector<EnquiryRow>(id, model)
{
    public override string AggregateType => "Enquiry";

    public override EnquiryRow Apply(EnquiryRow model, IEvent @event) => @event switch
    {
        EnquiryReceived received => model with { Subject = received.Subject },
        _                        => model
    };
}

var stored = await projections.GetAsync<EnquiryRow>(id, ct);
var projector = new EnquiryRowProjector(id, stored.Value);

var result = projector.Project(events);   // conflict failure on a gap: read the stream after projector.Version

if (result.IsSuccess && projector.HasChanges)
{
    await projections.SaveAsync(projector.Model, projector.OriginalVersion, ct);
}
```

- Every event must be the next version (`Version + 1`, then +1 each). An event already applied, or one
  after a gap, is a conflict failure and nothing is applied: read the stream after `Version` with
  `ReadStreamAsync` and project that. A redelivered event therefore fails; treat a conflict whose
  event is at or below `projector.Version` as already done.
- A lost save race is a conflict failure from `SaveAsync`: load again and project again.
- Build a new projector for each update; don't reuse one after saving.
- The projector sets `Id` and `Version` after every `Apply`.
- The handler routes: pass only this stream's events. An event of another aggregate type or stream
  is an unexpected failure from `Project`, and nothing is applied.
- The constructor throws `InvalidOperationException` if the read model is for another stream.
