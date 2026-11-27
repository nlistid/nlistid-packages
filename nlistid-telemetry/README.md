# NlistId.Telemetry

An OpenTelemetry-shaped facade: one injectable `ITelemetry<T>` for tracing, logging, metrics,
baggage and correlation.

## Packages

| Project | Holds | References |
|---|---|---|
| `NlistId.Telemetry.Abstractions` | `ITelemetry<T>` and the interfaces behind it | nothing |
| `NlistId.Telemetry` | The implementation and `services.AddTelemetry()` | Abstractions, `OpenTelemetry.Api` |
| `NlistId.Telemetry.Test` | MSTest BDD tests for every part and `AddTelemetry`. Not packed. | Both of the above |

## Install

Add the GitHub Packages source first, as in the root [Install](../README.md#install). Then:

```bash
dotnet add package NlistId.Telemetry   # brings in NlistId.Telemetry.Abstractions
```

Up to 2.1.2 the packages were `Nlist.Telemetry` and `Nlist.Telemetry.Abstractions`; 2.2.0 is 2.1.2
under the new ids. To move, change the `PackageReference` and `using` to `NlistId.Telemetry`. The
old versions stay on GitHub Packages until every consumer has moved.

## Use

```csharp
services.AddTelemetry();

services.AddOpenTelemetry()
    .WithTracing(tracing => tracing.AddSource(TelemetryTracer.Name))
    .WithMetrics(metrics => metrics.AddMeter(TelemetryTracer.Name));
```

```csharp
using var span = telemetry.Tracing.StartActiveSpan("Send enquiry emails");
telemetry.Logging.LogInformation("Sending emails.");
telemetry.Metrics.IncrementCounter("EnquiryEmailsSent");
```

The [package's README](src/NlistId.Telemetry/README.md) covers setup and each part in full.
`dotnet pack` ships it with the package.

## Maintain it

From this folder:

- Build: `cd src/NlistId.Telemetry && dotnet build`, which builds `NlistId.Telemetry.Abstractions`
  too.
- Test: `cd src/NlistId.Telemetry.Test && dotnet test`.
- Release: `/release-package telemetry <version>`, which bumps `VERSION` in
  [`nlistid-telemetry.yml`](../.github/workflows/nlistid-telemetry.yml).

## Docs

- [ARCHITECTURE.md](docs/ARCHITECTURE.md): how the projects are laid out, and how the tests keep
  apart in parallel
