# Starter Repository

A flexible .NET 10 repository structure for ASP.NET Core Web APIs, background worker services, shared libraries, and contracts.

## Prerequisites

- **[.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0)**: Version `10.0.100` or later (`latestMinor` roll-forward is configured in [global.json](global.json)).
- **Container Engine** *(optional)*: Docker, Rancher Desktop, etc.; required if running integration tests that use [Testcontainers](https://dotnet.testcontainers.org/).
- **IDE / Editor**: Visual Studio 2026+, JetBrains Rider, or VS Code with the C# Dev Kit or ReSharper.

Verify your .NET SDK installation:

```bash
dotnet --version
```

## Getting Started

1. **Clone the repository:**

   ```bash
   git clone <repository-url>
   cd <repo-name>
   ```

2. **Restore dependencies:**

   ```bash
   dotnet restore
   ```

3. **Build and run tests:**

   ```bash
   dotnet build
   dotnet test
   ```

   > Because this repo uses scoped `.slnx` solution files under `apps/`, you can also target a specific app's solution or project directly:
   >
   > ```bash
   > dotnet build apps/api/<AppName>/<AppName>.slnx
   > dotnet test apps/api/<AppName>/<AppName>.slnx
   > ```

4. **Verify formatting and code style:**

   ```bash
   dotnet format --verify-no-changes
   ```

## Repository Structure

```text
.
├── .github/workflows/   # GitHub Actions CI/CD workflows
├── apps/
│   ├── api/             # ASP.NET Core Web API apps
│   ├── client/          # API client libraries/apps
│   ├── lib/             # Shared libraries and .NET contracts/DTOs
│   ├── orchestration/   # .NET Aspire orchestration
│   ├── ui/              # Frontend / UI applications
│   └── worker/          # Background worker service apps
├── docs/                # Architecture, runbooks, and engineering standards
├── infra/               # Infrastructure as Code, deployment manifests, and local env configs
├── scripts/             # Build, automation, and developer utility scripts
├── specs/               # API specifications (OpenAPI, TypeSpec, schemas)
├── tests/               # Cross-service integration, E2E, and performance tests
├── AGENTS.md            # Guidance for AI coding agents
├── Directory.Build.props
├── Directory.Packages.props
└── global.json
```

## Build & Package Management

- **Shared MSBuild defaults** ([Directory.Build.props](Directory.Build.props)): Targets `net10.0` with `Nullable`, `ImplicitUsings`, `EnforceCodeStyleInBuild`, `TreatWarningsAsErrors`, and `latest-recommended` analyzers enabled across all projects.
- **Central Package Management** ([Directory.Packages.props](Directory.Packages.props)): All NuGet package versions are declared centrally. Do **not** specify `Version="..."` on `<PackageReference>` items in individual `.csproj` files.
- **Code Style** ([.editorconfig](.editorconfig)): Enforced during build; run `dotnet format` before committing.

## Documentation & Standards

- [Naming & Project Structure Standards](docs/standards.md)
- [AI Agent Guidance](AGENTS.md)
