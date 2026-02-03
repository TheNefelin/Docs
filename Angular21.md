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

Guía práctica de las características modernas de Angular. Cada sección explica **qué es**, **por qué existe**, **todos sus parámetros y opciones**, y **cómo implementarlo correctamente**.

---

## Índice

1. [Componentes Standalone](#1-componentes-standalone)
2. [Señales — `signal()`](#2-señales--signal)
3. [Señales Derivadas — `computed()`](#3-señales-derivadas--computed)
4. [De Observable a Señal — `toSignal()`](#4-de-observable-a-señal--tosignal)
5. [Inyección con `inject()`](#5-inyección-con-inject)
6. [Control de Flujo en Template (`@if`, `@for`, `@switch`)](#6-control-de-flujo-en-template-if-for-switch)
7. [Estrategias de Formularios — Comparación Profunda](#7-estrategias-de-formularios--comparación-profunda)
   - 7.1 [Estrategia A: Señales + evento nativo (RECOMENDADA)](#71-estrategia-a-señales--evento-nativo-recomendada)
   - 7.2 [Estrategia B: Formularios Reactivos (`ReactiveFormsModule`)](#72-estrategia-b-formularios-reactivos-reactiveformsmodule)
   - 7.3 [Estrategia C: Formularios Template (`FormsModule`)](#73-estrategia-c-formularios-template-formsmodule)
   - 7.4 [Tabla de Decisión](#74-tabla-de-decisión)
8. [Patrones de Comunicación con el Backend](#8-patrones-de-comunicación-con-el-backend)
9. [Estructura Recomendada de un Componente Moderno](#9-estructura-recomendada-de-un-componente-moderno)

---

## 1. Componentes Standalone

### ¿Qué es?

Los componentes standalone eliminan la necesidad de un `NgModule`. Cada componente declara sus propias dependencias directamente en el decorador `@Component`.

### ¿Por qué existe?

Antes, cada componente debía ser declarado en un `@NgModule`, lo cual generaba acoplamiento y archivos de módulo enormes. Los standalone hacen que cada componente sea **autosuficiente**.

### Firma del decorador

```typescript
@Component({
  selector: string,                  // Obligatorio. Etiqueta en el template.
  imports?: (Type | ModuleWithProviders)[], // Componentes, directivas, módulos que usa el template.
  template?: string,                 // Template inline.
  templateUrl?: string,              // Template externo (archivo .html).
  styles?: string[],                 // Estilos inline.
  styleUrls?: string[],              // Estilos externos (archivos .css/.scss).
  standalone?: boolean,              // true por defecto en Angular 19+.
  changeDetection?: ChangeDetectionStrategy, // Default o OnPush.
  providers?: Provider[],            // Servicios locales al componente.
})
```

### Implementación correcta

```typescript
import { Component } from '@angular/core';
import { RouterLink } from '@angular/router';

@Component({
  selector: 'app-url-form-page',
  imports: [
    RouterLink,              // Necesario si uso routerLink en el template
    LoadingComponent,        // Necesario si uso <app-loading-component>
    MessageErrorComponent,
    MessageSuccessComponent,
  ],
  templateUrl: './url-form-page.html',
})
export class UrlFormPage { }
```

### Reglas clave

- Si usas una directiva o componente en el template, **debe estar en `imports`**.
- Si usas `(ngSubmit)` o `[(ngModel)]`, **debe importar `FormsModule`**.
- Si usas `formGroup` o `formControlName`, **debe importar `ReactiveFormsModule`**.
- Si no necesitas ninguna directiva de Angular, **no importes nada**. Menos es más.

---

## 2. Señales — `signal()`

### ¿Qué es?

Una señal es una **celda reactiva** que contiene un valor. Cuando ese valor cambia, Angular actualiza automáticamente solo las partes del template que leen esa señal. Sin Zone.js, sin detección de cambios global.

### Firma

```typescript
import { signal, WritableSignal } from '@angular/core';

// Firma básica
signal<T>(value: T, options?: SignalOptions<T>): WritableSignal<T>

// Opciones disponibles
interface SignalOptions<T> {
  equal?: (prev: T, next: T) => boolean;  // Función custom para comparar si el valor cambió.
                                           // Por defecto usa Object.is().
}
```

### Métodos de una señal escribible (`WritableSignal<T>`)

```typescript
const miSignal = signal<string>('hola');

miSignal()                    // Leer el valor actual → 'hola'
miSignal.set('nuevo valor')   // Reemplazar el valor completamente
miSignal.update(prev => prev + '!') // Actualizar basándose en el valor anterior
miSignal.asReadonly()         // Devuelve una versión solo-lectura (Signal<T>)
```

### Todos los tipos de señal de un vistazo

| Tipo | Firma | Escribible | Uso principal |
|---|---|---|---|
| `WritableSignal<T>` | `signal<T>(valor)` | ✅ `.set()` / `.update()` | Estado local mutable |
| `Signal<T>` | resultado de `.asReadonly()` | ❌ Solo leer | Exponer estado sin permitir escritura externa |
| `computed` | `computed<T>(() => ...)` | ❌ Solo leer | Valores derivados de otras señales |
| `toSignal` | `toSignal<T>(observable$)` | ❌ Solo leer | Datos remotos desde un Observable |

### Implementación correcta

```typescript
import { signal } from '@angular/core';

export class UrlFormPage {
  // Señal con valor primitivo
  readonly isLoading = signal(false);                    // WritableSignal<boolean>
  readonly errorMessage = signal<string | null>(null);   // WritableSignal<string | null>

  // Señal con objeto complejo
  readonly formData = signal<Partial<UrlModel>>({
    name: '',
    link: '',
    isEnable: true,
    id_UrlGrp: 0,
  });

  // Señal con comparador custom (útil para objetos)
  readonly usuario = signal<Usuario | null>(null, {
    equal: (prev, next) => prev?.id === next?.id  // Solo re-emite si cambia el id
  });
}
```

### Cómo leer y escribir

```typescript
// --- En TypeScript ---
const valor = this.isLoading();            // Leer → false
this.isLoading.set(true);                  // Escribir valor completo
this.formData.update((d) => ({ ...d, name: 'nuevo' })); // Escribir basándose en anterior

// --- En el template ---
// Siempre con paréntesis, como una función
[disabled]="isLoading()"
{{ formData().name }}
@if (errorMessage()) { ... }
```

### Señal solo-lectura expuesta al exterior

```typescript
// Internamente escribible, externamente solo lectura
private readonly _contador = signal(0);
readonly contador = this._contador.asReadonly(); // Signal<number>, no tiene .set()

increment(): void {
  this._contador.update(n => n + 1);  // Solo el componente puede escribirla
}
```

### Reglas clave

- Siempre declara con `readonly` para que solo el componente dueño la escriba.
- En el template **siempre con paréntesis**: `isLoading()`, nunca `isLoading`.
- Usa `.set()` cuando reemplazas el valor completo.
- Usa `.update()` cuando el nuevo valor depende del anterior.
- Usa `equal` custom cuando trabajas con objetos y quieres evitar actualizaciones innecesarias.

---

## 3. Señales Derivadas — `computed()`

### ¿Qué es?

`computed()` crea una señal que se recalcula automáticamente cuando **cualquiera de las señales que lee internamente** cambia. Es la forma de crear lógica derivada sin duplicar estado.

### Firma

```typescript
import { computed, Signal } from '@angular/core';

// Firma
computed<T>(computation: () => T, options?: SignalOptions<T>): Signal<T>

// Opciones (mismas que signal)
interface SignalOptions<T> {
  equal?: (prev: T, next: T) => boolean;
}
```

### Comportamiento interno

```
señal A cambia  →  computed se marca como "sucio"  →  la próxima vez que alguien lo lee, se recalcula
```

Es **lazy**: no se recalcula hasta que alguien lo lee. Si nadie lo lee, no se ejecuta.

### Implementación correcta

```typescript
import { computed, signal, toSignal } from '@angular/core';

export class UrlFormPage {
  readonly url = signal<UrlUrlgrpModel | null>(null);

  // Derivada de una señal escribible
  readonly isEditMode = computed(() => this.url() !== null);

  // Derivada de una señal toSignal
  private readonly urlgrpSignal = toSignal(this.urlgrpService.getAll(), { initialValue: undefined });
  readonly urlgrpList = computed(() => this.urlgrpSignal()?.data ?? []);

  // Derivada de múltiples señales (se recalcula si cualquiera cambia)
  readonly formData = signal<Partial<UrlModel>>({ name: '', id_UrlGrp: 0 });

  readonly resumen = computed(() => {
    const nombre = this.formData().name;                                    // lee formData
    const grupo = this.urlgrpList().find(g => g.id === this.formData().id_UrlGrp); // lee urlgrpList
    return `${nombre} → ${grupo?.name ?? 'Sin grupo'}`;
  });

  // Con comparador custom
  readonly nombreUpperCase = computed(
    () => (this.formData().name ?? '').toUpperCase(),
    { equal: (prev, next) => prev === next }
  );
}
```

### Reglas clave

- Una señal `computed` es **solo lectura**. No tiene `.set()` ni `.update()`.
- Nunca incluyas efectos secundarios dentro (peticiones HTTP, console.log, mutaciones).
- Es eficiente: Angular solo la recalcula cuando una dependencia cambia **y** alguien la lee.
- Puede leer otras señales `computed` sin problema (se encadenan).

---

## 4. De Observable a Señal — `toSignal()`

### ¿Qué es?

`toSignal()` convierte un Observable en una señal que Angular puede rastrear reactivamente. Maneja suscripción y desuscripción automáticamente.

### Firma completa

```typescript
import { toSignal, ToSignalOptions } from '@angular/core';

// Firma
toSignal<T>(source: Observable<T>, options?: ToSignalOptions<T>): Signal<T>

// Todas las opciones disponibles
interface ToSignalOptions<T> {
  initialValue?: T;              // Valor antes de que el Observable emita por primera vez.
  requireSync?: boolean;         // Si es true, el Observable DEBE emitir sincrónicamente o lanza error.
  injector?: Injector;           // Injector custom (para usar fuera del contexto de inyección).
  equal?: (prev: T, next: T) => boolean; // Comparador custom.
}
```

### Cómo afecta el tipo según las opciones

```typescript
// Sin initialValue → el tipo incluye undefined
const datos = toSignal(servicio.getAll());
// tipo: Signal<ApiResponseModel<UrlGrpModel[]> | undefined>

// Con initialValue → el tipo NO incluye undefined
const datos = toSignal(servicio.getAll(), { initialValue: { data: [], isSuccess: true } });
// tipo: Signal<ApiResponseModel<UrlGrpModel[]>>

// Con requireSync → Angular verifica que el Observable emita inmediatamente
const datos = toSignal(servicio.getAll(), { requireSync: true });
// Útil con BehaviorSubject o ShareReplay(1), lanza error si no emite sincrónicamente
```

### Implementación correcta

```typescript
import { toSignal } from '@angular/core';
import { catchError } from 'rxjs/operators';
import { of } from 'rxjs';

export class UrlFormPage {
  private readonly urlgrpService = inject(UrlGrpService);

  // Patrón recomendado: toSignal con catchError en el pipe
  private readonly urlgrpSignal = toSignal(
    this.urlgrpService.getAll().pipe(
      catchError((err) => {
        console.error('Error cargando grupos:', err);
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

  // Señal derivada que extrae solo la lista
  readonly urlgrpList = computed(() => this.urlgrpSignal()?.data ?? []);
}
```

### Usando `injector` para usar fuera del contexto de inyección

```typescript
import { toSignal, inject } from '@angular/core';
import { Injector } from '@angular/core';

export class MiComponente {
  private readonly injector = inject(Injector);

  // Si por alguna razón necesitas crear toSignal dentro de un método:
  private datos!: Signal<any>;

  ngOnInit(): void {
    this.datos = toSignal(
      this.miServicio.getAll(),
      { injector: this.injector }  // ← Necesario fuera de la zona de inicialización
    );
  }
}
```

### Reglas clave

- Se suscribe automáticamente al crear el componente y se desustacribe al destruirlo.
- Si no provienes `initialValue`, el tipo incluye `| undefined`.
- **No es para peticiones que dispara el usuario** (como POST/PUT). Para esos, usa `.subscribe()` directamente.
- El `catchError` dentro del pipe es esencial: si el Observable falla sin catch, la señal queda en estado de error y no se recupera.

---

## 5. Inyección con `inject()`

### ¿Qué es?

`inject()` obtiene una instancia de un servicio dentro del **contexto de inyección** de un componente, sin usar el constructor.

### Firma

```typescript
import { inject } from '@angular/core';

// Firma
inject<T>(token: InjectionToken<T> | Type<T>, options?: InjectOptions): T

// Opciones
interface InjectOptions {
  optional?: boolean;   // Si es true, retorna null en lugar de lanzar error si no existe el servicio
  self?: boolean;       // Solo busca en el injector del componente actual, no sube al padre
  skipSelf?: boolean;   // Busca solo en el injector padre, no en el actual
}
```

### Implementación correcta

```typescript
import { inject } from '@angular/core';

export class UrlFormPage {
  // ✅ Básico: obtener un servicio
  private readonly urlService = inject(UrlService);
  private readonly router = inject(Router);

  // ✅ Con optional: el servicio puede no existir
  private readonly logger = inject(LoggerService, { optional: true });
  // tipo: LoggerService | null

  // ✅ Con token de inyección custom
  private readonly apiUrl = inject(API_URL_TOKEN);  // donde API_URL_TOKEN es un InjectionToken<string>
}
```

### Dónde es válido usar `inject()`

`inject()` solo funciona dentro del **contexto de inyección**:

```typescript
export class MiComponente {
  // ✅ Válido: zona de inicialización de propiedades
  private readonly svc = inject(MiServicio);

  // ✅ Válido: constructor (se ejecuta dentro del contexto)
  constructor() {
    const otra = inject(OtroServicio);
  }

  // ✅ Válido: función llamada sincrónicamente desde la zona de inicialización
  private readonly datos = this.cargarInicial();
  private cargarInicial() {
    const svc = inject(MiServicio);  // OK porque se ejecuta sincrónicamente durante la init
    return svc.getInitial();
  }

  // ❌ Inválido: dentro de un método que se ejecuta después de la inicialización
  ngOnInit(): void {
    const svc = inject(MiServicio);  // ERROR en runtime
  }

  // ❌ Inválido: dentro de un callback asíncrono
  async cargarDatos(): Promise<void> {
    const svc = inject(MiServicio);  // ERROR en runtime
  }
}
```

### Reglas clave

- Es equivalente a declarar parámetros en el constructor, pero más limpio con señales.
- Si un servicio es opcional, usa `{ optional: true }` y maneja el `null`.
- Nunca lo uses dentro de métodos que se ejecutan después de la inicialización del componente.

---

## 6. Control de Flujo en Template (`@if`, `@for`, `@switch`)

### ¿Qué es?

Son directivas de control de flujo **nativas del lenguaje del template** de Angular. Reemplazan `*ngIf`, `*ngFor` y `*ngSwitch`. No necesitan importaciones.

### `@if` — Condicional

```html
<!-- Simple -->
@if (isLoading()) {
  <app-loading-component />
}

<!-- Con else -->
@if (isLoading()) {
  <app-loading-component />
} @else {
  <button type="submit">Crear</button>
}

<!-- Con else if (encadenados) -->
@if (errorMessage()) {
  <app-message-error [message]="errorMessage()!" />
} @else if (successMessage()) {
  <app-message-success [message]="successMessage()!" />
} @else {
  <p>Listo para enviar</p>
}
```

### `@for` — Bucle

```html
<!-- Básico. track es OBLIGATORIO -->
@for (grp of urlgrpList(); track grp.id) {
  <option [value]="grp.id">{{ grp.name }}</option>
}

<!-- Con bloque @empty (cuando la lista está vacía) -->
@for (item of items(); track item.id) {
  <li>{{ item.name }}</li>
} @empty {
  <li>No hay elementos disponibles</li>
}

<!-- Variables de contexto automáticas dentro de @for -->
@for (item of items(); track item.id) {
  <li>
    {{ $index }}    <!-- Índice actual (0, 1, 2...) -->
    {{ $count }}    <!-- Cantidad total de elementos -->
    {{ $first }}    <!-- true si es el primer elemento -->
    {{ $last }}     <!-- true si es el último elemento -->
    {{ $even }}     <!-- true si el índice es par -->
    {{ $odd }}      <!-- true si el índice es impar -->
    {{ item.name }}
  </li>
}
```

### `@switch` — Condicional múltiple

```html
@switch (formData().estado) {
  @case ('pendiente') {
    <span class="badge badge-warning">Pendiente</span>
  }
  @case ('activo') {
    <span class="badge badge-success">Activo</span>
  }
  @case ('inactivo') {
    <span class="badge badge-danger">Inactivo</span>
  }
  @default {
    <span class="badge">Desconocido</span>
  }
}
```

### Reglas clave

- `track` es **obligatorio** en `@for`. Usa el identificador único del elemento.
- No necesitan ninguna importación en el componente.
- Reemplazan completamente `*ngIf`, `*ngFor`, `[ngSwitch]`. En código nuevo, no uses las versiones antiguas.
- Las variables de contexto (`$index`, `$first`, etc.) solo existen dentro del bloque `@for`.

---

## 7. Estrategias de Formularios — Comparación Profunda

Esta es la sección más importante para decidir cómo construir formularios en Angular 21. Hay tres estrategias reales. Cada una se explica completa, con código de ejemplo del **mismo formulario** implementado de las tres formas, para que la comparación sea directa.

El formulario de ejemplo en las tres estrategias es idéntico: nombre, enlace, grupo (select), activo (checkbox), con validación y petición al backend.

---

### 7.1 Estrategia A: Señales + evento nativo ⭐ RECOMENDADA

#### ¿Cuándo usar esta estrategia?

Es la más recomendada cuando las validaciones son simples, el formulario tiene una cantidad fija de campos, y quieres un código que sea explícito, liviano y completamente alineado con el modelo de señales de Angular 21. Es lo que implementas en tu proyecto actual y es la decisión correcta.

#### Cómo funciona internamente

```
Usuario escribe en input
    ↓
(input) dispara evento nativo del DOM
    ↓
Llama a updateNombre(valor)
    ↓
formData.update() actualiza la señal
    ↓
Angular detecta el cambio y actualiza el template
    ↓
Usuario hace clic en "Crear"
    ↓
(submit) del form nativo dispara onSubmit()
    ↓
onSubmit() lee formData(), valida, hace petición
```

#### Componente — TypeScript

```typescript
import { Component, signal, computed, inject } from '@angular/core';
import { Router, RouterLink } from '@angular/router';

@Component({
  selector: 'app-url-form-page',
  imports: [RouterLink, LoadingComponent, MessageErrorComponent, MessageSuccessComponent],
  templateUrl: './url-form-page.html',
})
export class UrlFormPage {
  private readonly urlService  = inject(UrlService);
  private readonly urlgrpService = inject(UrlGrpService);
  private readonly router      = inject(Router);

  private readonly initialUrl = getUrlFromHistoryState();

  // Datos remotos
  private readonly urlgrpSignal = toSignal(
    this.urlgrpService.getAll().pipe(catchError((err) => of({ data: [] } as ApiResponseModel<UrlGrpModel[]>))),
    { initialValue: undefined }
  );

  // Estado
  readonly url         = signal<UrlUrlgrpModel | null>(this.initialUrl);
  readonly isEditMode  = computed(() => this.url() !== null);
  readonly urlgrpList  = computed(() => this.urlgrpSignal()?.data ?? []);

  readonly formData = signal<Partial<UrlModel>>({
    name:       this.initialUrl?.name ?? '',
    link:       this.initialUrl?.link ?? '',
    isEnable:   this.initialUrl?.isEnable ?? true,
    id_UrlGrp:  this.initialUrl?.UrlGrp?.id ?? 0,
  });

  readonly isLoading      = signal(false);
  readonly errorMessage   = signal<string | null>(null);
  readonly successMessage = signal<string | null>(null);

  // Actualizadores de campo
  protected updateName(value: string): void {
    this.formData.update((d) => ({ ...d, name: value }));
    this.errorMessage.set(null);
  }
  protected updateLink(value: string): void {
    this.formData.update((d) => ({ ...d, link: value }));
    this.errorMessage.set(null);
  }
  protected updateIsEnable(value: boolean): void {
    this.formData.update((d) => ({ ...d, isEnable: value }));
  }
  protected updateIdUrlGrp(value: number): void {
    this.formData.update((d) => ({ ...d, id_UrlGrp: value }));
    this.errorMessage.set(null);
  }

  // Submit
  protected onSubmit(): void {
    const data     = this.formData();
    const name     = (data.name ?? '').trim();
    const link     = (data.link ?? '').trim();
    const id_UrlGrp = data.id_UrlGrp ?? 0;

    // Validaciones
    if (!name)          { this.errorMessage.set('El nombre es obligatorio');       return; }
    if (!link)          { this.errorMessage.set('El enlace es obligatorio');       return; }
    if (id_UrlGrp === 0){ this.errorMessage.set('Debe seleccionar un grupo');     return; }

    this.isLoading.set(true);
    this.errorMessage.set(null);
    this.successMessage.set(null);

    const payload: UrlModel = {
      id:         this.isEditMode() ? this.url()!.id : 0,
      name, link,
      isEnable:   data.isEnable ?? true,
      id_UrlGrp,
    };

    const request$ = this.isEditMode()
      ? this.urlService.update(payload)
      : this.urlService.create(payload);

    request$.subscribe({
      next: (res) => {
        this.isLoading.set(false);
        if (res.isSuccess) {
          this.successMessage.set(this.isEditMode() ? 'Url actualizada' : 'Url creada');
          setTimeout(() => this.router.navigate(['/url']), 1500);
        } else {
          this.errorMessage.set(res.message ?? 'Error al procesar');
        }
      },
      error: (err) => {
        this.isLoading.set(false);
        this.errorMessage.set(err?.message ?? 'Error de conexión');
      },
    });
  }
}
```

#### Componente — Template HTML

```html
<section>
  <div class="pt-16 m-4 flex justify-between items-center">
    <h1 class="text-2xl font-bold">{{ isEditMode() ? 'Modificar' : 'Crear' }} Url</h1>
    <button routerLink="/url" class="btn btn-primary m-4">Volver</button>
  </div>

  @if (errorMessage()) {
    <app-message-error-component [message]="errorMessage()!" class="m-4" />
  }
  @if (successMessage()) {
    <app-message-success-component [message]="successMessage()!" class="m-4" />
  }

  <!-- (submit) nativo + preventDefault. No necesita FormsModule -->
  <form (submit)="$event.preventDefault(); onSubmit()" class="m-4">
    <fieldset class="fieldset bg-base-200 border-base-300 rounded-box border p-4 max-w-md" [disabled]="isLoading()">

      <label class="label" for="name">Nombre</label>
      <input
        id="name"
        type="text"
        class="input input-bordered w-full"
        placeholder="Mi página web"
        [value]="formData().name"
        (input)="updateName(($event.target as HTMLInputElement).value)"
      />

      <label class="label" for="link">Enlace</label>
      <input
        id="link"
        type="url"
        class="input input-bordered w-full"
        placeholder="https://ejemplo.com"
        [value]="formData().link"
        (input)="updateLink(($event.target as HTMLInputElement).value)"
      />

      <label class="label" for="id_UrlGrp">Grupo</label>
      <select
        id="id_UrlGrp"
        class="select select-bordered w-full"
        [value]="formData().id_UrlGrp"
        (change)="updateIdUrlGrp(+($event.target as HTMLSelectElement).value)"
      >
        <option [value]="0">Seleccione un grupo</option>
        @for (grp of urlgrpList(); track grp.id) {
          <option [value]="grp.id">{{ grp.name }}</option>
        }
      </select>

      <label class="label py-4 cursor-pointer justify-start gap-2">
        <input
          type="checkbox"
          class="checkbox checkbox-primary"
          [checked]="formData().isEnable"
          (change)="updateIsEnable(($event.target as HTMLInputElement).checked)"
        />
        Activo
      </label>

      <div class="form-control w-full mt-6 flex flex-col sm:flex-row gap-2">
        @if (isLoading()) {
          <app-loading-component />
        } @else {
          <button type="submit" class="btn btn-primary flex-1">
            {{ isEditMode() ? 'Modificar' : 'Crear' }}
          </button>
        }
      </div>
    </fieldset>
  </form>
</section>
```

#### ¿Por qué es la recomendada?

- **Cero dependencias extra**: no importa `FormsModule` ni `ReactiveFormsModule`.
- **Máximo control explícito**: cada campo tiene su función de actualización, cada validación es clara y visible.
- **Alineado con señales**: todo el estado es señales, todo el template es reactivo por señales. Es coherente.
- **Fácil de testear**: puedes llamar a `onSubmit()` directamente sin necesidad de simular un formulario Angular.
- **Ligero en bundle**: no carga la infraestructura de formularios de Angular.

---

### 7.2 Estrategia B: Formularios Reactivos (`ReactiveFormsModule`)

#### ¿Cuándo usar esta estrategia?

Cuando necesitas validaciones complejas (asíncronas, entre campos), formularios dinámicos donde los campos se agregan o eliminan en runtime, o integración con librerías de validación como Zod o Yup.

#### Cómo funciona internamente

```
FormBuilder crea el FormGroup con FormControls
    ↓
Cada FormControl tiene sus propios Validators
    ↓
El template se conecta con formGroup / formControlName
    ↓
Los FormControls se actualizan automáticamente cuando el usuario escribe
    ↓
onSubmit() lee form.value (ya es un objeto tipado)
    ↓
Las validaciones se ejecutan en tiempo real, Angular gestiona los estados (valid, invalid, pending)
```

#### Componente — TypeScript

```typescript
import { Component, inject, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators, ReactiveFormsModule } from '@angular/forms';
import { Router, RouterLink } from '@angular/router';

@Component({
  selector: 'app-url-form-page',
  imports: [
    RouterLink,
    ReactiveFormsModule,       // ← OBLIGATORIO para usar formGroup/formControlName
    LoadingComponent,
    MessageErrorComponent,
    MessageSuccessComponent,
  ],
  templateUrl: './url-form-page.html',
})
export class UrlFormPage implements OnInit {
  private readonly fb            = inject(FormBuilder);
  private readonly urlService    = inject(UrlService);
  private readonly urlgrpService = inject(UrlGrpService);
  private readonly router        = inject(Router);

  private readonly initialUrl = getUrlFromHistoryState();

  // Lista de grupos (puede ser señal o observable, aquí con señal)
  private readonly urlgrpSignal = toSignal(
    this.urlgrpService.getAll().pipe(catchError(() => of({ data: [] } as ApiResponseModel<UrlGrpModel[]>))),
    { initialValue: undefined }
  );
  readonly urlgrpList = computed(() => this.urlgrpSignal()?.data ?? []);

  // Formulario reactivo
  form!: FormGroup;

  readonly isEditMode  = this.initialUrl !== null;
  readonly isLoading      = signal(false);
  readonly errorMessage   = signal<string | null>(null);
  readonly successMessage = signal<string | null>(null);

  ngOnInit(): void {
    this.form = this.fb.group({
      name:      [this.initialUrl?.name ?? '',      [Validators.required, Validators.minLength(2)]],
      link:      [this.initialUrl?.link ?? '',      [Validators.required, Validators.url]],
      id_UrlGrp: [this.initialUrl?.UrlGrp?.id ?? 0, [Validators.required, Validators.min(1)]],
      isEnable:  [this.initialUrl?.isEnable ?? true],
    });
  }

  // Acceso directo a los controles (opcional, para mostrar errores por campo)
  get nameControl()      { return this.form.controls['name']; }
  get linkControl()      { return this.form.controls['link']; }
  get idUrlGrpControl()  { return this.form.controls['id_UrlGrp']; }

  protected onSubmit(): void {
    if (this.form.invalid) {
      this.form.markAllAsTouched();  // Muestra los errores visualmente
      return;
    }

    this.isLoading.set(true);
    this.errorMessage.set(null);

    const formValue = this.form.value;
    const payload: UrlModel = {
      id:         this.isEditMode ? this.initialUrl!.id : 0,
      name:       formValue.name.trim(),
      link:       formValue.link.trim(),
      isEnable:   formValue.isEnable,
      id_UrlGrp:  formValue.id_UrlGrp,
    };

    const request$ = this.isEditMode
      ? this.urlService.update(payload)
      : this.urlService.create(payload);

    request$.subscribe({
      next: (res) => {
        this.isLoading.set(false);
        if (res.isSuccess) {
          this.successMessage.set(this.isEditMode ? 'Url actualizada' : 'Url creada');
          setTimeout(() => this.router.navigate(['/url']), 1500);
        } else {
          this.errorMessage.set(res.message ?? 'Error al procesar');
        }
      },
      error: (err) => {
        this.isLoading.set(false);
        this.errorMessage.set(err?.message ?? 'Error de conexión');
      },
    });
  }
}
```

#### Componente — Template HTML

```html
<section>
  <div class="pt-16 m-4 flex justify-between items-center">
    <h1 class="text-2xl font-bold">{{ isEditMode ? 'Modificar' : 'Crear' }} Url</h1>
    <button routerLink="/url" class="btn btn-primary m-4">Volver</button>
  </div>

  @if (errorMessage()) {
    <app-message-error-component [message]="errorMessage()!" class="m-4" />
  }
  @if (successMessage()) {
    <app-message-success-component [message]="successMessage()!" class="m-4" />
  }

  <!-- (ngSubmit) es válido porque importamos ReactiveFormsModule -->
  <form [formGroup]="form" (ngSubmit)="onSubmit()" class="m-4">
    <fieldset class="fieldset bg-base-200 border-base-300 rounded-box border p-4 max-w-md" [disabled]="isLoading()">

      <label class="label" for="name">Nombre</label>
      <input
        id="name"
        formControlName="name"
        type="text"
        class="input input-bordered w-full"
        placeholder="Mi página web"
      />
      <!-- Errores de validación por campo -->
      @if (nameControl.invalid && nameControl.touched) {
        <p class="text-red-500 text-sm mt-1">
          @if (nameControl.errors?.['required']) { El nombre es obligatorio }
          @if (nameControl.errors?.['minlength']) { Mínimo 2 caracteres }
        </p>
      }

      <label class="label" for="link">Enlace</label>
      <input
        id="link"
        formControlName="link"
        type="text"
        class="input input-bordered w-full"
        placeholder="https://ejemplo.com"
      />
      @if (linkControl.invalid && linkControl.touched) {
        <p class="text-red-500 text-sm mt-1">
          @if (linkControl.errors?.['required']) { El enlace es obligatorio }
          @if (linkControl.errors?.['url'])      { Debe ser una URL válida }
        </p>
      }

      <label class="label" for="id_UrlGrp">Grupo</label>
      <select
        id="id_UrlGrp"
        formControlName="id_UrlGrp"
        class="select select-bordered w-full"
      >
        <option [value]="0">Seleccione un grupo</option>
        @for (grp of urlgrpList(); track grp.id) {
          <option [value]="grp.id">{{ grp.name }}</option>
        }
      </select>
      @if (idUrlGrpControl.invalid && idUrlGrpControl.touched) {
        <p class="text-red-500 text-sm mt-1">Debe seleccionar un grupo</p>
      }

      <label class="label py-4 cursor-pointer justify-start gap-2">
        <input
          type="checkbox"
          formControlName="isEnable"
          class="checkbox checkbox-primary"
        />
        Activo
      </label>

      <div class="form-control w-full mt-6 flex flex-col sm:flex-row gap-2">
        @if (isLoading()) {
          <app-loading-component />
        } @else {
          <button type="submit" class="btn btn-primary flex-1">
            {{ isEditMode ? 'Modificar' : 'Crear' }}
          </button>
        }
      </div>
    </fieldset>
  </form>
</section>
```

#### Ventajas y costos

**Ventajas**: validaciones declarativas con `Validators`, errores por campo automáticos, `markAllAsTouched()`, validadores asíncronos, `FormArray` para campos dinámicos.

**Costos**: importa `ReactiveFormsModule` (más peso en bundle), el estado del formulario vive en `FormGroup` (fuera del mundo de señales), necesita `ngOnInit` para inicializar, más boilerplate en el template con `formControlName`.

---

### 7.3 Estrategia C: Formularios Template (`FormsModule`)

#### ¿Cuándo usar esta estrategia?

Es la más antigua y la menos recomendada en Angular 21. Existe por compatibilidad. No la eliges para código nuevo.

#### Cómo funciona internamente

```
[(ngModel)] conecta directamente el input con una propiedad de la clase
    ↓
Angular gestiona la suscripción bidireccional automáticamente
    ↓
(ngSubmit) captura el submit del formulario
    ↓
onSubmit() lee directamente las propiedades de la clase
```

#### Componente — TypeScript

```typescript
import { Component, inject } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { Router, RouterLink } from '@angular/router';

@Component({
  selector: 'app-url-form-page',
  imports: [
    RouterLink,
    FormsModule,               // ← OBLIGATORIO para usar [(ngModel)] y (ngSubmit)
    LoadingComponent,
    MessageErrorComponent,
    MessageSuccessComponent,
  ],
  templateUrl: './url-form-page.html',
})
export class UrlFormPage {
  private readonly urlService    = inject(UrlService);
  private readonly urlgrpService = inject(UrlGrpService);
  private readonly router        = inject(Router);

  private readonly initialUrl = getUrlFromHistoryState();

  private readonly urlgrpSignal = toSignal(
    this.urlgrpService.getAll().pipe(catchError(() => of({ data: [] } as ApiResponseModel<UrlGrpModel[]>))),
    { initialValue: undefined }
  );
  readonly urlgrpList = computed(() => this.urlgrpSignal()?.data ?? []);

  readonly isEditMode = this.initialUrl !== null;

  // Estado como propiedades directas de la clase (no señales)
  name:      string  = this.initialUrl?.name ?? '';
  link:      string  = this.initialUrl?.link ?? '';
  isEnable:  boolean = this.initialUrl?.isEnable ?? true;
  id_UrlGrp: number  = this.initialUrl?.UrlGrp?.id ?? 0;

  readonly isLoading      = signal(false);
  readonly errorMessage   = signal<string | null>(null);
  readonly successMessage = signal<string | null>(null);

  protected onSubmit(): void {
    const name = this.name.trim();
    const link = this.link.trim();

    if (!name)              { this.errorMessage.set('El nombre es obligatorio');   return; }
    if (!link)              { this.errorMessage.set('El enlace es obligatorio');   return; }
    if (this.id_UrlGrp === 0) { this.errorMessage.set('Debe seleccionar un grupo'); return; }

    this.isLoading.set(true);
    this.errorMessage.set(null);

    const payload: UrlModel = {
      id:        this.isEditMode ? this.initialUrl!.id : 0,
      name, link,
      isEnable:  this.isEnable,
      id_UrlGrp: this.id_UrlGrp,
    };

    const request$ = this.isEditMode
      ? this.urlService.update(payload)
      : this.urlService.create(payload);

    request$.subscribe({
      next: (res) => {
        this.isLoading.set(false);
        if (res.isSuccess) {
          this.successMessage.set(this.isEditMode ? 'Url actualizada' : 'Url creada');
          setTimeout(() => this.router.navigate(['/url']), 1500);
        } else {
          this.errorMessage.set(res.message ?? 'Error al procesar');
        }
      },
      error: (err) => {
        this.isLoading.set(false);
        this.errorMessage.set(err?.message ?? 'Error de conexión');
      },
    });
  }
}
```

#### Componente — Template HTML

```html
<section>
  <div class="pt-16 m-4 flex justify-between items-center">
    <h1 class="text-2xl font-bold">{{ isEditMode ? 'Modificar' : 'Crear' }} Url</h1>
    <button routerLink="/url" class="btn btn-primary m-4">Volver</button>
  </div>

  @if (errorMessage()) {
    <app-message-error-component [message]="errorMessage()!" class="m-4" />
  }
  @if (successMessage()) {
    <app-message-success-component [message]="successMessage()!" class="m-4" />
  }

  <!-- (ngSubmit) funciona porque importamos FormsModule -->
  <form (ngSubmit)="onSubmit()" class="m-4">
    <fieldset class="fieldset bg-base-200 border-base-300 rounded-box border p-4 max-w-md" [disabled]="isLoading()">

      <label class="label" for="name">Nombre</label>
      <input
        id="name"
        type="text"
        class="input input-bordered w-full"
        placeholder="Mi página web"
        [(ngModel)]="name"
        name="name"             <!-- name es OBLIGATORIO con ngModel dentro de un form -->
      />

      <label class="label" for="link">Enlace</label>
      <input
        id="link"
        type="url"
        class="input input-bordered w-full"
        placeholder="https://ejemplo.com"
        [(ngModel)]="link"
        name="link"
      />

      <label class="label" for="id_UrlGrp">Grupo</label>
      <select
        id="id_UrlGrp"
        class="select select-bordered w-full"
        [(ngModel)]="id_UrlGrp"
        name="id_UrlGrp"
      >
        <option [value]="0">Seleccione un grupo</option>
        @for (grp of urlgrpList(); track grp.id) {
          <option [value]="grp.id">{{ grp.name }}</option>
        }
      </select>

      <label class="label py-4 cursor-pointer justify-start gap-2">
        <input
          type="checkbox"
          class="checkbox checkbox-primary"
          [(ngModel)]="isEnable"
          name="isEnable"
        />
        Activo
      </label>

      <div class="form-control w-full mt-6 flex flex-col sm:flex-row gap-2">
        @if (isLoading()) {
          <app-loading-component />
        } @else {
          <button type="submit" class="btn btn-primary flex-1">
            {{ isEditMode ? 'Modificar' : 'Crear' }}
          </button>
        }
      </div>
    </fieldset>
  </form>
</section>
```

#### Ventajas y costos

**Ventajas**: menos código que Reactive, la conexión es automática con `[(ngModel)]`, sencilla de entender para principiantes.

**Costos**: la lógica vive en el template (difícil de testear), no tiene validaciones declarativas robustas, el estado es propiedades mutables de la clase (no señales), no es el patrón recomendado en Angular moderno. Es la estrategia menos alineada con la visión actual de Angular.

---

### 7.4 Tabla de Decisión

| Criterio | A: Señales ⭐ | B: Reactive | C: Template |
|---|---|---|---|
| Importación necesaria | Ninguna | `ReactiveFormsModule` | `FormsModule` |
| Estado del formulario | `signal()` | `FormGroup` | Propiedades de la clase |
| Validaciones | Manual en `onSubmit()` | Declarativas con `Validators` | Muy limitadas |
| Validaciones asíncronas | Tú las escribes | ✅ Soportadas nativamente | ❌ |
| Campos dinámicos | Tú los gestiones | ✅ `FormArray` | ❌ |
| Errores por campo | Tú los gestiones | ✅ Automáticos | Muy limitado |
| Testabilidad | ✅ Alta (todo es explícito) | Media (necesita TestBed con forms) | Baja |
| Peso en bundle | Mínimo | Medio | Medio |
| Alineación con Angular 21 | ✅ Máxima | Media | Baja |
| Recomendado para | 80% de los formularios | Formularios complejos con validación | Legacy / migración |

---

## 8. Patrones de Comunicación con el Backend

### Crear vs Actualizar según el modo

```typescript
readonly isEditMode = computed(() => this.url() !== null);

const payload: UrlModel = {
  id:        this.isEditMode() ? this.url()!.id : 0,
  name, link, isEnable, id_UrlGrp,
};

const request$ = this.isEditMode()
  ? this.urlService.update(payload)
  : this.urlService.create(payload);
```

### Suscripción con manejo completo de estados

```typescript
request$.subscribe({
  next: (res) => {
    this.isLoading.set(false);
    if (res.isSuccess) {
      this.successMessage.set('Operación exitosa');
      setTimeout(() => this.router.navigate(['/url']), 1500);
    } else {
      this.errorMessage.set(res.message ?? 'Error al procesar');
    }
  },
  error: (err) => {
    this.isLoading.set(false);
    this.errorMessage.set(err?.message ?? 'Error de conexión');
  },
});
```

### Pasar datos entre rutas con `history.state`

```typescript
// Desde la página que navega al formulario
this.router.navigate(['/url/form'], {
  state: { url: urlSeleccionada }
});

// En el formulario, recupera el estado al cargarse
function getUrlFromHistoryState(): UrlUrlgrpModel | null {
  const state = history.state as { url?: UrlUrlgrpModel } | null;
  return state?.url ?? null;
}
```

Este patrón evita una petición HTTP extra para obtener los datos del elemento que se edita. Los datos ya viajaron en la navegación.

---

## 9. Estructura Recomendada de un Componente Moderno

Este es el orden y la organización que se recomienda mantener en todos los componentes.

```typescript
import { Component, signal, computed, inject } from '@angular/core';
import { toSignal } from '@angular/core';

@Component({
  selector: 'app-mi-componente',
  imports: [ /* solo lo necesario */ ],
  templateUrl: './mi-componente.html',
})
export class MiComponente {

  // ─── 1. Inyecciones ──────────────────────────────────
  private readonly miServicio = inject(MiServicio);
  private readonly router     = inject(Router);

  // ─── 2. Estado inicial (history.state, params de ruta) ──
  private readonly initialData = getDataFromState();

  // ─── 3. Señales de datos remotos (toSignal) ─────────
  private readonly datosRemotoSignal = toSignal(
    this.miServicio.getAll(),
    { initialValue: undefined }
  );

  // ─── 4. Señales derivadas (computed) ─────────────────
  readonly isEditMode  = computed(() => this.initialData !== null);
  readonly datosList   = computed(() => this.datosRemotoSignal()?.data ?? []);

  // ─── 5. Estado del formulario ────────────────────────
  readonly formData = signal<Partial<MiModel>>({ /* valores iniciales */ });

  // ─── 6. Estado de UI ─────────────────────────────────
  readonly isLoading      = signal(false);
  readonly errorMessage   = signal<string | null>(null);
  readonly successMessage = signal<string | null>(null);

  // ─── 7. Actualizadores de campo (protected) ─────────
  protected updateNombre(value: string): void { /* ... */ }
  protected updateDescripcion(value: string): void { /* ... */ }

  // ─── 8. Submit (protected) ───────────────────────────
  protected onSubmit(): void { /* ... */ }
}
```

### Reglas de visibilidad

| Modificador | Usar cuando |
|---|---|
| `private readonly` | Servicios inyectados y señales que solo el componente usa internamente |
| `readonly` (público) | Señales que el template necesita **leer** |
| `protected` | Métodos y propiedades que solo el **template** necesita invocar |
| Sin modificador | Solo si algo externo al componente necesita acceso directo |

---

## Resumen de importaciones

Esta tabla te dice exactamente qué importar según lo que uses:

| Si usas esto en el template... | Importa esto en `imports[]` |
|---|---|
| `routerLink` | `RouterLink` |
| `routerLinkActive` | `RouterLinkActive` |
| `<router-outlet>` | `RouterModule` |
| `(ngSubmit)` | `FormsModule` |
| `[(ngModel)]` | `FormsModule` |
| `[formGroup]` / `formControlName` | `ReactiveFormsModule` |
| `*ngIf` (antiguo) | `NgIf` |
| `*ngFor` (antiguo) | `NgFor` |
| `\| async` | `AsyncPipe` |
| `@if` / `@for` / `@switch` | **Nada** (es parte del lenguaje del template) |
| `(submit)` nativo | **Nada** (es un evento del DOM) |
| `(click)` nativo | **Nada** (es un evento del DOM) |
| `signal()` / `computed()` | **Nada** (se importan en TypeScript, no en `imports[]`) |

---

*Documento basado en las prácticas implementadas en el proyecto. Angular 21 — Febrero 2026.*