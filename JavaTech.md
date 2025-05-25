# Java Tech

## Basic Concepts
* <strong>Spring:</strong> Framework para desarrollo de apps en Java

* <strong>SpringBoot:</strong> Extencion de Spring Simplifica codigficar y trae configuraciones predeterminadas

* <strong>Herramientas para programar en Java:</strong>
  * JDK: Java Development Kit
    * Compilador (javac): convierte .class a bytcode
    * Java Runtime Environment (JRE): proporciona máquina virtual (JVM)
  * IDE
    * InteliJ IDEA
    * Eclipse
    * NetBEans

* <strong>Tecnologias:</strong>
  * JPA: (Java Persistence API) Es una especificación, una API para ORM
  * Hibernate: Es una implementación JPA, es un proveedor de JPA, Es el Framework ORM 

* <strong>POO:</strong>
  * Encapsulamiento,
  * Herencia
  * Polimorfismo @Override
  * Abstraccion

* <strong>Principios SOLID:</strong>
  * Single Responsibility: clase una única responsabilidad
  * Open/Closed: clases para extensión, cerradas para modificación.
  * Liskov Substitution: subclase pueda reemplazar a su superclase 
  * Interface Segregation: evitar la creación de interfaces "gordas"
  * Dependency Inversion: Reducir el acoplamiento entre componentes

* <strong>Anotaciones:</strong> son una forma de añadir metadatos a tu código
  * Sirven Para
    * injeccion de dependencias (@Autowired)
    * configuracion de aplicaciones y beans
    * pruebas unitarias
    * persistencia de datos
    * Servicios y Rest
    * Transacciones (@Transactional)
    * etc..

* <strong>Interfaz:</strong> Contrato que las clases deben segir e implementar sus metodos abstractos.
  * Injeccion de dependecia
    * por Constructor
    * por anotacion @Autowired
    * por serter
  * Implementacion
    * si son mas de una utilizar anotacion @Qualifier("nomServicio")

* <strong>Abstract:</strong> Contiene atributos y metodos con o sin implementacion, la clase que hereda debe implementar los metodos sin implementacion
  * 

* <strong>Patrones de Diseño:</strong>
  * Builder: patron creacional
  * Singleton: Instancia unica para mejor desempeño, acceso global
    * constructor privado
    * metodo estatico
    * declaracion estatica de si mismo

* <strong>Programacion Funcional:</strong> Son como funciones anonimas
  * Funciones como Objetos
  ```java
  public class FunctionalExample {
    public static void main(String[] args) {
        Function<Integer, String> intToString = (i) -> "Number: " + i;
        System.out.println(intToString.apply(5)); // Output: Number: 5
    }
  }  
  ```
    * Function<T, R> argumento T revuelve R
    * Consumer< T > argumento T revuelve nada
    * Supplier< T > sin argumento revuelve T
    * Predicate< T > argumento T revuelve bool
  * Expresiones lambda: .forEach(a -> b)
  * Streams: .stream().filter(lambda...
  * Inmutabilidad: clase con atributo final, se instancia en el constructor y tiene geter
  * Optional

* <strong>Arquitectura Monolito y Micro Servicio:</strong>
  * Monolito: aplicacion de estructura unificada un solo despliegue y aplicaiones pequeñas (poca cocurrencia de datos)
  * Micro Servicio: Aaplicacion separada en servicios pequeños con despliegue independeite y se escala por separado (concurrencia independecite para cada servicio)

* <strong>Caso1: Tareas que no manejo</strong>
  * Googlear
  * Compañeros
  * Tech Lead
  * Product Owner
  * Lo antes posible avisa y no esperar la DeadLine

* <strong>Caso2: Se cae el Deploy</strong>
  * Importancia del cliente
  * Gravedad del error
  * Importancia del requerimiento
  * Tiempo

* <strong>Otros:</strong>

## Patrones de Diseño

### Singleton
```java
public class Singleton {
    private static Singleton instancia;

    private Singleton() {}

    public static synchronized Singleton getInstancia() {
        if (instancia == null) {
            instancia = new Singleton();
        }
        return instancia;
    }
}

```

### Factory: crear objetos sin especificar la clase exacta del objeto
```java
public class AnimalFactory {
    public static Animal crearAnimal(String tipo) {
        switch (tipo) {
            case "Perro": return new Perro();
            case "Gato": return new Gato();
            default: throw new IllegalArgumentException("Tipo desconocido");
        }
    }
}
```

## Manejo de Colecciones
```java
public class UniqueList {
    public static List<Integer> obtenerUnicos(List<Integer> lista) {
        Set<Integer> set = new HashSet<>(lista);
        return new ArrayList<>(set);
    }
}
```

```java
public class UniqueList {
    public static List<Integer> obtenerUnicos(List<Integer> lista) {
        return lista.stream().distinct().toList();
    }
}
```

## Excepciones y Manejo de Errores
```java
public class FileReaderExample {
    public String leerArchivo(String ruta) {
        StringBuilder contenido = new StringBuilder();
        try (BufferedReader br = new BufferedReader(new FileReader(ruta))) {
            String linea;
            while ((linea = br.readLine()) != null) {
                contenido.append(linea).append("\n");
            }
        } catch (FileNotFoundException e) {
            System.out.println("Archivo no encontrado: " + e.getMessage());
        } catch (IOException e) {
            System.out.println("Error de lectura: " + e.getMessage());
        }
        return contenido.toString();
    }
}
```

## Concurrencia
```java
public class Contador extends Thread {
    @Override
    public void run() {
        for (int i = 1; i <= 10; i++) {
            System.out.println(i);
            try {
                Thread.sleep(1000); // Espera de 1 segundo
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        Contador hilo1 = new Contador();
        Contador hilo2 = new Contador();
        hilo1.start();
        hilo2.start();
    }
}
```
