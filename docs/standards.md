# Naming & Project Structure Standards

This document defines the conventions for organizing solutions, projects, folders, and symbols in this repository structure.

## 1. Solution & Project Granularity

Because the Web APIs and worker services in this repository are relatively small in scope, **prefer a single application project per service** paired with a sibling unit test project, rather than splitting each service into multiple layered projects (`Domain`, `Application`, `Infrastructure`, etc.).

- **Use folder namespaces** inside each app project to separate concerns cleanly without `.csproj` and project-reference overhead.
- **Reuse shared projects** in `apps/lib/` (cross-cutting utilities, shared infrastructure, and shared .NET DTOs/contracts) instead of duplicating code or over-splitting individual apps.
- **Keep `specs/` language-agnostic** for API specifications (OpenAPI, TypeSpec, AsyncAPI, schemas) rather than `.NET` class libraries.
- **When to extract a project:** Only split code out of an app into `apps/lib/` when it needs to be shared by multiple apps or published independently.

### Scoped Solutions (`.slnx`)

Each app or library area owns a scoped `.slnx` solution file alongside its directory:

```text
apps/api/OrdersApi/
├── OrdersApi.slnx
├── OrdersApi/
│   ├── OrdersApi.csproj
│   └── ...
└── OrdersApi.Tests/
    ├── OrdersApi.Tests.csproj
    └── ...
```

## 2. Standard Folder Layouts

### ASP.NET Core Web APIs (`apps/api/<AppName>/`)

Organize small-to-medium Web APIs using **feature folders (vertical slices)** for endpoints and handlers, plus dedicated folders for shared app-level infrastructure and configuration:

```text
apps/api/<AppName>/<AppName>/
├── Configuration/          # Strongly-typed IOptions<T> classes with DataAnnotations
├── Extensions/             # DI registration (Add<App>Services) and WebApplication setup
├── Features/               # Vertical slices grouped by domain capability
│   └── <FeatureName>/
│       ├── <Action>Endpoint.cs
│       ├── <FeatureName>Models.cs
│       └── <FeatureName>Service.cs
├── Infrastructure/         # Persistence (DbContext/repos), external HTTP clients, messaging
├── Middleware/             # Custom middleware or exception handlers (when needed)
├── Program.cs
├── appsettings.json
└── appsettings.Development.json
```

> **Tiny APIs (1–3 endpoints):** A flat `Endpoints/`, `Services/`, and `Models/` layout is acceptable if a `Features/` hierarchy would feel overly nested.

### Background Worker Services (`apps/worker/<AppName>/`)

Keep `BackgroundService` implementations thin—focused on scheduling, scoping (`IServiceScopeFactory`), and `stoppingToken` handling—while placing business logic in testable handlers/services:

```text
apps/worker/<AppName>/<AppName>/
├── Configuration/          # Strongly-typed IOptions<T> classes with DataAnnotations
├── Extensions/             # DI registration (Add<App>Services)
├── Handlers/               # Core processing/business logic (decoupled from BackgroundService)
├── Infrastructure/         # External clients, queues, persistence
├── Workers/                # BackgroundService / IHostedService host loops
│   └── <JobName>Worker.cs
├── Program.cs
├── appsettings.json
└── appsettings.Development.json
```

### Shared Libraries, Specs, Clients, Infra, Scripts, and Tests

- **`apps/lib/<LibName>/`**: Reusable `.NET` libraries (`<LibName>/<LibName>.csproj` + `<LibName>.Tests/<LibName>.Tests.csproj`) and shared `.NET` DTOs/message contracts (`apps/lib/<Domain>.Contracts/`).
- **`apps/client/<ClientName>/`**: Typed HTTP or SDK clients (hand-written or generated from `specs/`) consuming internal/external APIs.
- **`apps/orchestration/<AppName>/`**: .NET Aspire orchestration (AppHost, ServiceDefaults).
- **`apps/ui/<AppName>/`**: Frontend / UI applications.
- **`specs/<SpecName>/`**: Language-agnostic API specifications and API-first tooling (OpenAPI, TypeSpec, AsyncAPI, JSON schemas).
- **`infra/`**: Infrastructure as Code, deployment manifests, and local environment configuration.
- **`scripts/`**: Repository automation, build, and developer utility scripts.
- **`tests/<SuiteName>.Tests/`**: Cross-service integration, end-to-end, automation, and load/performance test suites.

## 3. Naming Conventions

### Repositories, Folders, and Projects

| Item | Convention | Example |
| :--- | :--- | :--- |
| Top-level category folders | lowercase | `apps/api`, `apps/worker`, `apps/lib`, `apps/orchestration`, `apps/ui`, `specs`, `infra`, `scripts`, `tests` |
| App / library root folder | `PascalCase` | `apps/api/OrdersApi/`, `apps/worker/InvoiceSyncWorker/` |
| Solution file | `<AppName>.slnx` | `OrdersApi.slnx` |
| Main project | `<AppName>.csproj` | `OrdersApi.csproj` |
| Unit test project | `<ProjectName>.Tests.csproj` | `OrdersApi.Tests.csproj` |
| Cross-service test project | `<Area>.<TestType>Tests.csproj` | `Orders.IntegrationTests.csproj` |

### Namespaces, Files, and C# Symbols

- **Namespaces**: Must match the folder path (`dotnet_style_namespace_match_folder = true` in `.editorconfig`) and use file-scoped syntax (`namespace OrdersApi.Features.Orders;`).
- **Files**: One primary public type per file; file name matches the type name (e.g., `CreateOrderEndpoint.cs`).
- **Role Suffixes**: Use clear suffixes for architectural roles:
  - `*Endpoint` / `*Endpoints` (Minimal API endpoint definitions)
  - `*Worker` (`BackgroundService` / `IHostedService`)
  - `*Handler` / `*Service` (application/business logic)
  - `*Options` (configuration classes bound via `IOptions<T>`)
  - `*Client` (typed `HttpClient` wrappers)
  - `*Tests` (xUnit test classes, e.g., `CreateOrderEndpointTests`)
- **Fields & Locals** (enforced by `.editorconfig`):
  - Private instance fields: `_camelCase`
  - Private static fields: `s_camelCase`
  - `static readonly` and `const` fields: `PascalCase`
  - Prefer explicit types over `var`
