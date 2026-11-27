# NlistId.Results

Return a `Result<T>` instead of throwing for expected failures, then map it to an HTTP response.

## Packages

| Project | Holds | References |
|---|---|---|
| `NlistId.Results.Abstractions` | `IResult<T>` | nothing |
| `NlistId.Results` | `Result<T>`, the failure types, `ResultFactory`, and `ResultExtensions` on `IResult<T>`: `Bind`, `Map`, `OnSuccess`, `OnFailure` | Abstractions |
| `NlistId.Results.AspNetCore` | `HttpResultFactory`, which maps results to RFC 9457 problem-details responses | Results, Abstractions |
| `NlistId.Results.Test` | MSTest BDD tests for `ResultExtensions` and `HttpResultFactory`. Not packed. | Results, Abstractions, AspNetCore |

## Install

Add the GitHub Packages source first, as in the root [Install](../README.md#install). Then:

```bash
dotnet add package NlistId.Results              # in a library
dotnet add package NlistId.Results.AspNetCore   # in an ASP.NET Core app; brings in the other two
```

Up to 2.1.2 the packages were `Nlist.Results`, `Nlist.Results.Abstractions` and
`Nlist.Results.AspNetCore`; 2.2.0 is 2.1.2 under the new ids. To move, change the
`PackageReference` and `using` to `NlistId.Results`. The old versions stay on GitHub Packages until
every consumer has moved.

## Use

```csharp
var factory = new HttpResultFactory();

app.MapGet("/users/{id}", (int id) => factory.ToHttpResult(
    GetUser(id)                                       // IResult<User>
        .Bind(user => ValidateUser(user))             // returns IResult<User>
        .Map(user => new UserDto(user.Id, user.Name)) // returns UserDto
));
```

A success becomes JSON. A failure becomes an [RFC 9457](https://www.rfc-editor.org/info/rfc9457/)
problem details response with the failure's status, e.g. `NotFoundFailureResult<T>` gives 404 and
`ValidationFailureResult<T>` gives 422 with its `errors`.

## Maintain it

From this folder:

- Build: `cd src/NlistId.Results.AspNetCore && dotnet build`, which builds `NlistId.Results` and
  `.Abstractions` too.
- Test: `cd src/NlistId.Results.Test && dotnet test`.
- Check style: `dotnet format --verify-no-changes` in each library project before committing.
- Release: `/release-package results <version>`, which bumps `VERSION` in
  [`nlistid-results.yml`](../.github/workflows/nlistid-results.yml).

## Docs

- [NlistId.Results](docs/guides/results.md): the result types, creating, chaining, logging and matching
- [NlistId.Results.AspNetCore](docs/guides/aspnetcore.md): HTTP mapping and example responses
- [SPEC.md](docs/SPEC.md): the rules the code must keep
