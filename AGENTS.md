# AGENTS.md

Guidance for AI coding agents working in this .NET 10 monorepo.

## Repository Structure

```text
.
├── .github/workflows/   # GitHub Actions CI/CD workflows
├── apps/
│   ├── api/             # ASP.NET Core Web API apps
│   ├── client/          # API client apps
│   ├── lib/             # Shared libraries and .NET contracts/DTOs
│   ├── orchestration/   # .NET Aspire orchestration
│   ├── ui/              # Frontend / UI applications
│   └── worker/          # Background worker service apps
├── docs/                # Architecture, runbooks, and project documentation
├── infra/               # Infrastructure as Code, deployment manifests, and local env configs
├── scripts/             # Build, automation, and developer utility scripts
├── specs/               # API specifications (OpenAPI, TypeSpec, schemas)
├── tests/               # Automation, integration, and performance test projects
├── Directory.Build.props
├── Directory.Packages.props
└── global.json
```

- **Unit tests** live alongside the project they test under `apps/` (for example, `apps/api/<App>/<App>.Tests/` or a sibling `<Project>.Tests` folder next to `<Project>`).
- **Non-unit tests** (end-to-end, automation, load/performance) live under `tests/`.
- **Solution files** use the `.slnx` format and are typically scoped per area (living alongside the relevant app or area rather than one monolithic root solution).
- Remove `.gitkeep` files from folders when no longer needed.

## Toolchain & Build Configuration

- **SDK**: Pinned in `global.json` (`10.0.100`, `rollForward: latestMinor`).
- **Shared MSBuild defaults** (`Directory.Build.props`):
  - `TargetFramework`: `net10.0`
  - `Nullable`: `enable`
  - `ImplicitUsings`: `enable`
  - `AnalysisLevel`: `latest-recommended`
  - `EnforceCodeStyleInBuild`: `true`
  - `TreatWarningsAsErrors`: `true`
  - `Deterministic`: `true` (with `ContinuousIntegrationBuild` enabled when `GITHUB_ACTIONS=true`)
  - Test projects (`IsTestProject=true`) automatically set `IsPackable=false`.
- **Central Package Management** (`Directory.Packages.props`):
  - `ManagePackageVersionsCentrally` and `CentralPackageTransitivePinningEnabled` are `true`.
  - Declare all NuGet package versions in `Directory.Packages.props` using `<PackageVersion Include="..." Version="..." />`.
  - Do **not** add `Version="..."` attributes to `<PackageReference>` items in individual `.csproj` files.

## Modern .NET 10 & C# Coding Standards

- **Formatting & Style (`.editorconfig`)**:
  - Follow the repository `.editorconfig` rules (`EnforceCodeStyleInBuild` is enabled).
  - Prefer explicit types over `var`, file-scoped namespaces (`namespace MyApp.Feature;`), and primary constructors where they improve clarity.
  - Naming: `_camelCase` for private instance fields, `s_camelCase` for private static fields, `PascalCase` for `static readonly` and `const` fields.
  - Prefer modern C# features where appropriate: collection expressions (`[]`), `required` members, `init` setters, pattern matching, and raw string literals.
  - Respect nullable reference type annotations; avoid the null-forgiving operator (`!`) unless invariant safety is clear and documented.
  - Use `System.Threading.Lock` instead of `lock (new object())` for in-process synchronization.
- **Async & Concurrency**:
  - Accept and propagate `CancellationToken` in async methods, Minimal API endpoints, and `BackgroundService` / `IHostedService` implementations.
  - Never block on async code (`.Result`, `.Wait()`, `.GetAwaiter().GetResult()`) or use `async void`.
- **Dependency Injection, Configuration & Logging**:
  - Register services cleanly via extension methods (e.g., `Add<Feature>Services(this IServiceCollection services, ...)`).
  - Bind configuration using the Options pattern (`IOptions<T>`, `IOptionsMonitor<T>`) with `ValidateDataAnnotations()` and `ValidateOnStart()`.
  - Use `ILogger<T>` with structured message templates; prefer `[LoggerMessage]` source-generated logging on hot paths.
- **ASP.NET Core & Worker Services**:
  - Prefer Minimal APIs or well-factored endpoint groups for new Web APIs, with explicit request/response DTOs and OpenAPI metadata.
  - Use `IHttpClientFactory` (or typed/generated clients) with standard resilience handlers rather than instantiating `HttpClient` directly.
  - Ensure worker services handle graceful shutdown via `stoppingToken`.

## Testing Standards

- Use **xUnit** (`xunit.v3`) as the testing framework, with `FluentAssertions` and `Moq` (plus `Microsoft.AspNetCore.Mvc.Testing` and `Testcontainers` for integration/API tests) as declared in `Directory.Packages.props`.
- Write unit tests for business logic, validation, and edge cases alongside each project in `apps/`.
- Keep unit tests fast, deterministic, and isolated from external network/filesystem dependencies.
- Place cross-service, automation, or performance test suites under `tests/`.

## Common Commands

Run these from the repository root (or target a specific `.csproj` / `.slnx`):

```bash
dotnet restore
dotnet build
dotnet test
dotnet format --verify-no-changes
```

## Agent Workflow Expectations

- Inspect existing project files and `Directory.Packages.props` before adding new dependencies or projects.
- Reuse shared abstractions and contracts in `apps/lib/` rather than duplicating types across services.
- Keep edits scoped to the task; do not modify unrelated projects or shared build configuration without reason.
- Run `dotnet build` and `dotnet test` for affected projects after making code changes, and report the exact results.
