# Principios SOLID con Ejemplos en C#

## 1. Principio de Responsabilidad Única (SRP)

**Definición**: Una clase debe tener una sola razón para cambiar.

```csharp
// Mal
public class Usuario {
    public void Guardar() { /*...*/ }
    public void EnviarEmail() { /*...*/ }
}

// Bien
public class Usuario { /*...*/ }
public class UsuarioRepository { public void Guardar() { /*...*/ } }
public class EmailService { public void Enviar() { /*...*/ } }
```

## 2. Principio Abierto/Cerrado (OCP)
**Definición**: Abierto para extensión, cerrado para modificación.

```csharp
// Mal
public class AreaCalculator {
    public double Calcular(object forma) {
        if (forma is Rectangulo) { /*...*/ }
        else if (forma is Circulo) { /*...*/ }
    }
}

// Bien
public abstract class Forma {
    public abstract double Area();
}
public class Rectangulo : Forma { /*...*/ }
public class Circulo : Forma { /*...*/ }
```

## 3. Principio de Sustitución de Liskov (LSP)
**Definición**: Los subtipos deben ser sustituibles por sus tipos base.

```csharp
// Mal
public class Pajaro {
    public virtual void Volar() { /*...*/ }
}
public class Pinguino : Pajaro {
    public override void Volar() => throw new Exception();
}

// Bien
public interface IVolador { void Volar(); }
public class Pinguino { /* Sin Volar */ }
```

## 4. Principio de Segregación de Interfaces (ISP)
**Definición**: Interfaces específicas > interfaces genéricas.

```csharp
// Mal
public interface IDispositivo {
    void Imprimir();
    void Escanear();
}

// Bien
public interface IImpresora { void Imprimir(); }
public interface IEscanner { void Escanear(); }
```

## 5. Principio de Inversión de Dependencias (DIP)
**Definición**: Depender de abstracciones, no de implementaciones.

```csharp
// Mal
public class Servicio {
    private MySQLDatabase _db = new MySQLDatabase();
}

// Bien
public class Servicio {
    private IDatabase _db;
    public Servicio(IDatabase db) { _db = db; }
}
```