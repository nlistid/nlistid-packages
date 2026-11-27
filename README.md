# Nlist ID Packages

Small, focused .NET 10 libraries for building Nlist ID's services, published to GitHub Packages.
All three families are released and in use.

## Packages

| Family | Packages | What it does |
|---|---|---|
| [`nlistid-results/`](nlistid-results/README.md) | `NlistId.Results`, `.Abstractions`, `.AspNetCore` | Typed results instead of exceptions for expected failures, mapped to problem-details HTTP responses |
| [`nlistid-telemetry/`](nlistid-telemetry/README.md) | `NlistId.Telemetry`, `.Abstractions` | One injectable `ITelemetry<T>` for tracing, logging, metrics, baggage and correlation, built on OpenTelemetry |
| [`nlistid-event-sourcing/`](nlistid-event-sourcing/README.md) | `NlistId.EventSourcing`, `.Abstractions` | Event-sourced aggregates and deciders, snapshots, projections and repository interfaces |

Each family builds, versions and publishes on its own. Its README has its install line, a sample
and how to build, test and release it.

## What you need

- The .NET 10 SDK.
- The GitHub CLI (`gh`), signed in to an account that can read the `nlistid` packages.

## Install

GitHub Packages needs a token with `read:packages`, even to restore.

```bash
gh auth refresh -h github.com -s read:packages

export GITHUB_PACKAGES_TOKEN=$(gh auth token)

dotnet nuget add source https://nuget.pkg.github.com/nlistid/index.json \
  --name github-nlist --username <github-user> \
  --password '%GITHUB_PACKAGES_TOKEN%' --store-password-in-clear-text

# NuGet stores `%GITHUB_PACKAGES_TOKEN%` and reads the variable at restore.

dotnet add package NlistId.Results
```

## Maintain it

- Build and test: there is no root `.sln`, so run `dotnet` from a project's folder, as each
  family's README shows.
- Release: bump `VERSION` in the family's workflow, `.github/workflows/nlistid-<family>.yml`, in a
  pull request. Merging it publishes every package in the family at that version. A run without a
  bump tests and packs but publishes nothing, as that version is already there.
  [`/release-package <family> <version>`](.claude/skills/release-package/SKILL.md) walks through it.

## Docs

- [ARCHITECTURE.md](docs/ARCHITECTURE.md): how the families reference each other, and how they
  are built and published.
