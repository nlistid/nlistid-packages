# Aggregates and deciders

Two styles of event-sourced aggregate: a mutable aggregate derived from `BaseAggregate<T>`, or a
pure decider that implements `IDecider<TCommand, TEvent, TState>`.

## Aggregate

Derive from `BaseAggregate<T>`. Business rules call `Raise`; `Apply` changes state.

```csharp
public abstract class BaseAggregate<T> : IAggregate<T>
    where T : IEntity, new()
{
    protected BaseAggregate(Guid id);

    public abstract string AggregateType { get; }
    public abstract string AggregateDataType { get; }

    public bool TryRestore(ISnapshot<T> snapshot);   // false = stale snapshot, replay the stream
    public void Load(IEnumerable<IEvent> history);
    public void ClearUncommittedChanges();

    protected IResult<IEvent> Raise(IEvent @event);  // check, apply, add to UncommittedChanges
    protected abstract void Apply(IEvent @event);    // must not fail; also runs on replay
}
```

Load and save it as in the README's [Use](../../README.md#use).

`Raise`, `Load` and `TryRestore` throw `InvalidOperationException` when an event or snapshot is
for another stream, has an empty `EventId`, or is not the next version.

## Decider

```csharp
public interface IDecider<in TCommand, TEvent, TState>
    where TEvent : IEvent
    where TState : IEntity
{
    TState InitialState { get; }

    IResult<IReadOnlyList<TEvent>> Decide(TCommand command, TState state);

    TState Evolve(TState state, TEvent @event);

    bool IsTerminal(TState state) => false;

    TState Fold(IEnumerable<TEvent> history) => this.Fold(this.InitialState, history);

    TState Fold(TState state, IEnumerable<TEvent> history) => history.Aggregate(state, this.Evolve);
}
```

- `Decide` — business rules, no I/O. A broken rule is a failed result; an empty list means no change.
- `Evolve` — applies a fact and sets `Version` to the event's version. Must not fail.
- `IsTerminal` — true once the stream accepts no more commands.

A handler loads, folds, decides and appends. Hold the decider as `IDecider<…>`: `Fold` and
`IsTerminal` are default interface methods, so they aren't visible on the concrete class.

```csharp
IDecider<AccountCommand, AccountEvent, AccountState> decider = new AccountDecider();

var history = await events.ReadStreamAsync(id, 0, ct);
var state = decider.Fold(history.Value!.Cast<AccountEvent>());

var decision = decider.Decide(command, state);
if (!decision.IsSuccess)
{
    return decision;
}

await events.AppendAsync(id, state.Version, decision.Value!, ct);
```

Example (from `IDecider.cs`):

```csharp
public sealed record AccountState(bool IsOpen, decimal Balance) : IEntity
{
    public Guid Id { get; set; }
    public long Version { get; set; }
}

public sealed class AccountDecider : IDecider<AccountCommand, AccountEvent, AccountState>
{
    public AccountState InitialState => new(false, 0m);

    public IResult<IReadOnlyList<AccountEvent>> Decide(AccountCommand command, AccountState state) => command switch
    {
        OpenAccount open         => Open(open, state),
        DepositMoney deposit     => Deposit(deposit, state),
        WithdrawMoney withdrawal => Withdraw(withdrawal, state),
        _                        => throw new ArgumentException($"Unknown command {command.GetType().Name}")
    };

    public AccountState Evolve(AccountState state, AccountEvent @event) => (@event switch
    {
        AccountOpened            => state with { IsOpen = true },
        MoneyDeposited deposited => state with { Balance = state.Balance + deposited.Amount },
        MoneyWithdrawn withdrawn => state with { Balance = state.Balance - withdrawn.Amount },
        _                        => state
    }) with { Version = @event.Version };
}
```
