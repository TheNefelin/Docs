
# 🌐 Guía Básica de Arquitectura Cloud y Distribución de Software

## 📘 1. ¿Qué es Cloud Computing?
Cloud Computing es un modelo que permite el acceso bajo demanda a recursos informáticos (servidores, almacenamiento, redes, software) a través de Internet, sin necesidad de gestionarlos físicamente.

---

## ☁️ 2. Modelos de Servicio Cloud

| Modelo  | Nombre completo               | ¿Qué proporciona?                                 | Ejemplos                      |
|---------|-------------------------------|---------------------------------------------------|-------------------------------|
| IaaS    | Infrastructure as a Service   | Infraestructura virtual                           | AWS EC2, Azure VM             |
| PaaS    | Platform as a Service         | Plataforma para desarrollar y desplegar apps      | Heroku, Google App Engine     |
| SaaS    | Software as a Service         | Software listo para usar desde el navegador       | Gmail, Salesforce, Dropbox    |

---

## 🏢 3. Diferencias entre On-Premise y Cloud

| Característica           | On-Premise                  | Cloud                          |
|--------------------------|-----------------------------|--------------------------------|
| Instalación              | Local, en tus servidores    | En la nube                     |
| Gestión de infraestructura | Por el cliente             | Por el proveedor               |
| Escalabilidad            | Limitada                    | Alta y flexible                |
| Costo inicial            | Alto                        | Bajo (pago por uso)            |

---

## 🧠 4. Arquitectura del Software

### 📦 Estática
Estructura fija del sistema (clases, capas, módulos).

```csharp
// Ejemplo de clase simple en C#
public class Producto {
    public int Id { get; set; }
    public string Nombre { get; set; }
}
```

### 🔄 Dinámica
Comportamiento en tiempo de ejecución (interacciones entre objetos).

```csharp
// Ejemplo de interacción en tiempo de ejecución
var producto = new Producto();
producto.Nombre = "Teclado";
Console.WriteLine(producto.Nombre);
```

### ⚙️ Funcional
Qué funciones realiza el sistema (casos de uso, funcionalidades del negocio):

- Registrar usuario
- Realizar pedido
- Enviar notificación

---

## 🧩 5. Patrones comunes en Cloud

- **MVC**: Separación entre Modelo (datos), Vista (UI) y Controlador (lógica).
- **Monolítica**: Toda la lógica en un solo proyecto/aplicación.
- **Microservicios**: Servicios independientes que colaboran.
- **Serverless**: Ejecutas funciones sin preocuparte del servidor.

```csharp
// Ejemplo simple de controlador en MVC (ASP.NET)
public class ProductosController : Controller {
    public IActionResult Index() {
        var productos = repositorio.ObtenerTodos();
        return View(productos);
    }
}
```

---

## ✅ 6. Consejos para Aprender

- Aprende por capas: empieza por SaaS → PaaS → IaaS.
- Relaciona arquitectura con patrones (como MVC).
- Practica con ejemplos simples (como clases en C#).
- Usa diagramas para visualizar (UML, flujo de datos).

---

## 📚 Recursos adicionales

- Microsoft Learn
- AWS Cloud Practitioner Essentials
- Google Cloud Training
