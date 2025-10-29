### Componentes
```sh
# Componente básico
ng generate component nombre-componente
ng g c nombre-componente

# Con opciones adicionales
ng g c nombre-componente --skip-tests           # Sin archivo de testing
ng g c nombre-componente --inline-template      # Template inline
ng g c nombre-componente --inline-style         # Estilos inline
ng g c nombre-componente --flat                 # Sin carpeta propia
ng g c nombre-componente --prefix mi-prefijo    # Con prefijo personalizado
ng g c nombre-componente --view-encapsulation None  # Sin encapsulación
```

### Servicios
```sh
# Servicio básico
ng generate service nombre-servicio
ng g s nombre-servicio

# Con opciones
ng g s nombre-servicio --flat                   # Sin carpeta
ng g s servicios/nombre-servicio               # En carpeta específica
```

### Módulos
```sh
# Módulo básico
ng generate module nombre-modulo
ng g m nombre-modulo

# Módulo con routing
ng g m nombre-modulo --routing
ng g m nombre-modulo --routing --route ruta-principal
```

### Directivas
```sh
ng generate directive nombre-directiva
ng g d nombre-directiva
```

### Pipes
```sh
ng generate pipe nombre-pipe
ng g p nombre-pipe
```

### Guards
```sh
# Diferentes tipos de guards
ng generate guard nombre-guard
ng g g nombre-guard

# Al ejecutar te preguntará qué tipo de guard:
# - CanActivate
# - CanActivateChild
# - CanDeactivate
# - CanLoad
```

### Interceptores
```sh
ng generate interceptor nombre-interceptor
ng g interceptor nombre-interceptor
```

### Interfaces
```sh
ng generate interface nombre-interface
ng g i nombre-interface
```

### Enums
```sh
ng generate enum nombre-enum
ng g e nombre-enum
```

### Classes
```sh
ng generate class nombre-clase
ng g cl nombre-clase
```
