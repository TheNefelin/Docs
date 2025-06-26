# 📘 Guía de Estudio para Entrevista Técnica - .NET C\#

---

## 🨀 Fundamentos Generales

### 🔹 Programación Orientada a Objetos (POO)

La POO es un paradigma que organiza el código en clases y objetos. Esencial para escribir código mantenible y escalable. (Maintainable and scalable)

**Conceptos clave:**

- **Encapsulamiento:** esconder detalles internos. Ejemplo:
  ```csharp
  public class Persona {
      private int edad;
      public int Edad {
          get => edad;
          set => edad = value > 0 ? value : 0;
      }
  }
  ```
- **Herencia:** compartir comportamiento entre clases. Ejemplo:
  ```csharp
  public class Animal { public void Comer() => Console.WriteLine("Comiendo"); }
  public class Perro : Animal {}
  ```
- **Polimorfismo:** capacidad de sobrescribir o implementar múltiples formas. Ejemplo:
  ```csharp
  public virtual void HacerSonido() => Console.WriteLine("Animal");
  public override void HacerSonido() => Console.WriteLine("Perro");
  ```
- **Abstracción:** ocultar la complejidad usando interfaces o clases abstractas.

### 🔹 Buenas Prácticas

- **SOLID:** principios para diseñar software limpio y mantenible.
- **DRY:** Don't Repeat Yourself
- **KISS:** Keep It Simple, Stupid
- **YAGNI:** You Aren't Gonna Need It
- **Inyección de dependencias:** evitar acoplamiento fuerte.

```csharp
public interface IEmailService {
    void Enviar(string mensaje);
}
public class EmailService : IEmailService {
    public void Enviar(string mensaje) => Console.WriteLine(mensaje);
}
public class Notificador {
    private readonly IEmailService _email;
    public Notificador(IEmailService email) { _email = email; }
}
```

### 📆 Clean Code (Código Limpio)

Clean Code es una filosofía de desarrollo de software basada en la legibilidad, mantenibilidad y simplicidad del código.

**Principios:**

- Usa **nombres descriptivos** para funciones, variables y clases.
- Divide tu código en **funciones pequeñas** con una sola responsabilidad.
- Evita **duplicación de lógica**.
- Escribe código **autodocumentado**, donde el nombre ya explica lo que hace.

#### ❌ Ejemplo de código NO limpio

```csharp
public void P1() {
    var x = new List<string>();
    x.Add("Juan");
    x.Add("Pedro");
    x.Add("Maria");

    foreach (var a in x) {
        if (a.StartsWith("J"))
            Console.WriteLine(a);
    }
}
```

#### ✅ Ejemplo de código limpio (Clean Code)

```csharp
public void ImprimirNombresQueEmpiezanConJ() {
    var nombres = ObtenerNombres();
    foreach (var nombre in nombres) {
        if (EmpiezaConJ(nombre))
            Console.WriteLine(nombre);
    }
}

private List<string> ObtenerNombres() => new() { "Juan", "Pedro", "Maria" };

private bool EmpiezaConJ(string nombre) => nombre.StartsWith("J");
```

---

## 💻 Lenguaje: C\#

### 🔹 Características del Lenguaje

- `var`: tipado implícito
- `async/await`: programación asincrónica
- `record`: tipos inmutables

```csharp
public record Persona(string Nombre, int Edad);
```

- Expresiones lambda:

```csharp
var lista = new List<int> { 1, 2, 3 };
var pares = lista.Where(x => x % 2 == 0);
```

- LINQ:

```csharp
var nombres = personas.Where(p => p.Edad > 18).Select(p => p.Nombre);
```

- `ref`, `out`, `in`:

```csharp
void Incrementar(ref int x) { x++; }
```

---

## ⚙️ .NET Core / ASP.NET Core

### 🔹 ASP.NET Core

- `Program.cs` y `Startup.cs` definen la configuración de la app.
- Middleware:

```csharp
app.Use(async (context, next) => {
    Console.WriteLine("Request entrante");
    await next();
});
```

- Inyección de dependencias:

```csharp
services.AddScoped<IMiServicio, MiServicio>();
```

### 🔹 API REST

- Crear controlador REST:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductosController : ControllerBase {
    [HttpGet]
    public IActionResult Get() => Ok("Lista de productos");
}
```

- Validación:

```csharp
public class Producto {
    [Required]
    public string Nombre { get; set; }
}
```

---

## 📄 Base de Datos: Entity Framework Core

- `DbContext`, `DbSet<T>`:

```csharp
public class AppDbContext : DbContext {
    public DbSet<Producto> Productos { get; set; }
}
```

- Relaciones:

```csharp
public class Pedido {
    public int Id { get; set; }
    public List<Detalle> Detalles { get; set; }
}
```

- Consultas LINQ:

```csharp
var productos = _context.Productos.Where(p => p.Precio > 100).ToList();
```

- `Include`:

```csharp
var pedido = _context.Pedidos.Include(p => p.Detalles).First();
```

---

## 🔐 Seguridad

- Autenticación con JWT

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(...);
```

- Autorizar acceso:

```csharp
[Authorize(Roles = "Admin")]
```

---

## 📊 Testing

- Unit testing con xUnit:

```csharp
public class CalculadoraTests {
    [Fact]
    public void Sumar_DeberiaRetornarCorrecto() {
        var resultado = new Calculadora().Sumar(2, 3);
        Assert.Equal(5, resultado);
    }
}
```

- Mocking con Moq:

```csharp
var mock = new Mock<IMiServicio>();
mock.Setup(x => x.Obtener()).Returns("Hola");
```

---

## 🛠️ Herramientas y DevOps

- Git: `git clone`, `git branch`, `git rebase`
- Docker:

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:6.0
COPY . /app
WORKDIR /app
ENTRYPOINT ["dotnet", "MiApp.dll"]
```

- Swagger:

```csharp
services.AddSwaggerGen();
app.UseSwagger();
app.UseSwaggerUI();
```

---

## 💬 Preguntas Típicas en Entrevistas

### Técnicas

- ¿Cuál es la diferencia entre `IEnumerable` y `IQueryable`?
- ¿Cómo funciona el `async/await` internamente?
- ¿Qué es un `Middleware` y para qué sirve?
- ¿Cómo se configura la inyección de dependencias en .NET?

### Experiencia

- Cuéntame un bug crítico que solucionaste.
- ¿Has trabajado con despliegues en Docker o Azure?
- ¿Cómo estructuras tus proyectos para que sean escalables?

---

✅ **Consejo Final:** Practica con proyectos reales, usa GitHub para mostrar tu trabajo, y prepara respuestas técnicas con ejemplos concretos. ¡Mucho éxito!

