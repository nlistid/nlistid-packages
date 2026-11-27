# Architecture

## Projects

- `NlistId.Telemetry.Abstractions` is interfaces only: `ITelemetry<T>` and the five it exposes,
  `ITelemetryTracing<T>` (whose spans are `ITelemetryTracingSpan`), `ITelemetryLogger<T>`,
  `ITelemetryMetrics`, `ITelemetryBaggage` and `ITelemetryCorrelation`. It must never take a
  `PackageReference`.
- `NlistId.Telemetry` implements them, one folder per part: `Tracing/`, `Logging/`, `Metrics/`,
  `Baggage/`, `Correlation/`. `Telemetry.cs` joins the five into `ITelemetry<T>`, and
  `TelemetryServiceCollectionExtensions.cs` holds `services.AddTelemetry()`. It references
  `OpenTelemetry.Api` and the `Microsoft.Extensions` dependency injection and logging abstractions.
- It packs its own `README.md`, the package's usage guide, through `PackageReadmeFile`.

## Tests

`NlistId.Telemetry.Test` runs against the real `Tracer`, `Meter` and `OpenTelemetry.Baggage`, not
fakes of them. The tests run in parallel, so a scenario that touches process-wide state keeps to
its own: tracing names its `ActivitySource` once per scenario, metrics listens to its own `Meter`
only, and baggage clears `OpenTelemetry.Baggage` first.
