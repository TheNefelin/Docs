# Angular 21

## Fetch
- ng g interface core/models/api-response-model
```ts
export interface ApiResponseModel<T> {
  isSuccess: boolean
  statusCode: number
  message: string
  data: T
}
```
- ng g s core/helpers/api-response-service --skip-tests
```ts
@Injectable({
  providedIn: 'root',
})
export class ApiResponseService<R,T> {
  private http = inject(HttpClient);
  private apiUrl = environment.apiUrl;
  private apiKey = environment.apiKey

  private getHeaders(): HttpHeaders {
    return new HttpHeaders({
      'ApiKey': this.apiKey,
      'Content-Type': 'application/json'
    });
  }

  getAll(endpoint: string): Observable<R> {
    return this.http.get<R>(
      `${this.apiUrl}/${endpoint}`,
      { headers: this.getHeaders() }
    );
  }

  getById(endpoint: string, id: string | number): Observable<R> {
    return this.http.get<R>(
      `${this.apiUrl}/${endpoint}/${id}`,
      { headers: this.getHeaders() }
    );
  }

  create(endpoint: string, data: T): Observable<R> {
    return this.http.post<R>(
      `${this.apiUrl}/${endpoint}`,
      data,
      { headers: this.getHeaders() }
    );
  }

  update(endpoint: string, data: T): Observable<R> {
    return this.http.put<R>(
      `${this.apiUrl}/${endpoint}`,
      data,
      { headers: this.getHeaders() }
    );
  }

  delete(endpoint: string, id: string | number): Observable<R> {
    return this.http.delete<R>(
      `${this.apiUrl}/${endpoint}/${id}`,
      { headers: this.getHeaders() }
    );
  }
}
```

# Angular 21 — Prácticas Modernas

Guía práctica de las características modernas de Angular que están siendo utilizadas actualmente. Cada sección explica **qué es**, **por qué existe**, y **cómo implementarlo correctamente**.

---

## Índice

1. [Componentes Standalone](#1-componentes-standalone)
2. [Señales (Signals)](#2-señales-signals)
3. [Inyección con `inject()`](#3-inyección-con-inject)
4. [`toSignal` — Convertir Observables a Señales](#4-tosignal--convertir-observables-a-señales)
5. [`computed` — Señales Derivadas](#5-computed--señales-derivadas)
6. [Control de Flujo en Template (`@if`, `@for`)](#6-control-de-flujo-en-template-if-for)
7. [Eventos Nativos vs `ngSubmit`](#7-eventos-nativos-vs-ngsubmit)
8. [Manejo de Estado sin Formularios de Angular](#8-manejo-de-estado-sin-formularios-de-angular)
9. [Patrones de Comunicación con el Backend](#9-patrones-de-comunicación-con-el-backend)
10. [Estructura Recomendada de un Componente Moderno](#10-estructura-recomendada-de-un-componente-moderno)

---

## 1. Componentes Standalone

### ¿Qué es?

Los componentes standalone eliminan la necesidad de un `NgModule` para declarar y organizar componentes. Cada componente es autosuficiente: declara sus propias dependencias directamente.

### ¿Por qué existe?

Antes de esta característica, cada componente debía ser declarado en un `@NgModule`, lo cual generaba acoplamiento innecesario y archivos de módulo enormes. Los standalone simplifican la arquitectura radicalmente.

### Implementación correcta

```typescript
@Component({
  selector: 'app-url-form-page',
  imports: [          // ← Solo importas lo que este componente necesita
    RouterLink,
    LoadingComponent,
    MessageErrorComponent,
    MessageSuccessComponent,
  ],
  templateUrl: './url-form-page.html',
})
export class UrlFormPage {
  // ...
}
```

### Reglas clave

- Si usas una directiva o componente en el template, **debe estar en `imports`**.
- Si usas `(ngSubmit)` o `[(ngModel)]`, **debe importar `FormsModule`**.
- Si usas `FormBuilder` o `FormControl`, **debe importar `ReactiveFormsModule`**.
- Si no necesitas ninguna de estas directivas, **no las importes**. Menos es más.

### Ejemplo: qué importar según lo que uses en el template

```typescript
// Si usas routerLink en el template
imports: [RouterLink]

// Si usas (ngSubmit) o [(ngModel)]
imports: [FormsModule]

// Si usas formGroup, formControlName
imports: [ReactiveFormsModule]

// Si no usas directivas de Angular en el template, imports puede estar vacío
imports: []
```

---

## 2. Señales (Signals)

### ¿Qué es?

Una señal es una **celda reactiva** que contiene un valor. Cuando ese valor cambia, Angular sabe automáticamente qué partes del template necesitan actualizarse, sin necesidad de Zone.js ni detección de cambios manual.

### ¿Por qué existe?

Antes, Angular usaba Zone.js para interceptar eventos y disparar detección de cambios en todo el árbol de componentes, lo cual era costoso en rendimiento. Las señales permiten una detección de cambios **granular y explícita**.

### Tipos de señales

| Tipo | Función | Mutable | Uso |
|---|---|---|---|
| `signal()` | Crea una señal escribible | ✅ Sí | Estado local del componente |
| `computed()` | Crea una señal derivada | ❌ No | Valores que dependen de otras señales |
| `toSignal()` | Convierte un Observable en señal | ❌ No | Datos que vienen del backend |

### Implementación correcta

```typescript
// Señal escribible: estado que el componente modifica directamente
readonly isLoading = signal(false);
readonly errorMessage = signal<string | null>(null);

// Señal con objeto complejo
readonly formData = signal<Partial<UrlModel>>({
  name: '',
  link: '',
  isEnable: true,
  id_UrlGrp: 0,
});
```

### Cómo leer y escribir

```typescript
// Leer el valor actual (en TypeScript)
const valor = this.isLoading();        // valor: false

// Escribir un nuevo valor
this.isLoading.set(true);

// Actualizar basándose en el valor actual
this.formData.update((data) => ({ ...data, name: 'nuevo nombre' }));
```

### Cómo usar en el template

```html
<!-- Leer una señal en el template: simplemente llamar como función -->
@if (isLoading()) {
  <app-loading-component />
}

<!-- Pasar el valor de una señal a un input -->
[value]="formData().name"

<!-- Usar una señal derivada -->
{{ isEditMode() ? 'Modificar' : 'Crear' }}
```

### Reglas clave

- Siempre declara señales con `readonly` para que solo el componente dueño las escriba.
- En el template, **siempre con paréntesis**: `isLoading()`, no `isLoading`.
- Usa `.set()` cuando reemplazas el valor completo.
- Usa `.update()` cuando el nuevo valor depende del anterior.

---

## 3. Inyección con `inject()`

### ¿Qué es?

`inject()` es una función que obtiene una instancia de un servicio dentro del contexto de inyección de un componente, sin necesidad de usar el constructor.

### ¿Por qué existe?

El constructor tradicional para inyección de dependencias sigue siendo válido, pero `inject()` permite un código más limpio cuando se combina con señales y composición funcional. Elimina la necesidad de constructores largos.

### Implementación correcta

```typescript
export class UrlFormPage {
  // ✅ Correcto: inject() en la zona de inicialización de propiedades
  private readonly urlService = inject(UrlService);
  private readonly router = inject(Router);

  // ❌ Incorrecto: inject() dentro de un método (fuera del contexto de inyección)
  protected onSubmit(): void {
    const service = inject(UrlService); // ← ERROR en runtime
  }
}
```

### Dónde es válido usar `inject()`

`inject()` solo funciona dentro del **contexto de inyección**, que incluye:

- Zona de inicialización de propiedades de la clase.
- El constructor del componente.
- Funciones llamadas **sincrónicamente** desde alguno de los dos lugares anteriores.

```typescript
export class MiComponente {
  // ✅ Válido
  private readonly svc = inject(MiServicio);

  // ✅ Válido (constructor)
  constructor() {
    const otra = inject(OtroServicio);
  }

  // ❌ Inválido (método async, fuera del contexto)
  async ngOnInit() {
    const svc = inject(MiServicio); // Error
  }
}
```

---

## 4. `toSignal` — Convertir Observables a Señales

### ¿Qué es?

`toSignal()` toma un Observable y lo convierte en una señal que Angular puede rastrear reactivamente en el template. Esto elimina la necesidad de usar `async pipe` o suscribirse manualmente.

### ¿Por qué existe?

Los servicios HTTP devuelven Observables. Las señales son el sistema reactivo nativo de Angular. `toSignal()` es el puente entre ambos mundos.

### Implementación correcta

```typescript
import { toSignal } from '@angular/core';

// Básico: el valor inicial es undefined hasta que el Observable emite
private readonly datos = toSignal(this.miServicio.getAll());

// Con valor inicial explícito
private readonly datos = toSignal(
  this.miServicio.getAll(),
  { initialValue: [] }
);

// Con manejo de errores (patrón recomendado)
private readonly urlgrpSignal = toSignal(
  this.urlgrpService.getAll().pipe(
    catchError((err) => {
      console.error('Error:', err);
      return of({
        isSuccess: false,
        statusCode: 500,
        message: err?.message ?? 'Error 500',
        data: []
      } as ApiResponseModel<UrlGrpModel[]>);
    })
  ),
  { initialValue: undefined }
);
```

### Reglas clave

- Si no provienes `initialValue`, el tipo de la señal incluye `undefined` automáticamente.
- El Observable **se suscribe automáticamente** cuando el componente se crea y **se desustacribe automáticamente** cuando se destruye.
- No es adecuado para Observables que necesitas disparar manualmente (como un POST). Para esos casos, usa `.subscribe()` directamente.

---

## 5. `computed` — Señales Derivadas

### ¿Qué es?

`computed()` crea una señal que se recalcula automáticamente cuando **cualquiera de las señales que lee** cambia. Es la forma de crear lógica derivada sin lógica duplicada.

### ¿Por qué existe?

Si tienes un valor que depende de otros valores reactivos, `computed()` lo mantiene sincronizado automáticamente. Es equivalente a una fórmula en una hoja de cálculo.

### Implementación correcta

```typescript
import { computed } from '@angular/core';

// Derivada de una señal
readonly isEditMode = computed(() => this.url() !== null);

// Derivada de otra señal (toSignal)
readonly urlgrpList = computed(() => this.urlgrpSignal()?.data ?? []);

// Derivada de múltiples señales
readonly resumen = computed(() => {
  const nombre = this.formData().name;
  const grupo = this.urlgrpList().find(g => g.id === this.formData().id_UrlGrp);
  return `${nombre} → ${grupo?.name ?? 'Sin grupo'}`;
});
```

### Reglas clave

- Una señal `computed` es **solo lectura**. No tiene `.set()` ni `.update()`.
- Angular solo la recalcula cuando una de sus dependencias cambia (es lazy y eficiente).
- Nunca incluyas efectos secundarios (peticiones HTTP, console.log, etc.) dentro de un `computed`.

---

## 6. Control de Flujo en Template (`@if`, `@for`)

### ¿Qué es?

Son directivas de control de flujo **nativas del template** de Angular, que reemplazan `*ngIf` y `*ngFor`.

### ¿Por qué existe?

Las directivas antiguas (`*ngIf`, `*ngFor`) requerían importar `CommonModule` o `NgIf`/`NgFor` individualmente. Las nuevas sintaxis son parte del lenguaje del template y **no necesitan importaciones**.

### Implementación correcta

```html
<!-- Condicional simple -->
@if (isLoading()) {
  <app-loading-component />
}

<!-- Condicional con else -->
@if (isLoading()) {
  <app-loading-component />
} @else {
  <button type="submit">Crear</button>
}

<!-- Condicional con else if -->
@if (errorMessage()) {
  <app-message-error [message]="errorMessage()!" />
} @else if (successMessage()) {
  <app-message-success [message]="successMessage()!" />
} @else {
  <p>Todo bien</p>
}

<!-- Bucle con track (obligatorio) -->
@for (grp of urlgrpList(); track grp.id) {
  <option [value]="grp.id">{{ grp.name }}</option>
}

<!-- Bucle con bloque vacío -->
@for (item of items(); track item.id) {
  <li>{{ item.name }}</li>
} @empty {
  <li>No hay elementos</li>
}
```

### Reglas clave

- `track` es **obligatorio** en `@for`. Usa el identificador único del elemento (generalmente `item.id`).
- No necesitan importaciones adicionales en el componente.
- Reemplazan completamente `*ngIf` y `*ngFor`. En código nuevo, no uses las versiones antiguas.

---

## 7. Eventos Nativos vs `ngSubmit`

### ¿Qué es el problema?

`(ngSubmit)` es una directiva de Angular que requiere `FormsModule` para funcionar. Si no lo importas, el evento **silenciosamente no se ejecuta**. Es una de las causas más comunes de bugs en formularios.

### La solución: usar el evento nativo `(submit)`

```html
<!-- ✅ Correcto: evento nativo del formulario HTML -->
<form (submit)="$event.preventDefault(); onSubmit()">
  <!-- ... -->
</form>
```

El `$event.preventDefault()` es necesario porque el comportamiento nativo de un formulario HTML al hacer submit es **recargar la página**.

### Cuándo usar cada opción

| Situación | Usa | Necesita importar |
|---|---|---|
| Manejo de estado con señales (sin FormsModule) | `(submit)` nativo | Nada |
| Usando `[(ngModel)]` o `FormGroup` | `(ngSubmit)` | `FormsModule` o `ReactiveFormsModule` |
| Quieres máximo control y claridad | `(click)` en el botón | Nada |

### Ejemplo con `(click)` en el botón (alternativa válida)

```html
<div class="m-4">
  <!-- El form no existe como contenedor de formulario Angular -->
  <input [value]="formData().name" (input)="updateName($event.target.value)" />
  <button type="button" (click)="onSubmit()">Crear</button>
</div>
```

Esta opción es útil cuando no necesitas la semántica del formulario ni la validación nativa del browser.

---

## 8. Manejo de Estado sin Formularios de Angular

### ¿Qué es este patrón?

En lugar de usar `FormBuilder`, `FormControl` o `[(ngModel)]`, el estado del formulario se maneja **íntegramente con señales**. Cada campo del formulario es una propiedad dentro de una señal de objeto.

### ¿Por qué es válido?

Este patrón es completamente legítimo cuando las validaciones son simples y la lógica de estado no requiere la infraestructura de `ReactiveFormsModule`. Es más liviano, más explícito, y se alinea mejor con el modelo de señales de Angular moderno.

### Implementación correcta

```typescript
// Estado del formulario como una sola señal
readonly formData = signal<Partial<UrlModel>>({
  name: '',
  link: '',
  isEnable: true,
  id_UrlGrp: 0,
});

// Cada campo tiene su propia función de actualización
protected updateName(value: string): void {
  this.formData.update((data) => ({ ...data, name: value }));
  this.errorMessage.set(null); // Limpia errores al escribir
}
```

### Template correspondiente

```html
<input
  type="text"
  [value]="formData().name"
  (input)="updateName(($event.target as HTMLInputElement).value)"
/>
```

### Validación manual en onSubmit

```typescript
protected onSubmit(): void {
  const data = this.formData();
  const name = (data.name ?? '').trim();

  if (!name) {
    this.errorMessage.set('El nombre es obligatorio');
    return;  // ← Detiene la ejecución, no hace la petición
  }

  // Si pasa todas las validaciones, continúa con la petición
  this.isLoading.set(true);
  // ...
}
```

### Cuándo usar este patrón vs ReactiveFormsModule

| Usa señales cuando | Usa ReactiveFormsModule cuando |
|---|---|
| Validaciones simples (campo requerido, formato básico) | Validaciones complejas y asíncronas |
| El formulario tiene pocos campos | Necesitas `validators` personalizados |
| Quieres máximo control explícito | El formulario es dinámico (campos que se agregan/eliminan) |
| Quieres menos dependencias | Necesitas integración con librerías de formularios |

---

## 9. Patrones de Comunicación con el Backend

### Crear vs Actualizar según el modo

```typescript
// Determinar si es crear o actualizar basándose en una señal
readonly isEditMode = computed(() => this.url() !== null);

// Construir el payload según el modo
const payload: UrlModel = this.isEditMode()
  ? { id: this.url()!.id, name, link, isEnable, id_UrlGrp }  // Actualizar
  : { id: 0, name, link, isEnable, id_UrlGrp };               // Crear

// Elegir la petición según el modo
const request = this.isEditMode()
  ? this.urlService.update(payload)
  : this.urlService.create(payload);
```

### Suscripción con manejo completo de estados

```typescript
request.subscribe({
  next: (res) => {
    this.isLoading.set(false);

    if (res.isSuccess) {
      this.successMessage.set('Operación exitosa');
      setTimeout(() => this.router.navigate(['/url']), 1500);
    } else {
      // El backend respondió, pero con un error de lógica
      this.errorMessage.set(res.message ?? 'Error al procesar');
    }
  },
  error: (err) => {
    // Error de red o error no controlado
    this.isLoading.set(false);
    this.errorMessage.set(err?.message ?? 'Error de conexión');
  },
});
```

### Pasar datos entre rutas con `history.state`

```typescript
// Cuando navega hacia el formulario (desde otra página)
this.router.navigate(['/url/form'], {
  state: { url: urlSeleccionada }  // ← Pasa el objeto completo
});

// Cuando el formulario se carga, recupera el estado
function getUrlFromHistoryState(): UrlUrlgrpModel | null {
  const state = history.state as { url?: UrlUrlgrpModel } | null;
  return state?.url ?? null;
}
```

Este patrón evita hacer una petición HTTP adicional para obtener los datos del elemento que se está editando.

---

## 10. Estructura Recomendada de un Componente Moderno

Este es el orden y la organización que se recomienda para mantener los componentes legibles y consistentes.

```typescript
@Component({
  selector: 'app-mi-componente',
  imports: [ /* solo lo necesario */ ],
  templateUrl: './mi-componente.html',
})
export class MiComponente {

  // 1. Inyecciones de servicios
  private readonly miServicio = inject(MiServicio);
  private readonly router = inject(Router);

  // 2. Estado inicial (derivado de parámetros de ruta, history.state, etc.)
  private readonly initialData = getDataFromState();

  // 3. Señales de datos remotos (toSignal)
  private readonly datosRemotoSignal = toSignal(
    this.miServicio.getAll(),
    { initialValue: undefined }
  );

  // 4. Señales públicas (las que usa el template)
  readonly isEditMode = computed(() => this.initialData !== null);
  readonly datosList = computed(() => this.datosRemotoSignal()?.data ?? []);
  readonly formData = signal<Partial<MiModel>>({ /* ... */ });

  // 5. Señales de control de UI
  readonly isLoading = signal(false);
  readonly errorMessage = signal<string | null>(null);
  readonly successMessage = signal<string | null>(null);

  // 6. Métodos de actualización de campos (protected, uno por campo)
  protected updateNombre(value: string): void { /* ... */ }
  protected updateDescripcion(value: string): void { /* ... */ }

  // 7. Método de submit (protected)
  protected onSubmit(): void { /* ... */ }
}
```

### Reglas de visibilidad

| Modificador | Usar cuando |
|---|---|
| `private readonly` | Servicios inyectados y señales que solo el componente lee internamente |
| `readonly` (público) | Señales que el template necesita leer |
| `protected` | Métodos y señales que solo el template necesita invocar |
| Sin modificador | Solo si es necesario acceder desde fuera del componente |

---

## Resumen Rápido

| Característica | Reemplaza a | Importación necesaria |
|---|---|---|
| Componentes Standalone | `NgModule` | Ninguna |
| `signal()` | Variables locales + detección de cambios | `signal` de `@angular/core` |
| `computed()` | Getters reactivos | `computed` de `@angular/core` |
| `toSignal()` | `async pipe` | `toSignal` de `@angular/core` |
| `inject()` | Parámetros del constructor | `inject` de `@angular/core` |
| `@if` / `@for` | `*ngIf` / `*ngFor` | Ninguna |
| `(submit)` nativo | `(ngSubmit)` (cuando no se usa FormsModule) | Ninguna |

---

*Documento basado en las prácticas implementadas en el proyecto. Angular 21 — Febrero 2026.*
