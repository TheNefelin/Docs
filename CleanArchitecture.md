# Clean Architecture en C#

## Estructura Básica

```
└── Proyecto/
    ├── Core/             (Domain + Application)
    ├── Infrastructure/
    └── Web/              (Presentation)
```

## Ejemplo Domain Layer

```csharp
// Core/Entities/Product.cs
public class Product {
    public int Id { get; set; }
    public string Name { get; set; }
}

// Core/Interfaces/IProductRepository.cs
public interface IProductRepository {
    Product GetById(int id);
}
```

## Ejemplo Application Layer

```csharp
// Core/Services/ProductService.cs
public class ProductService {
    private readonly IProductRepository _repo;
    
    public ProductService(IProductRepository repo) {
        _repo = repo;
    }
    
    public Product GetProduct(int id) {
        return _repo.GetById(id);
    }
}
```

## Ejemplo Infrastructure

```csharp
// Infrastructure/Repositories/ProductRepository.cs
public class ProductRepository : IProductRepository {
    public Product GetById(int id) {
        // Lógica de acceso a datos
    }
}
```

## Ejemplo Presentation (Web API)

```csharp
// Web/Controllers/ProductsController.cs
[ApiController]
public class ProductsController : ControllerBase {
    private readonly ProductService _service;
    
    public ProductsController(ProductService service) {
        _service = service;
    }
    
    [HttpGet("{id}")]
    public IActionResult Get(int id) {
        return Ok(_service.GetProduct(id));
    }
}
```

## Reglas Clave

- 1. Dependencias: Solo hacia adentro (Web → Infrastructure → Application → Domain)
- 2. Testing: Domain y Application son testables sin infraestructura
- 3. Frameworks: Solo en capas externas (Web/Infrastructure)

---

# Estructura Completa de Clean Architecture

```
src/
├── Core/
│   ├── Domain/             # Capa de dominio
│   │   ├── Entities/       # Entidades de negocio
│   │   ├── ValueObjects/   # Objetos de valor
│   │   ├── Enums/          # Enumeraciones
│   │   ├── Exceptions/     # Excepciones personalizadas
│   │   └── Services/       # Servicios de dominio
│   │
│   └── Application/        # Capa de aplicación
│       ├── DTOs/           # Objetos de transferencia
│       ├── Interfaces/     # Interfaces para servicios externos
│       ├── Mappings/       # AutoMapper profiles
│       ├── Features/       # Organización por features (opcional)
│       ├── Behaviors/      # MediatR behaviors
│       └── Services/       # Servicios de aplicación
│
├── Infrastructure/
│   ├── Persistence/        # Acceso a datos
│   │   ├── Repositories/   # Implementaciones de repositorios
│   │   ├── Migrations/     # Migraciones de EF Core
│   │   └── Context/        # DbContext
│   │
│   ├── Shared/             # Servicios compartidos
│   │   ├── FileStorage/    # Almacenamiento de archivos
│   │   ├── Email/          # Servicio de email
│   │   └── Caching/        # Caché
│   │
│   └── External/           # Integraciones externas
│       ├── APIs/           # Clients para APIs externas
│       └── Services/       # Servicios de terceros
│
└── Web/                    # Capa de presentación
    ├── Controllers/        # Controladores API/MVC
    ├── Middlewares/        # Middlewares personalizados
    ├── Filters/            # Filtros
    ├── Views/              # Vistas (si es MVC)
    └── wwwroot/            # Archivos estáticos
```

## Ejemplo Completo por Capas

### 1. Domain Layer (Core/Domain)

```csharp
// Entidad de negocio
public class Product : EntityBase
{
    public string Name { get; private set; }
    public decimal Price { get; private set; }
    public int Stock { get; private set; }

    public void UpdateStock(int quantity)
    {
        if (quantity < 0) throw new DomainException("Stock no puede ser negativo");
        Stock = quantity;
    }
}

// Objeto de valor
public class Address : ValueObject
{
    public string Street { get; }
    public string City { get; }
    public string ZipCode { get; }

    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return Street;
        yield return City;
        yield return ZipCode;
    }
}
```

### 2. Application Layer (Core/Application)

```csharp
// DTO
public record ProductDto(int Id, string Name, decimal Price);

// Interfaz de repositorio
public interface IProductRepository
{
    Task<Product> GetByIdAsync(int id);
    Task AddAsync(Product product);
}

// Caso de uso con MediatR
public class GetProductByIdQuery : IRequest<ProductDto>
{
    public int Id { get; set; }
}

public class GetProductByIdHandler : IRequestHandler<GetProductByIdQuery, ProductDto>
{
    private readonly IProductRepository _repository;
    private readonly IMapper _mapper;

    public async Task<ProductDto> Handle(GetProductByIdQuery request, CancellationToken cancellationToken)
    {
        var product = await _repository.GetByIdAsync(request.Id);
        return _mapper.Map<ProductDto>(product);
    }
}
```

### 3. Infrastructure Layer

```csharp
// Implementación de repositorio
public class ProductRepository : IProductRepository
{
    private readonly ApplicationDbContext _context;

    public async Task<Product> GetByIdAsync(int id)
    {
        return await _context.Products.FindAsync(id);
    }
}

// DbContext
public class ApplicationDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }
}

// Servicio externo (ejemplo Email)
public class EmailService : IEmailService
{
    public async Task SendAsync(string to, string subject, string body)
    {
        // Implementación con SendGrid/MailKit/etc
    }
}
```

### 4. Presentation Layer (Web)

```csharp
// Controlador API
[ApiController]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    private readonly IMediator _mediator;

    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDto>> Get(int id)
    {
        var product = await _mediator.Send(new GetProductByIdQuery { Id = id });
        return Ok(product);
    }
}

// Startup/Program.cs (Configuración)
builder.Services
    .AddApplication()    // Capa Application
    .AddInfrastructure() // Capa Infrastructure
    .AddWeb();           // Capa Web
```

---