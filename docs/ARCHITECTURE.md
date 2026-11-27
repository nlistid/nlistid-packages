# Architecture

## References

- Inside a family, projects use plain `ProjectReference`s. `dotnet pack` turns each into a NuGet
  dependency at the version being packed, so no version is pinned by hand.
- Across families, a project takes a `PackageReference` to a published version, never a
  `ProjectReference` into another family's folder. Only `nlistid-event-sourcing` does this today,
  on `NlistId.Results` and `NlistId.Results.Abstractions`.
- There is no `nuget.config`. The GitHub Packages source is added on each machine (the README's
  Install) and, in CI, by the build action.

## Build and publish

Each family has two workflows in `.github/workflows/`: `nlistid-<family>.yml` publishes it, and
`nlistid-<family>-pr.yml` checks every pull request. Both build through the composite action
`.github/actions/build-package/`, which adds the GitHub Packages source, runs the family's test
project, then packs each package at the version given. The pull request check packs at
`0.0.0-pr` and pushes nothing.

Each package's `PackageId`, `Description`, `Authors` and `RepositoryUrl` are set in its `.csproj`.
Passed on the command line instead, `-p:PackageId` reaches the `ProjectReference` restore graph
and NuGet fails with `Ambiguous project name`.

## Shared settings

- The root `.editorconfig` and `.gitignore` apply to every project.
- Code style: camelCase private fields with no `_`, `this.` qualification, file-scoped namespaces.
  Only `nlistid-results` enforces it at build, with StyleCop and `EnforceCodeStyleInBuild`;
  elsewhere match the surrounding code.
