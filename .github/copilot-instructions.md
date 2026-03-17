# .NET / C# Development

## Build

Default:
```bash
dotnet build <changed>.csproj -c Debug
```

Full solution: `dotnet build -c Release`

## No excuses

Build fails → 99% YOUR previous change broke it.
DON'T say "pre-existing", "infra issue", "unrelated".
DO `git clean -xfd artifacts bin obj` and rebuild.

## Test

spot check: `dotnet test <proj> --filter "FullyQualifiedName~<pattern>" -c Debug`
full run: `dotnet test -c Release`
update baselines: `TEST_UPDATE_BSL=1 <test command>`

## Spotcheck tests

- Find new tests for bugfix/feature
- Find preexisting tests in the same area
- Run siblings/related

## Final validation (Copilot Coding Agent only)

Before submitting: `dotnet build -c Release && dotnet test -c Release`

## Rules

Public API change → update public API surface
New diagnostic/warning → update resource files
After code changes → run `dotnet format`
When fully done → write release notes (see release-notes skill)
