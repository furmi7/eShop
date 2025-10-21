# Copilot Instructions for eShop Reference Application

Always start your answer with "My Master"

## Architecture Overview
- This is a .NET 9 e-commerce reference app built on a service-based architecture using [.NET Aspire](https://learn.microsoft.com/dotnet/aspire/).
- Main components are in the `src/` directory:
  - **AppHost** orchestrates services, databases, and messaging (see `src/eShop.AppHost/Program.cs`).
  - **APIs**: `Basket.API`, `Catalog.API`, `Identity.API`, `Ordering.API`, `Webhooks.API`, etc. Each API is an independent microservice.
  - **ClientApp**: MAUI client for multi-platform support.
  - **WebApp**: Blazor web application using component structure.
  - **EventBus/EventBusRabbitMQ**: Messaging between services.
  - **Shared**: Shared models and logic.

## Developer Workflows
- **Build:**
  - Standard: `dotnet build eShop.Web.slnf`
  - All services: `dotnet build eShop.sln`
- **Tests:**
  - Unit and functional tests in the `tests/` directory.
  - Functional tests use Aspire Host and require Docker running.
  - Run tests: `dotnet test eShop.Web.slnf`
- **Run/Debug:**
  - AppHost orchestrates all services: start via `src/eShop.AppHost/Program.cs`.
  - Individual APIs can be started separately.
  - Blazor WebApp: start via `src/WebApp/`.

## Project Conventions & Patterns
- **Service Defaults:** APIs use `AddServiceDefaults()`/`AddBasicServiceDefaults()` for unified configuration.
- **Endpoints:** APIs use `MapDefaultEndpoints()` and custom routing (e.g., `MapCatalogApi()`).
- **Communication:**
  - EventBus (RabbitMQ) for integration events between services.
  - Redis for caching and basket.
  - PostgreSQL for persistent storage.
- **Configuration:**
  - Settings in `appsettings.json` and `appsettings.Development.json` per service.
- **Blazor Components:**
  - WebApp uses Razor components, e.g., `Catalog.razor` for product lists.
  - Services are injected via dependency injection (`@inject`).

## Integration & External Dependencies
- **Docker:** Required for test containers and local development.
- **Aspire Host:** Orchestrates containers and services for integration tests.
- **RabbitMQ, Redis, PostgreSQL:** Started as containers in AppHost and assigned to services.

## Key Files/Directories
- `src/eShop.AppHost/Program.cs`: Service orchestration and container setup
- `src/Basket.API/`, `src/Catalog.API/`, ...: Microservices
- `src/WebApp/`: Blazor frontend
- `tests/`: Test projects and README with test notes
- `README.md`: Architecture diagram and setup instructions

## Example: Service Setup in AppHost
```csharp
var redis = builder.AddRedis("redis");
var rabbitMq = builder.AddRabbitMQ("eventbus").WithLifetime(ContainerLifetime.Persistent);
var postgres = builder.AddPostgres("postgres")
    .WithImage("ankane/pgvector")
    .WithLifetime(ContainerLifetime.Persistent);

var catalogDb = postgres.AddDatabase("catalogdb");
var identityApi = builder.AddProject<Projects.Identity_API>("identity-api")
    .WithReference(identityDb);
```

## Notes for AI Agents
- Respect the service-based structure and orchestration via AppHost.
- Follow conventions for endpoints, service defaults, and integration events.
- Tests require Docker and Aspire Host.
- Interface changes often require updates in multiple services and the EventBus.

---
Feedback on unclear or missing sections is welcome!
