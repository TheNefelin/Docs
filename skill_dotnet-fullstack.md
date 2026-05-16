# .NET 8/9 + C# - Skill Completo

> Guía completa para desarrollo moderno con C# y .NET.
> Transversal, sin dependencias de proyectos específicos.

> **Nota:** .NET 10 no existe aún. Este skill cubre .NET 8/9 (últimas versiones LTS y Preview).

---

## Tabla de Contenidos

1. [Setup y Configuración](#1-setup-y-configuración)
2. [Arquitectura](#2-arquitectura)
3. [Minimal APIs](#3-minimal-apis)
4. [Entity Framework Core](#4-entity-framework-core)
5. [Dependency Injection](#5-dependency-injection)
6. [Authentication & Authorization](#6-authentication--authorization)
7. [Testing](#7-testing)
8. [Performance](#8-performance)
9. [Background Services](#9-background-services)
10. [gRPC](#10-grpc)
11. [Deployment](#11-deployment)

---

## 1. Setup y Configuración

### 1.1 .NET CLI

```bash
# Install .NET 8 SDK
dotnet --version  # Check version

# Create project
dotnet new web -n MyApp
dotnet new webapi -n MyApi
dotnet new blazor -n MyBlazorApp
dotnet new worker -n MyWorker

# Run
dotnet run
dotnet watch run  # Hot reload

# Build
dotnet build
dotnet publish -c Release
```

### 1.2 csproj Moderno

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    <LangVersion>latest</LangVersion>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.*" />
    <PackageReference Include="Swashbuckle.AspNetCore" Version="6.5.*" />
    <PackageReference Include="FluentValidation.AspNetCore" Version="11.3.*" />
  </ItemGroup>

</Project>
```

### 1.3 Program.cs Minimal

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Middleware pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

---

## 2. Arquitectura

### 2.1 Clean Architecture

```
src/
├── Domain/                 # Entities, value objects, interfaces
│   ├── Entities/
│   ├── ValueObjects/
│   └── Interfaces/
├── Application/            # Use cases, DTOs, interfaces impl
│   ├── UseCases/
│   ├── DTOs/
│   ├── Interfaces/
│   └── Services/
├── Infrastructure/         # EF, external services
│   ├── Persistence/
│   ├── Repositories/
│   └── Services/
└── Api/                   # Controllers, minimal APIs
    ├── Controllers/
    ├── Endpoints/
    └── Filters/
```

### 2.2 Feature-Based Structure (Alternative)

```
src/
├── Features/
│   ├── Products/
│   │   ├── Commands/
│   │   ├── Queries/
│   │   ├── Validators/
│   │   └── ProductController.cs
│   └── Orders/
├── Shared/
│   ├── Extensions/
│   └── Middleware/
└── Program.cs
```

### 2.3 Layer References

```csharp
// Api references Application
// Application references Domain
// Infrastructure references Application

// Domain - NO other references
// Application - references Domain
// Infrastructure - references Application + Domain
// Api - references all
```

---

## 3. Minimal APIs

### 3.1 Basic Endpoints

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// GET
app.MapGet("/api/products", async (IProductService service) =>
{
    return await service.GetAllAsync();
});

// GET with route
app.MapGet("/api/products/{id:int}", async (int id, IProductService service) =>
{
    var product = await service.GetByIdAsync(id);
    return product is not null ? Results.Ok(product) : Results.NotFound();
});

// POST
app.MapPost("/api/products", async (CreateProductRequest request, IProductService service) =>
{
    var product = await service.CreateAsync(request);
    return Results.Created($"/api/products/{product.Id}", product);
});

// PUT
app.MapPut("/api/products/{id:int}", async (int id, UpdateProductRequest request, IProductService service) =>
{
    var updated = await service.UpdateAsync(id, request);
    return updated ? Results.NoContent() : Results.NotFound();
});

// DELETE
app.MapDelete("/api/products/{id:int}", async (int id, IProductService service) =>
{
    var deleted = await service.DeleteAsync(id);
    return deleted ? Results.NoContent() : Results.NotFound();
});
```

### 3.2 DTOs

```csharp
public record CreateProductRequest(
    string Name,
    string Description,
    decimal Price,
    int CategoryId
);

public record UpdateProductRequest(
    string? Name = null,
    string? Description = null,
    decimal? Price = null,
    int? CategoryId = null
);

public record ProductResponse(
    int Id,
    string Name,
    string Description,
    decimal Price,
    int CategoryId,
    DateTime CreatedAt
);
```

### 3.3 Grouping & Versioning

```csharp
app.MapGroup("/api/v1/products")
    .WithTags("Products")
    .MapProductEndpoints();  // Extension method

// Versioning
app.MapGroup("/api/v2/products")
    .HasApiVersion(2.0)
    .AddEndpointFilter<VersionHeaderFilter>();
```

### 3.4 Validation

```csharp
app.MapPost("/api/products", async (
    CreateProductRequest request,
    IValidator<CreateProductRequest> validator,
    IProductService service) =>
{
    var validationResult = await validator.ValidateAsync(request);
    if (!validationResult.IsValid)
    {
        return Results.BadRequest(validationResult.Errors.Select(e => e.ErrorMessage));
    }

    var product = await service.CreateAsync(request);
    return Results.Created($"/api/products/{product.Id}", product);
});
```

---

## 4. Entity Framework Core

### 4.1 DbContext

```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    public DbSet<Product> Products => Set<Product>();
    public DbSet<Category> Categories => Set<Category>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Product config
        modelBuilder.Entity<Product>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).IsRequired().HasMaxLength(200);
            entity.Property(e => e.Price).HasPrecision(18, 2);

            entity.HasOne(e => e.Category)
                .WithMany(c => c.Products)
                .HasForeignKey(e => e.CategoryId)
                .OnDelete(DeleteBehavior.Restrict);

            entity.HasIndex(e => e.Name);
        });
    }
}
```

### 4.2 Entities

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string? Description { get; set; }
    public decimal Price { get; set; }
    public int CategoryId { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }

    // Navigation
    public Category Category { get; set; } = null!;
}

public class Category
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;

    // Navigation
    public ICollection<Product> Products { get; set; } = new List<Product>();
}
```

### 4.3 Repository Pattern

```csharp
public interface IProductRepository
{
    Task<Product?> GetByIdAsync(int id);
    Task<IEnumerable<Product>> GetAllAsync();
    Task<PaginatedResult<Product>> GetPaginatedAsync(PaginationRequest request);
    Task<Product> AddAsync(Product product);
    Task UpdateAsync(Product product);
    Task DeleteAsync(Product product);
}

public class ProductRepository : IProductRepository
{
    private readonly ApplicationDbContext _context;

    public ProductRepository(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<Product?> GetByIdAsync(int id)
    {
        return await _context.Products
            .Include(p => p.Category)
            .FirstOrDefaultAsync(p => p.Id == id);
    }

    public async Task<PaginatedResult<Product>> GetPaginatedAsync(PaginationRequest request)
    {
        var query = _context.Products
            .Include(p => p.Category)
            .AsQueryable();

        // Apply filters
        if (!string.IsNullOrEmpty(request.Search))
        {
            query = query.Where(p => p.Name.Contains(request.Search));
        }

        var total = await query.CountAsync();

        var items = await query
            .OrderByDescending(p => p.CreatedAt)
            .Skip((request.Page - 1) * request.Limit)
            .Take(request.Limit)
            .ToListAsync();

        return new PaginatedResult<Product>(items, total, request.Page, request.Limit);
    }

    public async Task<Product> AddAsync(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
        return product;
    }
}
```

### 4.4 Query + Command Split (CQRS-ish)

```csharp
// Queries
public record GetProductsQuery(int Page = 1, int Limit = 10, string? Search = null) :
    IRequest<PaginatedResult<ProductResponse>>;

public class GetProductsHandler : IRequestHandler<GetProductsQuery, PaginatedResult<ProductResponse>>
{
    private readonly IProductRepository _repository;

    public GetProductsHandler(IProductRepository repository)
    {
        _repository = repository;
    }

    public async Task<PaginatedResult<ProductResponse>> Handle(GetProductsQuery request, CancellationToken ct)
    {
        var result = await _repository.GetPaginatedAsync(new PaginationRequest
        {
            Page = request.Page,
            Limit = request.Limit,
            Search = request.Search
        });

        return new PaginatedResult<ProductResponse>(
            result.Items.Select(p => new ProductResponse(p.Id, p.Name, p.Description!, p.Price, p.CategoryId, p.CreatedAt)),
            result.Total,
            result.Page,
            result.Limit
        );
    }
}

// Commands
public record CreateProductCommand(string Name, string Description, decimal Price, int CategoryId) :
    IRequest<ProductResponse>;

public class CreateProductHandler : IRequestHandler<CreateProductCommand, ProductResponse>
{
    private readonly IProductRepository _repository;

    public CreateProductHandler(IProductRepository repository)
    {
        _repository = repository;
    }

    public async Task<ProductResponse> Handle(CreateProductCommand request, CancellationToken ct)
    {
        var product = new Product
        {
            Name = request.Name,
            Description = request.Description,
            Price = request.Price,
            CategoryId = request.CategoryId,
            CreatedAt = DateTime.UtcNow
        };

        var created = await _repository.AddAsync(product);
        return new ProductResponse(created.Id, created.Name, created.Description!, created.Price, created.CategoryId, created.CreatedAt);
    }
}
```

---

## 5. Dependency Injection

### 5.1 Service Registration

```csharp
// Program.cs
builder.Services.AddScoped<IProductRepository, ProductRepository>();
builder.Services.AddScoped<IProductService, ProductService>();

// MediatR
builder.Services.AddMediatR(cfg => cfg.RegisterServicesFromAssembly(typeof(ProductRepository).Assembly));

// FluentValidation
builder.Services.AddValidatorsFromAssemblyContaining<CreateProductRequestValidator>();

// AutoMapper
builder.Services.AddAutoMapper(typeof(MappingProfile));
```

### 5.2 Options Pattern

```csharp
public class JwtSettings
{
    public string Key { get; set; } = string.Empty;
    public string Issuer { get; set; } = string.Empty;
    public int ExpiryMinutes { get; set; }
}

// Program.cs
builder.Services.Configure<JwtSettings>(builder.Configuration.GetSection("Jwt"));

// Usage
public class TokenService
{
    private readonly IOptions<JwtSettings> _settings;

    public TokenService(IOptions<JwtSettings> settings)
    {
        _settings = settings;
    }
}
```

### 5.3 Keyed Services (.NET 8)

```csharp
// Register
builder.Services.AddKeyedSingleton<ICacheService, RedisCacheService>("redis");
builder.Services.AddKeyedSingleton<ICacheService, MemoryCacheService>("memory");

// Use
public class ProductService
{
    public ProductService(
        IKeyedServiceProvider keyedProvider,
        [FromKeyedServices("redis")] ICacheService cache)
    {
        _cache = cache;
    }
}
```

---

## 6. Authentication & Authorization

### 6.1 JWT Bearer

```csharp
// Program.cs
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = jwtSettings.Issuer,
            ValidAudience = jwtSettings.Audience,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtSettings.Key))
        };
    });

builder.Services.AddAuthorization();
```

### 6.2 Policy-Based Authorization

```csharp
// Policies
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));
    options.AddPolicy("CanManageProducts", policy =>
        policy.RequireAssertion(ctx =>
            ctx.User.IsInRole("Admin") ||
            ctx.User.IsInRole("Manager")));
});

// Usage
[Authorize(Policy = "AdminOnly")]
public class AdminController : Controller { }
```

### 6.3 Claims-Based

```csharp
[Authorize]
public class ProductsController : Controller
{
    public IActionResult Index()
    {
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        var role = User.FindFirst(ClaimTypes.Role)?.Value;
        // ...
    }
}
```

### 6.4 Cookie Authentication

```csharp
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/auth/login";
        options.AccessDeniedPath = "/auth/forbidden";
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    });
```

---

## 7. Testing

### 7.1 Unit Tests (xUnit)

```csharp
public class ProductServiceTests
{
    private readonly Mock<IProductRepository> _repositoryMock;
    private readonly ProductService _service;

    public ProductServiceTests()
    {
        _repositoryMock = new Mock<IProductRepository>();
        _service = new ProductService(_repositoryMock.Object);
    }

    [Fact]
    public async Task GetByIdAsync_ExistingId_ReturnsProduct()
    {
        // Arrange
        var product = new Product { Id = 1, Name = "Test" };
        _repositoryMock.Setup(r => r.GetByIdAsync(1)).ReturnsAsync(product);

        // Act
        var result = await _service.GetByIdAsync(1);

        // Assert
        Assert.NotNull(result);
        Assert.Equal("Test", result.Name);
    }

    [Fact]
    public async Task GetByIdAsync_NonExistingId_ReturnsNull()
    {
        // Arrange
        _repositoryMock.Setup(r => r.GetByIdAsync(999)).ReturnsAsync((Product?)null);

        // Act
        var result = await _service.GetByIdAsync(999);

        // Assert
        Assert.Null(result);
    }
}
```

### 7.2 Integration Tests

```csharp
public class ProductsControllerTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;

    public ProductsControllerTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetProducts_ReturnsOk()
    {
        // Arrange & Act
        var response = await _client.GetAsync("/api/products");

        // Assert
        response.EnsureSuccessStatusCode();
    }
}
```

### 7.3 Test Containers

```csharp
public class DatabaseTests : IAsyncLifetime
{
    private readonly MsSqlContainer _msSql;

    public DatabaseTests()
    {
        _msSql = new MsSqlBuilder().Build();
    }

    public async Task InitializeAsync()
    {
        await _msSql.StartAsync();
    }

    public async Task DisposeAsync()
    {
        await _msSql.DisposeAsync();
    }

    [Fact]
    public async Task CanConnect()
    {
        var connectionString = _msSql.GetConnectionString();
        using var connection = new SqlConnection(connectionString);
        await connection.OpenAsync();
        Assert.True(true);
    }
}
```

---

## 8. Performance

### 8.1 Output Caching

```csharp
builder.Services.AddOutputCache();

var app = builder.Build();

app.UseOutputCache();

app.MapGet("/api/products", async (IProductService service) =>
{
    return await service.GetAllAsync();
}).Cache(c => c.SetVaryByHeader("Accept").Expire(TimeSpan.FromMinutes(5)));
```

### 8.2 Response Caching

```csharp
app.UseResponseCaching();

app.MapGet("/api/products", async (IProductService service) =>
{
    return await service.GetAllAsync();
}).CacheOutput();
```

### 8.3 Minimal APIs Optimization

```csharp
// Disable model binding metadata cache for OpenAPI (performance)
builder.Services.AddEndpointsApiExplorer();

// Use ReadAsAsync for large payloads
app.MapPost("/api/upload", async (HttpContext context) =>
{
    var form = await context.Request.ReadFormAsync();
    // Process files
});
```

### 8.4 Indexes & Query Optimization

```csharp
modelBuilder.Entity<Product>()
    .HasIndex(p => p.Name)
    .HasFilter("[Name] IS NOT NULL");

modelBuilder.Entity<Product>()
    .HasIndex(p => new { p.CategoryId, p.Price });
```

---

## 9. Background Services

### 9.1 IHostedService

```csharp
public class EmailSenderService : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;
    private readonly ILogger<EmailSenderService> _logger;

    public EmailSenderService(
        IServiceScopeFactory scopeFactory,
        ILogger<EmailSenderService> logger)
    {
        _scopeFactory = scopeFactory;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            using var scope = _scopeFactory.CreateScope();
            var emailService = scope.ServiceProvider.GetRequiredService<IEmailService>();

            await emailService.SendPendingEmailsAsync(stoppingToken);

            await Task.Delay(TimeSpan.FromMinutes(5), stoppingToken);
        }
    }
}

builder.Services.AddHostedService<EmailSenderService>();
```

### 9.2 Queue-Based Processing

```csharp
public interface IBackgroundTaskQueue
{
    ValueTask QueueTaskAsync(Func<CancellationToken, ValueTask> workItem);
    ValueTask<Func<CancellationToken, ValueTask>> DequeueAsync(CancellationToken cancellationToken);
}

public class BackgroundTaskQueue : IBackgroundTaskQueue
{
    private readonly Channel<Func<CancellationToken, ValueTask>> _queue;

    public BackgroundTaskQueue()
    {
        _queue = Channel.CreateBounded<Func<CancellationToken, ValueTask>>(100);
    }

    public ValueTask QueueTaskAsync(Func<CancellationToken, ValueTask> workItem) =>
        _queue.Writer.WriteAsync(workItem);

    public async ValueTask<Func<CancellationToken, ValueTask>> DequeueAsync(CancellationToken cancellationToken) =>
        await _queue.Reader.ReadAsync(cancellationToken);
}
```

---

## 10. gRPC

### 10.1 Proto File

```protobuf
syntax = "proto3";

option csharp_namespace = "MyApp.Protos";

package product;

service ProductService {
  rpc GetProduct (GetProductRequest) returns (Product);
  rpc GetProducts (GetProductsRequest) returns (stream Product);
  rpc CreateProduct (CreateProductRequest) returns (Product);
}

message GetProductRequest {
  int32 id = 1;
}

message GetProductsRequest {
  int32 page = 1;
  int32 limit = 10;
}

message Product {
  int32 id = 1;
  string name = 2;
  string description = 3;
  double price = 4;
}

message CreateProductRequest {
  string name = 1;
  string description = 2;
  double price = 3;
}
```

### 10.2 gRPC Server

```csharp
// Program.cs
builder.Services.AddGrpc();

app.MapGrpcService<ProductGrpcService>();
app.MapGet("/", () => "gRPC Server");

// Service implementation
public class ProductGrpcService : ProductService.ProductServiceBase
{
    private readonly IProductRepository _repository;

    public ProductGrpcService(IProductRepository repository)
    {
        _repository = repository;
    }

    public override async Task<Product> GetProduct(GetProductRequest request, ServerCallContext context)
    {
        var product = await _repository.GetByIdAsync(request.Id);
        return new Product
        {
            Id = product.Id,
            Name = product.Name,
            Description = product.Description ?? "",
            Price = (double)product.Price
        };
    }
}
```

---

## 11. Deployment

### 11.1 Docker

```dockerfile
# Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["MyApp.csproj", "./"]
RUN dotnet restore MyApp.csproj
COPY . .
RUN dotnet build MyApp.csproj -c Release -o /app/build

FROM build AS publish
RUN dotnet publish MyApp.csproj -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

### 11.2 docker-compose.yml

```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8080:80"
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ConnectionStrings__DefaultConnection=Server=db;Database=app;User=sa;Password=Secret123!
    depends_on:
      - db

  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=Secret123!
      - MSSQL_PID=Developer
    ports:
      - "1433:1433"
```

### 11.3 Azure App Service

```yaml
# azure.yaml
resources:
  - name: myapp
    type: Microsoft.Web/sites
    properties:
      location: westus2
      serverFarmId: /subscriptions/sub-id/resourceGroups/rg-name/providers/Microsoft.Web/serverPlans/plan-name
      siteConfig:
        appSettings:
          - name: ConnectionStrings__DefaultConnection
            value: ${azuresql-connection-string}
```

---

## Checklist

### Project Setup
- [ ] Nullable reference types enabled
- [ ] Implicit usings configured
- [ ] Treat warnings as errors

### API
- [ ] Minimal APIs or Controllers
- [ ] DTOs for requests/responses
- [ ] Validation (FluentValidation)
- [ ] Swagger/OpenAPI

### Data
- [ ] EF Core with proper configuration
- [ ] Repository pattern or CQRS
- [ ] Async operations
- [ ] Pagination

### Security
- [ ] JWT authentication
- [ ] Authorization policies
- [ ] CORS configured

### Testing
- [ ] Unit tests
- [ ] Integration tests
- [ ] Test coverage

### Performance
- [ ] Output caching
- [ ] Database indexes
- [ ] Async/await

---

*.NET 8/9 Modern C# Skill*
*Versión: 1.0*