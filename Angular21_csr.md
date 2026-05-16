# Angular 21+ - Guía Completa de Desarrollo Moderno

Guía práctica exhaustiva de Angular 21+. Cubre desde las señales hasta guards, interceptores, control de flujo nativo, y patrones de comunicación con el backend. Cada sección explica **qué es**, **por qué existe**, **todas sus firmas y opciones**, y **cómo implementarlo correctamente**.

> **Nota:** Esta guía es para proyectos sin SSR (Client-Side Rendering). Para SSR, consulta `skill_angular-21-fullstack.md`.

---

## Índice

1. [Configuración Inicial](#1-configuración-inicial)
2. [Señales — `signal()`](#2-señales--signal)
3. [Señales Derivadas — `computed()`](#3-señales-derivadas--computed)
4. [De Observable a Señal — `toSignal()`](#4-de-observable-a-señal--tosignal)
5. [Inyección con `inject()`](#5-inyección-con-inject)
6. [Control de Flujo en Template (`@if`, `@for`, `@switch`)](#6-control-de-flujo-en-template-if-for-switch)
7. [Enrutamiento Moderno — `provideRouter` y `Routes`](#7-enrutamiento-moderno--providerouter-y-routes)
8. [Guards Funcionales](#8-guards-funcionales)
9. [Interceptores Funcionales](#9-interceptores-funcionales)
10. [`HttpClient` Moderno](#10-httpclient-moderno)
11. [Formularios: Señales vs Reactive Forms](#11-formularios-señales-vs-reactive-forms)
12. [Detección de Cambios — `OnPush`](#12-detección-de-cambios--onpush)
13. [Estructura de Proyecto](#13-estructura-de-proyecto)
14. [Componentes Standalone](#14-componentes-standalone)
15. [Routing y Navegación](#15-routing-y-navegación)
16. [Servicios y HTTP](#16-servicios-y-http)
17. [Mejores Prácticas](#17-mejores-prácticas)

---

## 1. Configuración Inicial

### Crear Proyecto
```bash
# Sin SSR (recomendado para SPAs internas)
ng new mi-app --no-ssr --routing --style=scss

# Con pnpm (más rápido y seguro)
pnpm create @angular/latest mi-app --style=scss --ssr

# Agregar Angular Material
ng add @angular/material
```

### Estructura Recomendada
```
src/
├── app/
│   ├── core/                    # Singleton services, guards, interceptors
│   │   ├── services/
│   │   │   ├── api.service.ts
│   │   │   └── auth.service.ts
│   │   ├── guards/
│   │   └── interceptors/
│   ├── shared/                  # Componentes reutilizables
│   │   ├── components/
│   │   ├── pipes/
│   │   └── directives/
│   ├── features/                # Dominios de negocio
│   │   ├── users/
│   │   ├── products/
│   │   └── dashboard/
│   ├── layouts/                 # Layout components
│   ├── app.component.ts
│   ├── app.routes.ts
│   └── app.config.ts
└── environments/
```

---

## 2. Señales — `signal()`

### ¿Qué es?

Una señal es una **celda reactiva** que contiene un valor. Cuando ese valor cambia, Angular actualiza automáticamente solo las partes del template que leen esa señal.

### Firma
```typescript
signal<T>(value: T, options?: SignalOptions<T>): WritableSignal<T>

interface SignalOptions<T> {
  equal?: (prev: T, next: T) => boolean;
}
```

### Métodos
```typescript
const miSignal = signal<string>('hola');

miSignal()                    // Leer el valor actual
miSignal.set('nuevo valor')   // Reemplazar completamente
miSignal.update(prev => prev + '!') // Actualizar basado en anterior
miSignal.asReadonly()         // Versión solo-lectura
```

### Implementación
```typescript
export class MiComponente {
  // Estado básico
  readonly isLoading = signal(false);
  readonly errorMessage = signal<string | null>(null);

  // Estado con objeto
  readonly formData = signal<Partial<User>>({
    name: '',
    email: ''
  });

  // Comparador custom (para objetos)
  readonly usuario = signal<Usuario | null>(null, {
    equal: (prev, next) => prev?.id === next?.id
  });
}
```

### Reglas clave
- Siempre declarar con `readonly`
- En template **siempre con paréntesis**: `isLoading()`, nunca `isLoading`
- Usar `.set()` para valor completo, `.update()` cuando depende del anterior

---

## 3. Señales Derivadas — `computed()`

### ¿Qué es?

`computed()` crea una señal que se recalcula automáticamente cuando **cualquiera de las señales que lee** cambia.

### Firma
```typescript
computed<T>(computation: () => T, options?: SignalOptions<T>): Signal<T>
```

### Implementación
```typescript
export class MiComponente {
  readonly url = signal<Url | null>(null);

  // Derivada de señal escribible
  readonly isEditMode = computed(() => this.url() !== null);

  // Derivada de múltiples señales
  readonly formData = signal({ name: '', id_UrlGrp: 0 });
  readonly resumen = computed(() => {
    const nombre = this.formData().name;
    return `${nombre} → validación`;
  });
}
```

### Reglas clave
- `computed` es **solo lectura** (no tiene `.set()` ni `.update()`)
- No incluir efectos secundarios dentro (HTTP, console.log)
- Es lazy: no se recalcula hasta que alguien lo lee

---

## 4. De Observable a Señal — `toSignal()`

### ¿Qué es?

Convierte un Observable en una señal que Angular rastrea reactivamente.

### Firma completa
```typescript
toSignal<T>(source: Observable<T>, options?: ToSignalOptions<T>): Signal<T>

interface ToSignalOptions<T> {
  initialValue?: T;
  requireSync?: boolean;
  injector?: Injector;
  equal?: (prev: T, next: T) => boolean;
}
```

### Implementación recomendada
```typescript
import { toSignal } from '@angular/core/rxjs-interop';
import { catchError, of } from 'rxjs';

export class MiComponente {
  private readonly userService = inject(UserService);

  private readonly usersSignal = toSignal(
    this.userService.getAll().pipe(
      catchError((err) => {
        console.error('Error:', err);
        return of({ data: [] } as ApiResponse<User[]>);
      })
    ),
    { initialValue: undefined }
  );

  readonly users = computed(() => this.usersSignal()?.data ?? []);
}
```

---

## 5. Inyección con `inject()`

### ¿Qué es?

Obtiene una instancia de servicio sin usar el constructor.

### Firma
```typescript
inject<T>(token: InjectionToken<T> | Type<T>, options?: InjectOptions): T

interface InjectOptions {
  optional?: boolean;
  self?: boolean;
  skipSelf?: boolean;
}
```

### Implementación
```typescript
export class MiComponente {
  // Básico
  private readonly userService = inject(UserService);
  private readonly router = inject(Router);

  // Optional
  private readonly logger = inject(LoggerService, { optional: true });
}
```

### Dónde es válido
```typescript
export class MiComponente {
  // ✅ Válido: zona de inicialización
  private readonly svc = inject(MiServicio);

  // ✅ Válido: constructor
  constructor() {
    const otra = inject(OtroServicio);
  }

  // ❌ Inválido: dentro de métodos asíncronos
  async ngOnInit() {
    const svc = inject(MiServicio); // ERROR
  }
}
```

---

## 6. Control de Flujo en Template (`@if`, `@for`, `@switch`)

### `@if` — Condicional
```html
@if (isLoading()) {
  <app-loading />
} @else if (error()) {
  <app-error [message]="error()!" />
} @else {
  <p>Contenido cargado</p>
}
```

### `@for` — Bucle
```html
<!-- track es OBLIGATORIO -->
@for (user of users(); track user.id) {
  <li>{{ user.name }}</li>
} @empty {
  <li>No hay usuarios</li>
}

<!-- Variables de contexto -->
@for (item of items(); track item.id; let i = $index; let first = $first) {
  {{ i + 1 }}. {{ item.name }} ({{ first ? 'primero' : 'otros' }})
}
```

### `@switch` — Condicional múltiple
```html
@switch (userRole()) {
  @case ('admin') { <admin-dashboard /> }
  @case ('editor') { <editor-dashboard /> }
  @default { <guest-dashboard /> }
}
```

### Reglas clave
- `track` es obligatorio en `@for`
- No necesitan importaciones
- Reemplazan `*ngIf`, `*ngFor`, `[ngSwitch]`

---

## 7. Enrutamiento Moderno — `provideRouter` y `Routes`

### Configuración en app.config.ts
```typescript
import { provideRouter, Routes } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withPreloading(PreloadAllModules))
  ]
};
```

### Definición de rutas
```typescript
// app.routes.ts
export const routes: Routes = [
  // Ruta raíz con redirección
  {
    path: '',
    redirectTo: 'home',
    pathMatch: 'full'
  },

  // Eager loading
  {
    path: 'home',
    component: HomeComponent,
    title: 'Inicio'
  },

  // Lazy loading
  {
    path: 'users',
    loadComponent: () => import('./users/users.component').then(m => m.UsersComponent)
  },

  // Con guard
  {
    path: 'admin',
    canActivate: [authGuard],
    loadComponent: () => import('./admin/admin.component').then(m => m.AdminComponent)
  },

  // Con resolver
  {
    path: 'user/:id',
    loadComponent: () => import('./user/user.component').then(m => m.UserComponent),
    resolve: { user: userResolver }
  },

  // Catch-all (SIEMPRE al final)
  {
    path: '**',
    loadComponent: () => import('./not-found/not-found.component').then(m => m.NotFoundComponent)
  }
];
```

### Título dinámico
```typescript
{
  path: 'user/:id',
  title: (route) => `Usuario ${route.paramMap.get('id')}`
}
```

---

## 8. Guards Funcionales

### Tipos de guards
```typescript
// canActivate: ¿puede entrar?
type CanActivateFn = (route: ActivatedRouteSnapshot, state: RouterStateSnapshot) => MaybeAsync<GuardResult>;

// canDeactivate: ¿puede salir?
type CanDeactivateFn<T> = (component: T, currentRoute: ActivatedRouteSnapshot, currentState: RouterStateSnapshot) => MaybeAsync<GuardResult>;

// resolve: cargar datos antes de activar
type ResolveFn<T> = (route: ActivatedRouteSnapshot, state: RouterStateSnapshot) => MaybeAsync<T>;
```

### Guard de autenticación
```typescript
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }

  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
};
```

### Guard de rol (higher-order function)
```typescript
export const roleGuard = (...roles: string[]): CanActivateFn => {
  return (route, state) => {
    const authService = inject(AuthService);
    const router = inject(Router);

    const hasRole = roles.some(role => authService.hasRole(role));
    return hasRole || router.createUrlTree(['/acceso-denegado']);
  };
};

// Uso: canActivate: [authGuard, roleGuard('admin')]
```

### Resolver
```typescript
export const userResolver: ResolveFn<User> = (route, state) => {
  const service = inject(UserService);
  const id = route.paramMap.get('id')!;
  return service.getById(id);
};
```

---

## 9. Interceptores Funcionales

### Firma
```typescript
type HttpInterceptorFn = (
  req: HttpRequest<unknown>,
  next: HttpHandlerFn
) => Observable<HttpEvent<unknown>>;
```

### Registrar en app.config.ts
```typescript
import { provideHttpClient, withInterceptors } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor, errorInterceptor])
    )
  ]
};
```

### Interceptor de autenticación
```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    const authReq = req.clone({
      headers: req.headers.set('Authorization', `Bearer ${token}`)
    });
    return next(authReq);
  }

  return next(req);
};
```

### Interceptor de errores
```typescript
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      const router = inject(Router);

      switch (error.status) {
        case 401: router.navigate(['/login']); break;
        case 403: router.navigate(['/acceso-denegado']); break;
        case 404: router.navigate(['/no-encontrado']); break;
        case 500: router.navigate(['/error']); break;
      }

      return throwError(() => error);
    })
  );
};
```

---

## 10. `HttpClient` Moderno

### Configuración
```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor, errorInterceptor])
    )
  ]
};
```

### Servicio típico
```typescript
@Injectable({ providedIn: 'root' })
export class ApiService {
  private http = inject(HttpClient);
  private baseUrl = '/api';

  get<T>(endpoint: string): Observable<T> {
    return this.http.get<T>(`${this.baseUrl}/${endpoint}`);
  }

  post<T, B>(endpoint: string, body: B): Observable<T> {
    return this.http.post<T>(`${this.baseUrl}/${endpoint}`, body);
  }

  put<T, B>(endpoint: string, body: B): Observable<T> {
    return this.http.put<T>(`${this.baseUrl}/${endpoint}`, body);
  }

  delete<T>(endpoint: string): Observable<T> {
    return this.http.delete<T>(`${this.baseUrl}/${endpoint}`);
  }
}
```

---

## 11. Formularios: Señales vs Reactive Forms

### Estrategia A: Señales + evento nativo (RECOMENDADO)

```typescript
export class UserFormPage {
  readonly formData = signal<Partial<User>>({
    name: '',
    email: ''
  });

  readonly isLoading = signal(false);
  readonly errorMessage = signal<string | null>(null);

  updateName(value: string): void {
    this.formData.update(d => ({ ...d, name: value }));
  }

  onSubmit(): void {
    if (!this.formData().name?.trim()) {
      this.errorMessage.set('El nombre es obligatorio');
      return;
    }
    // Procesar...
  }
}
```

```html
<input [value]="formData().name" (input)="updateName($any($event.target).value)" />
```

### Estrategia B: Reactive Forms

```typescript
export class UserFormPage {
  form = new FormGroup({
    name: new FormControl('', [Validators.required, Validators.maxLength(50)]),
    email: new FormControl('', [Validators.required, Validators.email])
  });

  onSubmit(): void {
    if (this.form.valid) {
      console.log(this.form.value);
    }
  }
}
```

```html
<form [formGroup]="form" (ngSubmit)="onSubmit()">
  <input formControlName="name" />
</form>
```

### Cuándo usar qué

| Señales | Reactive Forms |
|---------|----------------|
| Validaciones simples | Validaciones complejas |
| Campos fijos | Campos dinámicos |
| Código ligero | Formularios grandes |

---

## 12. Detección de Cambios — `OnPush`

### ¿Qué es?

Solo verifica cuando los inputs cambian de referencia, cuando una señal cambia, o cuando un evento interno se dispara.

### Implementación
```typescript
@Component({
  selector: 'app-user-card',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <h3>{{ user().name }}</h3>
    <p>{{ user().email }}</p>
  `
})
export class UserCardComponent {
  readonly user = input.required<User>();
}
```

### Cuándo Angular actualiza
```
✅ SE actualiza cuando:
  → Un input() recibe valor con referencia diferente
  → Una señal que el template lee cambia
  → Un evento del DOM se dispara desde el componente

❌ NO se actualiza cuando:
  → Un objeto mutable dentro de un input se modifica
  → Un Observable emite sin usar toSignal
```

---

## 13. Estructura de Proyecto

```
src/
├── app/
│   ├── core/
│   │   ├── services/
│   │   │   ├── api.service.ts
│   │   │   └── auth.service.ts
│   │   ├── guards/
│   │   │   └── auth.guard.ts
│   │   ├── interceptors/
│   │   │   └── auth.interceptor.ts
│   │   └── models/
│   │
│   ├── shared/
│   │   ├── components/
│   │   │   ├── loading-spinner/
│   │   │   ├── data-table/
│   │   │   └── confirm-dialog/
│   │   ├── pipes/
│   │   └── directives/
│   │
│   ├── features/
│   │   ├── users/
│   │   │   ├── pages/
│   │   │   │   ├── user-list-page/
│   │   │   │   └── user-form-page/
│   │   │   ├── components/
│   │   │   ├── services/
│   │   │   ├── models/
│   │   │   └── user.routes.ts
│   │   └── products/
│   │
│   ├── layouts/
│   │   ├── main-layout/
│   │   └── auth-layout/
│   │
│   ├── app.component.ts
│   ├── app.routes.ts
│   └── app.config.ts
└── environments/
```

---

## 14. Componentes Standalone

### Estructura
```typescript
@Component({
  selector: 'app-user-card',
  standalone: true, // Por defecto en Angular 16+
  imports: [CommonModule, RouterLink],
  templateUrl: './user-card.html',
  styleUrl: './user-card.scss'
})
export class UserCardComponent {
  // input() y output() (Angular 17+)
  readonly user = input.required<User>();
  readonly userClick = output<User>();

  // Estado
  isExpanded = signal(false);

  // Template usa user() como señal
}
```

### Reglas clave
- Si usas algo en el template, debe estar en `imports`
- Usar `input()` y `output()` en lugar de `@Input()`/`@Output`
- `standalone: true` es el default en Angular 16+

---

## 15. Routing y Navegación

### Leer parámetros de ruta
```typescript
export class UserDetailComponent {
  private route = inject(ActivatedRoute);

  // Con toSignal (recomendado)
  readonly userId = toSignal(
    this.route.paramMap.pipe(map(params => params.get('id')!)),
    { initialValue: 0 }
  );
}
```

### Navegación programática
```typescript
export class MiComponente {
  private router = inject(Router);

  goToUser(id: number): void {
    this.router.navigate(['/users', id]);
  }

  goWithQuery(): void {
    this.router.navigate([], {
      queryParams: { page: 2 },
      queryParamsHandling: 'merge'
    });
  }
}
```

---

## 16. Servicios y HTTP

### Servicio genérico
```typescript
@Injectable({ providedIn: 'root' })
export class ApiService {
  private http = inject(HttpClient);
  private baseUrl = environment.apiUrl;

  get<T>(endpoint: string): Observable<T> {
    return this.http.get<T>(`${this.baseUrl}/${endpoint}`);
  }

  getById<T>(endpoint: string, id: string | number): Observable<T> {
    return this.http.get<T>(`${this.baseUrl}/${endpoint}/${id}`);
  }

  post<T, B>(endpoint: string, body: B): Observable<T> {
    return this.http.post<T>(`${this.baseUrl}/${endpoint}`, body);
  }

  put<T, B>(endpoint: string, id: string | number, body: B): Observable<T> {
    return this.http.put<T>(`${this.baseUrl}/${endpoint}/${id}`, body);
  }

  delete<T>(endpoint: string, id: string | number): Observable<T> {
    return this.http.delete<T>(`${this.baseUrl}/${endpoint}/${id}`);
  }
}
```

### Feature Service
```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  private apiService = inject(ApiService);

  getAll(): Observable<ApiResponse<User[]>> {
    return this.apiService.get<User[]>('users');
  }

  getById(id: number): Observable<ApiResponse<User>> {
    return this.apiService.getById<User>('users', id);
  }

  create(user: CreateUser): Observable<ApiResponse<User>> {
    return this.apiService.post<User, CreateUser>('users', user);
  }

  update(id: number, user: UpdateUser): Observable<ApiResponse<User>> {
    return this.apiService.put<User, UpdateUser>('users', id, user);
  }

  delete(id: number): Observable<ApiResponse<void>> {
    return this.apiService.delete<void>('users', id);
  }
}
```

---

## 17. Mejores Prácticas

### Componentes
- ✅ Usar componentes standalone
- ✅ Usar `input()` y `output()` en lugar de decoradores
- ✅ Usar `computed()` para estado derivado
- ✅ Mantener componentes pequeños
- ✅ Usar `ChangeDetectionStrategy.OnPush`

### Templates
- ✅ Usar control flow nativo (`@if`, `@for`, `@switch`)
- ✅ Usar `track` en todos los `@for`
- ✅ No usar getters en templates

### Estado
- ✅ Usar signals para estado local
- ✅ Usar `update()` o `set()`, NO mutar directamente
- ✅ Usar `computed()` para estado derivado

### Servicios
- ✅ `providedIn: 'root'` para singletons
- ✅ Usar `inject()` en lugar de constructor
- ✅ Retornar Observables

### TypeScript
- ✅ Usar strict type checking
- ✅ Evitar `any`, usar `unknown` si no conoces el tipo
- ✅ Definir interfaces para todos los modelos

### Optimización
- ✅ Lazy loading para rutas
- ✅ Usar `track` en `@for`
- ✅ OnPush change detection

---

## Recursos

- [Angular.dev](https://angular.dev) - Documentación oficial
- [Angular Signals](https://angular.dev/guide/signals)
- [Angular Router](https://angular.dev/guide/routing)

---

*Esta guía cubre Angular 21+ para proyectos sin SSR. Para SSR, consulta skill_angular-21-fullstack.md*