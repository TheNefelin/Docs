# Angular 21 - Skill Completo para Proyectos Nuevos

> Guía completa para desarrollo moderno Angular 21+ con soporte completo CSR/SSR.
> Transversal, sin dependencias de proyectos específicos.

> **⚠️ Importante:** Usar **pnpm** en lugar de npm/yarn. npm tiene vulnerabilidades graves de seguridad y no es seguro para producción.

---

## ¿Por qué pnpm?

| Aspecto | npm | pnpm |
|---------|-----|------|
| **Seguridad** | Vulnerabilidades | Protegido contra dependency confusion |
| **Espacio** | Copias múltiples | Disco virtual compartido |
| **Velocidad** | Lento | 3x más rápido |
| **Estricto** | Permite conflictos | Bloquea conflictos de versiones |

```bash
# Instalar pnpm
npm install -g pnpm

# Crear proyecto
pnpm create @angular/latest my-app

# Comandos
pnpm install    # Instalar dependencias
pnpm dev        # Desarrollo
pnpm build      # Produccion
pnpm add <pkg>  # Agregar paquete
pnpm remove <pkg>  # Eliminar paquete
```

---

## Tabla de Contenidos

1. [Setup y Configuración](#1-setup-y-configuración)
2. [Fundamentos Modernos](#2-fundamentos-modernos)
3. [Estado y Datos Asíncronos](#3-estado-y-datos-asíncronos)
4. [Arquitectura de Componentes](#4-arquitectura-de-componentes)
5. [Servicios y HTTP](#5-servicios-y-http)
6. [Routing](#6-routing)
7. [Formularios](#7-formularios)
8. [CSR vs SSR - Guía Completa](#8-csr-vs-ssr---guía-completa)
9. [Componentes UI Reutilizables](#9-componentes-ui-reutilizables)
10. [Rendimiento](#10-rendimiento)
11. [Testing](#11-testing)
12. [Estructura de Proyecto](#12-estructura-de-proyecto)
13. [Checklist de Código](#13-checklist-de-código)

---

## 1. Setup y Configuración

### 1.1 Crear Proyecto Angular 21

```bash
# Usando Angular CLI
ng new my-app --style=scss --ssr

# Con pnpm (recomendado)
pnpm create @angular/latest my-app --style=scss --ssr

# Versión específica
pnpm create @angular@21 my-app
```

### 1.2 Configuración Manual SSR

```bash
# Agregar SSR a proyecto existente
ng add @angular/ssr
```

### 1.3 Estructura CSR/SSR

```
src/
├── app/
│   ├── app.component.ts
│   ├── app.config.ts
│   ├── app.routes.ts
│   └── main.ts              # CSR entry
├── main.server.ts           # SSR entry
└── server.ts               # Express server (SSR)
```

---

## 2. Fundamentos Modernos

### 2.1 Signals (Angular 16+)

```typescript
import { signal, computed, effect } from "@angular/core";

// Estado básico
const count = signal<number>(0);
const user = signal<User | null>(null);

// Lectura
count(); // getter como función

// Mutaciones
count.set(5);                              // set directo
count.update(v => v + 1);                  // función
items.update(items => [...items, newItem]); // inmutable

// Computed (derivado)
const doubled = computed(() => count() * 2);
const hasItems = computed(() => items().length > 0);

// Effects (side effects)
effect(() => {
  console.log('Count:', count());
});
```

### 2.2 input() y output() (Angular 17+)

```typescript
@Component({
  selector: 'app-user-card',
  imports: [CommonModule],
  template: `
    <div class="card">
      <h3>{{ name() }}</h3>
      <span>{{ role() }}</span>
      <button (click)="select.emit(id())">Select</button>
    </div>
  `
})
export class UserCardComponent {
  // Signal inputs
  readonly id = input.required<string>();
  readonly name = input.required<string>();
  readonly role = input<string>('User');

  // Output
  readonly select = output<string>();

  // Two-way binding (model)
  readonly isSelected = model(false);
}

// Uso: <app-user-card [id]="'123'" [name]="'John'" [(isSelected)]="selected" />
```

### 2.3 viewChild / viewChildren (Angular 17+)

```typescript
@Component({
  selector: 'app-container',
  template: `
    <input #searchInput />
    <app-item *ngFor="let item of items()" />
  `
})
export class ContainerComponent {
  // Signal-based queries
  readonly searchInput = viewChild<ElementRef>('searchInput');
  readonly items = viewChildren(ItemComponent);

  focusSearch() {
    this.searchInput()?.nativeElement.focus();
  }
}
```

### 2.4 rxResource (Angular 19+)

Manejo de observables con signals:

```typescript
// Básico
private readonly userRX = rxResource({
  params: () => this.userId(),
  stream: ({ params }) => this.userService.getById(params)
});

readonly user = computed(() => this.userRX.value() ?? null);
readonly isLoading = computed(() => this.userRX.isLoading());
readonly error = computed(() => this.userRX.error());
```

```typescript
// Con transformación
private readonly usersRX = rxResource({
  params: () => this.pagination(),
  stream: ({ params }) => this.service.getAll(params).pipe(
    map(response => response.data)
  )
});
```

```typescript
// Sin params (ejecuta una vez)
private readonly categoriesRX = rxResource({
  stream: () => this.categoryService.getAll()
});
```

```typescript
// Recargar datos
onRefresh(): void {
  this.userRX.reload();
}
```

### 2.5 toSignal

```typescript
import { toSignal } from '@angular/core/rxjs-interop';

// Básico
readonly users$ = toSignal(this.userService.getAll());

// Con initialValue
readonly users = toSignal(this.userService.getAll(), { initialValue: [] });

// Con transformación
readonly userId = toSignal(
  this.activatedRoute.paramMap.pipe(
    map(params => Number(params.get('id')) || 0)
  ),
  { initialValue: 0 }
);
```

---

## 3. Estado y Datos Asíncronos

### 3.1 Patrón: Page vs Component

**Regla:** Pages manejan datos, Components solo presentan.

```typescript
// ══════════════════════════════════════════════════════
// PAGE - Lógica de negocio, consume servicios, maneja estado
// ══════════════════════════════════════════════════════
@Component({
  selector: 'app-user-list-page',
  imports: [UserListComponent],
  template: `
    <app-user-list
      [userList]="computedList()"
      [isLoading]="isLoading()"
      (onSelect)="handleSelect($event)"
    />
  `
})
export class UserListPage {
  private readonly service = inject(UserService);

  private readonly usersRX = rxResource({
    params: () => this.paginationPayload(),
    stream: ({ params }) => this.service.getAll(params).pipe(
      map(r => r.data)
    )
  });

  readonly computedList = computed(() => this.usersRX.value() ?? []);
  readonly isLoading = computed(() => this.usersRX.isLoading());

  onSelect(user: User): void {
    this.router.navigate(['/user', user.id]);
  }
}

// ══════════════════════════════════════════════════════
// COMPONENT - Solo presentación, sin HTTP
// ══════════════════════════════════════════════════════
@Component({
  selector: 'app-user-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [CommonModule],
  template: `
    @for (user of userList(); track user.id) {
      <tr (click)="onSelect.emit(user)">{{ user.name }}</tr>
    }
  `
})
export class UserListComponent {
  readonly userList = input.required<User[]>();
  readonly isLoading = input<boolean>(false);
  readonly onSelect = output<User>();
}
```

### 3.2 Paginación Completa

```typescript
interface PaginationRequest {
  page: number;
  limit: number;
  search?: string;
  filter?: Record<string, unknown>;
}

export class ProductListPage {
  private readonly service = inject(ProductService);

  readonly currentPage = signal(1);
  readonly limit = signal(10);
  readonly search = signal('');
  readonly totalPages = signal(0);

  readonly paginationPayload = computed<PaginationRequest>(() => ({
    page: this.currentPage(),
    limit: this.limit(),
    search: this.search()
  }));

  private readonly productsRX = rxResource({
    params: () => this.paginationPayload(),
    stream: ({ params }) => this.service.getAll(params).pipe(
      map(r => {
        this.totalPages.set(r.pages);
        return r.data;
      })
    )
  });

  readonly isLoading = computed(() => this.productsRX.isLoading());
  readonly products = computed(() => this.productsRX.value() ?? []);

  nextPage(): void {
    if (this.currentPage() < this.totalPages()) {
      this.currentPage.update(p => p + 1);
    }
  }

  search(text: string): void {
    this.search.set(text);
    this.currentPage.set(1);
  }
}
```

### 3.3 Estados Múltiples

```typescript
export class AdminPage {
  // Estado principal (lectura)
  private readonly itemsRX = rxResource({...});

  // Estado de operaciones (escritura)
  private readonly deleteRX = rxResource({
    params: () => this.deleteId(),
    stream: ({ params: id }) => this.service.delete(id)
  });

  // Loading compuesto
  readonly isLoading = computed(() =>
    this.itemsRX.isLoading() || this.deleteRX.isLoading()
  );

  // Errores
  readonly errorMessage = signal<string | null>(null);

  private handleError(err: unknown): void {
    const message = err instanceof Error
      ? err.message
      : (err as any)?.error?.detail || 'Error inesperado';
    this.errorMessage.set(message);
  }
}
```

### 3.4 rxResource para Submit

```typescript
export class UserFormPage {
  private readonly submitPayload = signal<CreateUserPayload | null>(null);

  private readonly saveRX = rxResource({
    params: () => this.submitPayload(),
    stream: ({ params: payload }) => {
      if (!payload) return of(null);
      return this.userService.create(payload).pipe(
        map(response => {
          if (!response.isSuccess) throw new Error(response.message);
          return response.data;
        }),
        catchError(err => {
          this.handleError(err);
          return of(null);
        })
      );
    }
  });

  protected onFormSubmit(form: User): void {
    this.submitPayload.set(form);
  }

  protected readonly isLoading = computed(() => this.saveRX.isLoading());
}
```

---

## 4. Arquitectura de Componentes

### 4.1 Standalone Components

Angular 17+ no requiere `standalone: true` (es el default):

```typescript
@Component({
  selector: 'app-user-list',
  imports: [CommonModule, RouterLink],
  templateUrl: './user-list.html'
})
export class UserListComponent { }
```

### 4.2 ChangeDetectionStrategy.OnPush

**SIEMPRE** en componentes de presentación:

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  selector: 'app-user-list',
  imports: [...],
  templateUrl: './user-list.html'
})
export class UserListComponent { }
```

### 4.3 Bootstrap CSR

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { AppComponent } from './app/app.component';
import { routes } from './app/app.routes';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes),
    provideHttpClient()
  ]
});
```

### 4.4 Bootstrap SSR

```typescript
// main.server.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { config } from './app/app.config.server';

const bootstrap = () => bootstrapApplication(AppComponent, config);

export default bootstrap;
```

---

## 5. Servicios y HTTP

### 5.1 ApiService Genérico

```typescript
@Injectable({ providedIn: 'root' })
export class ApiService {
  private http = inject(HttpClient);
  private baseUrl = inject(API_BASE_URL);

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

### 5.2 Feature Service

```typescript
@Injectable({ providedIn: 'root' })
export class BookService {
  private apiService = inject(ApiService);
  private endpoint = 'books';

  getAll(params: PaginationRequest): Observable<ApiResponse<PaginatedResult<Book>>> {
    return this.apiService.get(`${this.endpoint}${buildQuery(params)}`);
  }

  getById(id: number): Observable<ApiResponse<Book>> {
    return this.apiService.getById(this.endpoint, id);
  }

  create(book: CreateBook): Observable<ApiResponse<Book>> {
    return this.apiService.post(this.endpoint, book);
  }

  update(id: number, book: UpdateBook): Observable<ApiResponse<Book>> {
    return this.apiService.put(this.endpoint, id, book);
  }

  delete(id: number): Observable<ApiResponse<boolean>> {
    return this.apiService.delete(this.endpoint, id);
  }
}
```

### 5.3 Query Builder

```typescript
function buildQuery(params: Record<string, unknown>): string {
  const query = new URLSearchParams();
  Object.entries(params).forEach(([key, value]) => {
    if (value !== undefined && value !== null && value !== '') {
      query.set(key, String(value));
    }
  });
  return query.toString() ? `?${query}` : '';
}
```

### 5.4 HTTP Interceptors

```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    const token = this.authService.getToken();
    const authReq = token
      ? req.clone({ setHeaders: { Authorization: `Bearer ${token}` } })
      : req;
    return next.handle(authReq);
  }
}

@Injectable()
export class ErrorInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    return next.handle(req).pipe(
      catchError(error => {
        this.errorService.handle(error);
        return throwError(() => error);
      })
    );
  }
}
```

```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(withInterceptors([authInterceptor, errorInterceptor]))
  ]
};
```

---

## 6. Routing

### 6.1 Routes Constants

```typescript
export const ROUTES = {
  HOME: {
    ROOT: '/',
    BOOK: (id: number) => `/book/${id}`
  },
  ADMIN: {
    ROOT: '/admin',
    DASHBOARD: '/admin/dashboard',
    USERS: '/admin/users',
    USER: {
      LIST: '/admin/user',
      FORM: (id?: number) => id ? `/admin/user/form/${id}` : '/admin/user/form'
    }
  },
  AUTH: {
    LOGIN: '/auth/login',
    REGISTER: '/auth/register'
  }
};
```

### 6.2 Routes Definition

```typescript
// app.routes.ts
export const routes: Routes = [
  {
    path: '',
    loadComponent: () => import('./pages/home/home.page').then(m => m.HomePage)
  },
  {
    path: 'admin',
    canActivate: [authGuard],
    children: [
      {
        path: 'users',
        loadComponent: () => import('./pages/admin/users/users.page').then(m => m.UsersPage)
      }
    ]
  },
  {
    path: '**',
    loadComponent: () => import('./pages/not-found/not-found.page').then(m => m.NotFoundPage)
  }
];
```

### 6.3 Functional Guards

```typescript
// auth.guard.ts
export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);

  if (auth.isAuthenticated()) {
    return true;
  }

  return router.createUrlTree(['/auth/login'], {
    queryParams: { returnUrl: state.url }
  });
};

// admin.guard.ts
export const adminGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  const router = inject(Router);

  return auth.hasRole('admin')
    ? true
    : router.createUrlTree(['/']);
};
```

### 6.4 Route Resolvers

```typescript
// user.resolver.ts
export const userResolver: ResolveFn<User> = (route) => {
  const service = inject(UserService);
  const userId = route.paramMap.get('id');
  return service.getById(Number(userId));
};

// En routes
{
  path: 'user/:id',
  resolve: { user: userResolver },
  loadComponent: () => import('./user.page').then(m => m.UserPage)
}

// En componente
export class UserPage {
  private route = inject(ActivatedRoute);
  user = toSignal(this.route.data.pipe(map(d => d['user'])));
}
```

### 6.5 Navigation

```typescript
export class SomePage {
  private router = inject(Router);

  navigateToUser(id: number): void {
    this.router.navigate([ROUTES.ADMIN.USER.FORM(id)]);
  }

  navigateWithQuery(): void {
    this.router.navigate([], {
      queryParams: { page: 2, filter: 'active' },
      queryParamsHandling: 'merge'
    });
  }

  navigateReplace(): void {
    this.router.navigate(['/home'], { replaceUrl: true });
  }
}
```

---

## 7. Formularios

### 7.1 Reactive Forms

```typescript
export class UserFormPage {
  form = new FormGroup({
    name: new FormControl('', [Validators.required, Validators.maxLength(50)]),
    email: new FormControl('', [Validators.required, Validators.email]),
    role: new FormControl('user', [Validators.required])
  });

  onSubmit(): void {
    if (this.form.valid) {
      this.save(this.form.value as User);
    }
  }
}
```

### 7.2 Signal Forms Pattern

Para componentes que reciben datos vía input:

```typescript
@Component({
  selector: 'app-user-form',
  imports: [CommonModule],
  templateUrl: './user-form.html'
})
export class UserFormComponent {
  readonly userModel = input<User | null>(null);
  readonly formSubmit = output<User>();

  readonly formData = signal<Partial<User>>({});

  private readonly syncEffect = effect(() => {
    const user = this.userModel();
    if (user) {
      this.formData.set({ ...user });
    }
  });

  updateField(field: keyof User, value: string): void {
    this.formData.update(data => ({ ...data, [field]: value }));
  }

  onSubmit(event: Event): void {
    event.preventDefault();
    const data = this.formData();
    const error = this.validate(data);
    if (error) {
      this.error.set(error);
      return;
    }
    this.formSubmit.emit(data as User);
  }

  private validate(data: Partial<User>): string | null {
    if (!data.name?.trim()) return 'El nombre es requerido';
    if (!data.email?.trim()) return 'El email es requerido';
    return null;
  }

  readonly error = signal<string | null>(null);
}
```

### 7.3 Custom Validators

```typescript
export function uniqueFieldValidator(
  checkFn: (value: string) => Observable<boolean>
): ValidatorFn {
  return (control: AbstractControl) => {
    if (!control.value) return null;
    return control.valueChanges.pipe(
      debounceTime(300),
      switchMap(value => checkFn(value)),
      map(isUnique => isUnique ? null : { unique: true })
    );
  };
}

// Uso
this.form = new FormGroup({
  email: new FormControl('', [
    Validators.required,
    Validators.email,
    uniqueFieldValidator(email => this.userService.checkEmailExists(email))
  ])
});
```

---

## 8. CSR vs SSR - Guía Completa

### 8.1 Cuándo Usar Qué

| Escenario | Recomendado |
|-----------|-------------|
| App pública con SEO | SSR (Angular Universal) |
| SPA interna (no SEO) | CSR |
| E-commerce | SSR + Hydration |
| Dashboard/admin | CSR |
| Landing pages | SSR con Prerendering |
| App móvil/PWA | CSR + Service Workers |

### 8.2 Configuración CSR (Client-Side Rendering)

```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient()
  ]
};
```

### 8.3 Configuración SSR (Server-Side Rendering)

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideClientHydration, withEventReplay } from '@angular/platform-browser';
import { provideServerRendering } from '@angular/platform-server';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
    provideServerRendering(),
    provideClientHydration(withEventReplay())
  ]
};
```

```typescript
// app.config.server.ts
import { mergeApplicationConfig } from '@angular/platform-server';
import { appConfig } from './app.config';

const serverConfig = {
  providers: [
    provideServerRendering()
  ]
};

export const config = mergeApplicationConfig(appConfig, serverConfig);
```

### 8.4 Hydration Modes

```typescript
import {
  provideClientHydration,
  withEventReplay,
  withIncrementalHydration
} from '@angular/platform-browser';

// Básico
provideClientHydration()

// Con event replay (reproduce eventos del servidor)
provideClientHydration(withEventReplay())

// Hyd incremental (Angular 20+)
provideClientHydration(withIncrementalHydration())
```

### 8.5 Defer Blocks (Lazy Loading de Componentes)

```typescript
@Component({
  selector: 'app-page',
  imports: [HeavyChartComponent, CommentsComponent],
  template: `
    <!-- Carga inmediata -->
    <app-header />

    <!-- Defer hasta visible -->
    @defer (on viewport) {
      <app-heavy-chart />
    } @placeholder {
      <div class="skeleton-chart" />
    } @loading {
      <app-spinner />
    } @error {
      <p>Error al cargar</p>
    }

    <!-- Defer hasta interacción -->
    @defer (on interaction) {
      <app-comments [postId]="postId()" />
    } @placeholder {
      <button>Mostrar comentarios</button>
    }

    <!-- Defer en idle -->
    @defer (on idle) {
      <app-analytics />
    }
  `
})
export class PageComponent {}
```

**Triggers disponibles:**

| Trigger | Descripción |
|---------|-------------|
| `on viewport` | Cuando el elemento entra en viewport |
| `on interaction` | En primer click/hover del elemento |
| `on hover` | En hover del elemento |
| `on idle` | Cuando el browser está idle |
| `on timer(ms)` | Después de X milisegundos |
| `on immediate` | Inmediatamente después de hydration |

### 8.6 TransferState (Evitar Request Duplicados)

```typescript
import { TransferState, makeStateKey } from '@angular/platform-browser';

const USERS_KEY = makeStateKey<User[]>('USERS');

@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);
  private transferState = inject(TransferState);

  getAll(): Observable<User[]> {
    // Verificar si ya existe en server state
    if (this.transferState.hasKey(USERS_KEY)) {
      const users = this.transferState.get(USERS_KEY, []);
      this.transferState.remove(USERS_KEY);
      return of(users);
    }

    return this.http.get<User[]>('/api/users').pipe(
      tap(users => this.transferState.set(USERS_KEY, users))
    );
  }
}
```

### 8.7 SSR - HttpClient con fetch

```typescript
// app.config.ts
import { provideHttpClient, withFetch } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(withFetch())
  ]
};
```

### 8.8 Prerendering (Static Site Generation)

```typescript
// routes.txt (angular.json)
{
  "prerender": {
    "routes": [
      "/",
      "/about",
      "/products"
    ],
    "discoverRoutes": true
  }
}
```

```typescript
// dynamic routes
export const routes: Routes = [
  {
    path: 'product/:id',
    loadComponent: () => import('./product.page').then(m => m.ProductPage)
  }
];

// angular.json
{
  "prerender": {
    "routesFile": "routes.txt"
  }
}
```

---

## 9. Componentes UI Reutilizables

### 9.1 Select Component

```typescript
@Component({
  selector: 'app-select',
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [CommonModule],
  template: `
    <select
      [disabled]="disabled()"
      (change)="onChange($event)"
      class="form-select"
    >
      <option value="">{{ placeholder() }}</option>
      @for (item of items(); track item.id) {
        <option [value]="item.id" [selected]="item.id === selectedId()">
          {{ item.label }}
        </option>
      }
    </select>
  `
})
export class SelectComponent {
  readonly disabled = input<boolean>(false);
  readonly items = input.required<SelectItem[]>();
  readonly selectedId = input<number>(0);
  readonly placeholder = input<string>('Seleccionar...');

  readonly selectionChange = output<SelectItem>();

  onChange(event: Event): void {
    const select = event.target as HTMLSelectElement;
    const id = Number(select.value);
    const item = this.items().find(i => i.id === id);
    if (item) {
      this.selectionChange.emit(item);
    }
  }
}

export interface SelectItem {
  id: number;
  label: string;
}
```

### 9.2 Loading Spinner

```typescript
@Component({
  selector: 'app-loading',
  standalone: true,
  template: `<span class="loading loading-spinner loading-md"></span>`
})
export class LoadingComponent {}
```

### 9.3 Error Message

```typescript
@Component({
  selector: 'app-error-message',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    @if (message()) {
      <div class="alert alert-error">
        <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        <span>{{ message() }}</span>
      </div>
    }
  `
})
export class ErrorMessageComponent {
  readonly message = input<string | null>(null);
}
```

### 9.4 Modal

```typescript
@Component({
  selector: 'app-modal',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    @if (isOpen()) {
      <dialog class="modal modal-open">
        <div class="modal-box">
          <h3 class="font-bold text-lg">{{ title() }}</h3>
          <ng-content />
          <div class="modal-action">
            <button class="btn btn-ghost" (click)="cancel.emit()">Cancelar</button>
            <button class="btn btn-primary" (click)="confirm.emit()">Confirmar</button>
          </div>
        </div>
        <form method="dialog" class="modal-backdrop">
          <button (click)="cancel.emit()">close</button>
        </form>
      </dialog>
    }
  `
})
export class ModalComponent {
  readonly isOpen = input<boolean>(false);
  readonly title = input<string>('');

  readonly cancel = output<void>();
  readonly confirm = output<void>();
}
```

### 9.5 Pagination

```typescript
@Component({
  selector: 'app-pagination',
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [CommonModule],
  template: `
    @if (totalPages() > 1) {
      <div class="join">
        <button
          class="join-item btn"
          [disabled]="currentPage() === 1"
          (click)="prev.emit()"
        >«</button>

        @for (page of visiblePages(); track page) {
          <button
            class="join-item btn"
            [class.btn-active]="page === currentPage()"
            (click)="goToPage.emit(page)"
          >{{ page }}</button>
        }

        <button
          class="join-item btn"
          [disabled]="currentPage() === totalPages()"
          (click)="next.emit()"
        >»</button>
      </div>
    }
  `
})
export class PaginationComponent {
  readonly currentPage = input.required<number>();
  readonly totalPages = input.required<number>();

  readonly prev = output<void>();
  readonly next = output<void>();
  readonly goToPage = output<number>();

  readonly visiblePages = computed(() => {
    const current = this.currentPage();
    const total = this.totalPages();
    const pages: number[] = [];

    for (let i = Math.max(1, current - 2); i <= Math.min(total, current + 2); i++) {
      pages.push(i);
    }
    return pages;
  });
}
```

### 9.6 Table List

```typescript
@Component({
  selector: 'app-table-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [CommonModule],
  template: `
    @if (isLoading()) {
      <app-loading />
    } @else if (items().length === 0) {
      <div class="text-center py-8 text-gray-500">No hay datos</div>
    } @else {
      <table class="table table-zebra">
        <thead>
          <tr>
            <th>ID</th>
            <th>Nombre</th>
            <th>Estado</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          @for (item of items(); track item.id) {
            <tr>
              <td>{{ item.id }}</td>
              <td>{{ item.name }}</td>
              <td>
                <span class="badge" [class.badge-success]="item.active">
                  {{ item.active ? 'Activo' : 'Inactivo' }}
                </span>
              </td>
              <td>
                <button class="btn btn-sm" (click)="edit.emit(item)">Editar</button>
              </td>
            </tr>
          }
        </tbody>
      </table>
    }
  `
})
export class TableListComponent<T extends { id: number }> {
  readonly items = input.required<T[]>();
  readonly isLoading = input<boolean>(false);

  readonly edit = output<T>();
  readonly delete = output<T>();
}
```

---

## 10. Rendimiento

### 10.1 OnPush - Obligatorio

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  ...
})
export class MyComponent {}
```

### 10.2 track en @for

```typescript
@for (item of items(); track item.id) { ... }

@for (item of items(); track item.id + item.name) {
  // Para IDs no únicas
}
```

### 10.3 Evitar Getters en Templates

```typescript
// ❌ MAL
get total(): number {
  return this.items.reduce((sum, i) => sum + i.price, 0);
}

// ✅ BIEN
readonly total = computed(() =>
  this.items().reduce((sum, i) => sum + i.price, 0)
);
```

### 10.4 Evitar Computed Costosos

```typescript
// ❌ MAL - sort en computed
readonly sorted = computed(() =>
  this.items().sort((a, b) => a.name.localeCompare(b.name))
);

// ✅ BIEN - solo derivaciones simples
readonly total = computed(() => this.items().length);
readonly hasItems = computed(() => this.items().length > 0);
```

### 10.5 NgOptimizedImage

```typescript
@Component({
  imports: [NgOptimizedImage],
  template: `
    <!-- Prioritaria (LCP) -->
    <img
      ngSrc="hero.jpg"
      width="800"
      height="600"
      priority
    />

    <!-- Lazy -->
    <img
      ngSrc="thumbnail.jpg"
      width="200"
      height="150"
      loading="lazy"
      placeholder="data:image/..."
    />
  `
})
export class ImageComponent {}
```

---

## 11. Testing

### 11.1 Test Component con Signals

```typescript
describe('UserCardComponent', () => {
  let component: UserCardComponent;
  let fixture: ComponentFixture<UserCardComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [UserCardComponent]
    }).compileComponents();

    fixture = TestBed.createComponent(UserCardComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should display user name', () => {
    component.name.set('John');
    fixture.detectChanges();

    const el = fixture.nativeElement.querySelector('h3');
    expect(el.textContent).toContain('John');
  });

  it('should emit on select', () => {
    const emitSpy = jest.spyOn(component.select, 'emit');

    component.select.emit('123');

    expect(emitSpy).toHaveBeenCalledWith('123');
  });
});
```

### 11.2 Test Component con input()

```typescript
describe('UserListComponent', () => {
  let component: UserListComponent;
  let fixture: ComponentFixture<UserListComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [UserListComponent]
    }).compileComponents();

    fixture = TestBed.createComponent(UserListComponent);
    component = fixture.componentInstance;
    componentRef = fixture.componentRef;

    componentRef.setInput('userList', mockUsers);
    componentRef.setInput('isLoading', false);
    fixture.detectChanges();
  });

  it('should render users', () => {
    const rows = fixture.nativeElement.querySelectorAll('tr');
    expect(rows.length).toBe(mockUsers.length);
  });
});
```

### 11.3 Test Service con rxResource

```typescript
describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [UserService]
    });
    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  it('should get users', () => {
    const mockResponse = { isSuccess: true, data: [{ id: 1, name: 'John' }] };

    service.getAll().subscribe(response => {
      expect(response.data.length).toBe(1);
    });

    const req = httpMock.expectOne('/api/users');
    req.flush(mockResponse);
  });
});
```

---

## 12. Estructura de Proyecto

```
src/
├── app/
│   ├── core/                    # Singleton services, guards, interceptors
│   │   ├── guards/
│   │   ├── interceptors/
│   │   └── services/
│   │       ├── api.service.ts
│   │       └── auth.service.ts
│   ├── features/                # Módulos por dominio
│   │   ├── auth/
│   │   │   ├── components/
│   │   │   ├── pages/
│   │   │   ├── services/
│   │   │   └── auth.routes.ts
│   │   ├── users/
│   │   ├── products/
│   │   └── dashboard/
│   ├── shared/                  # Componentes, directives, pipes reutilizables
│   │   ├── components/
│   │   │   ├── loading/
│   │   │   ├── modal/
│   │   │   ├── select/
│   │   │   └── pagination/
│   │   ├── directives/
│   │   ├── pipes/
│   │   └── models/
│   ├── layouts/                 # Layout components
│   │   ├── main-layout/
│   │   └── auth-layout/
│   ├── app.component.ts
│   ├── app.config.ts
│   ├── app.config.server.ts     # SSR config
│   └── app.routes.ts
├── environments/
│   ├── environment.ts
│   └── environment.prod.ts
└── main.ts                      # CSR entry
├── main.server.ts              # SSR entry
└── server.ts                   # Express server
```

### tsconfig paths

```json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@core/*": ["src/app/core/*"],
      "@features/*": ["src/app/features/*"],
      "@shared/*": ["src/app/shared/*"],
      "@env/*": ["src/environments/*"]
    }
  }
}
```

---

## 13. Checklist de Código

### Componentes
- [ ] `ChangeDetectionStrategy.OnPush` presente
- [ ] Solo usa `input()` y `output()`, no @Input/@Output
- [ ] Template no tiene lógica de negocio
- [ ] `track` en todos los `@for`
- [ ] No usa getters en template

### Pages
- [ ] Separa datos (rxResource) de presentación (input/output)
- [ ] Manejo de errores presente
- [ ] Loading state expuesto
- [ ] No hace mutaciones directas

### Servicios
- [ ] Tipado completo de retorno
- [ ] Usa ApiService centralizado
- [ ] Endpoint como constante

### Routing
- [ ] Usa ROUTES constants
- [ ] Params en URL para datos persistentes

### rxResource
- [ ] `??` con valor por defecto
- [ ] `catchError` retorna valor
- [ ] `.reload()` para refresh

### SSR
- [ ] `provideClientHydration()` configurado
- [ ] `withFetch()` para HttpClient
- [ ] `withEventReplay()` para event hydration
- [ ] Imágenes con `priority` las críticas (LCP)
- [ ] `@defer` para componentes pesados

### General
- [ ] No `any` sin razón
- [ ] No `console.log` en producción
- [ ] No código de debug

---

## Recursos

- [Angular.dev](https://angular.dev)
- [Angular SSR Guide](https://angular.dev/guide/ssr)
- [Angular Signals API](https://angular.dev/guide/signals)
- [Angular Signals Documentation](https://angular.io/guide/signals)
- [Standalone Components](https://angular.dev/guide/standalone-components)
- [Hydration Guide](https://angular.dev/guide/hydration)

---

*Angular 21+ Modern Development Skill*
*Versión: 1.0*