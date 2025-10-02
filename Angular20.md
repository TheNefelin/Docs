# Guía Completa Angular 19/20+ (Zoneless, Sin SSR)

## Índice

1. [Conceptos Fundamentales](#1-conceptos-fundamentales)
2. [Estructura de Proyecto](#2-estructura-de-proyecto)
3. [Signals - Manejo de Estado](#3-signals---manejo-de-estado)
4. [Servicios y Consumo de APIs](#4-servicios-y-consumo-de-apis)
5. [Control Flow Nativo](#5-control-flow-nativo)
6. [Componentes](#6-componentes)
7. [Formularios Reactivos](#7-formularios-reactivos)
8. [Routing y Navegación](#8-routing-y-navegación)
9. [Angular Material](#9-angular-material)
10. [Ejemplo CRUD Completo](#10-ejemplo-crud-completo)
11. [Mejores Prácticas](#11-mejores-prácticas)

---

## 1. Conceptos Fundamentales

### 1.1 ¿Qué es Angular 19/20+?

Angular 19/20+ es la versión moderna del framework que introduce:
- **Standalone components** (sin NgModules)
- **Signals** para manejo de estado reactivo
- **Control flow nativo** (`@if`, `@for`, `@switch`)
- **Zoneless mode** (mejor rendimiento)
- **inject()** en lugar de constructor injection

### 1.2 Configuración Inicial

```bash
# Crear proyecto sin SSR
ng new mi-app --no-ssr --routing --style=scss

# Instalar Angular Material
ng add @angular/material

# Estructura básica
cd mi-app
npm start
```

---

## 2. Estructura de Proyecto

### 2.1 Estructura Recomendada para Escalabilidad

```
src/
├── app/
│   ├── core/                    # Servicios singleton (una instancia)
│   │   ├── services/
│   │   │   ├── crud.service.ts
│   │   │   ├── auth.service.ts
│   │   │   └── api.service.ts
│   │   ├── guards/
│   │   │   └── auth.guard.ts
│   │   ├── interceptors/
│   │   │   └── auth.interceptor.ts
│   │   └── models/              # Interfaces/types globales
│   │
│   ├── shared/                  # Reutilizable en toda la app
│   │   ├── components/
│   │   │   ├── confirm-dialog/
│   │   │   ├── loading-spinner/
│   │   │   └── data-table/
│   │   ├── pipes/
│   │   ├── directives/
│   │   └── utils/
│   │
│   ├── features/                # Un folder por dominio de negocio
│   │   ├── auth/                # Login, register, etc.
│   │   │   ├── pages/
│   │   │   ├── components/
│   │   │   ├── services/
│   │   │   └── auth.routes.ts
│   │   │
│   │   ├── users/               # CRUD usuarios
│   │   │   ├── pages/
│   │   │   │   ├── user-list-page/
│   │   │   │   ├── user-detail-page/
│   │   │   │   └── user-create-page/
│   │   │   ├── components/      # Solo para users
│   │   │   │   ├── user-card/
│   │   │   │   └── user-form/
│   │   │   ├── services/
│   │   │   │   └── user.service.ts
│   │   │   ├── models/
│   │   │   │   └── user.model.ts
│   │   │   └── user.routes.ts
│   │   │
│   │   ├── products/            # CRUD productos
│   │   ├── dashboard/           # Dashboard + widgets
│   │   └── public/              # Home, About, Contact (sin auth)
│   │       ├── pages/
│   │       │   ├── home-page/
│   │       │   ├── about-page/
│   │       │   └── contact-page/
│   │       └── public.routes.ts
│   │
│   ├── layout/                  # Estructura visual
│   │   ├── main-layout/
│   │   ├── auth-layout/
│   │   └── components/
│   │       ├── sidebar/
│   │       ├── header/
│   │       └── footer/
│   │
│   ├── app.component.ts
│   ├── app.routes.ts           # Rutas principales
│   └── app.config.ts
```

### 2.2 Reglas de Organización

| Carpeta | Propósito | Ejemplo |
|---------|-----------|---------|
| **core/** | Servicios singleton globales | AuthService, ApiService |
| **shared/** | Componentes usados en 2+ features | LoadingSpinner, ConfirmDialog |
| **features/** | Dominios de negocio independientes | users/, products/, dashboard/ |
| **layout/** | Solo estructura visual | Sidebar, Header, Footer |
| **pages/** | Componentes con ruta (páginas) | UserListPage, ProductDetailPage |
| **components/** | Piezas reutilizables sin ruta | UserCard, ProductForm |

---

## 3. Signals - Manejo de Estado

### 3.1 ¿Qué son los Signals?

Los **Signals** son la nueva forma de manejar estado reactivo en Angular. Reemplazan muchos casos de uso de RxJS para estado local.

### 3.2 Tipos de Signals

#### Signal Básico
```typescript
import { signal } from '@angular/core';

export class UserComponent {
  // Signal simple
  count = signal(0);
  user = signal<User | null>(null);
  users = signal<User[]>([]);
  
  // Leer valor
  ngOnInit() {
    console.log(this.count()); // 0
  }
  
  // Actualizar valor
  increment() {
    this.count.set(5);           // Establece valor directo
    this.count.update(n => n + 1); // Basado en valor anterior
  }
}
```

#### Computed Signals (Estado Derivado)
```typescript
import { computed } from '@angular/core';

export class UserComponent {
  users = signal<User[]>([]);
  searchTerm = signal('');
  
  // Se recalcula automáticamente cuando users o searchTerm cambian
  filteredUsers = computed(() => 
    this.users().filter(u => 
      u.name.toLowerCase().includes(this.searchTerm().toLowerCase())
    )
  );
  
  // Múltiples dependencias
  activeUsers = computed(() => 
    this.users().filter(u => u.active)
  );
  
  userCount = computed(() => this.users().length);
}
```

#### Effect (Reaccionar a cambios)
```typescript
import { effect } from '@angular/core';

export class UserComponent {
  count = signal(0);
  
  constructor() {
    // Se ejecuta cada vez que count cambia
    effect(() => {
      console.log('Count cambió a:', this.count());
      
      // Efectos secundarios (localStorage, logs, etc)
      localStorage.setItem('count', this.count().toString());
    });
  }
}
```

### 3.3 Signals en Templates

```typescript
@Component({
  template: `
    <!-- Leer signal directamente -->
    <p>Count: {{ count() }}</p>
    <p>Users: {{ users().length }}</p>
    
    <!-- Computed signals -->
    <p>Active users: {{ activeUsers().length }}</p>
    
    <!-- Iterar sobre signal -->
    @for (user of users(); track user.id) {
      <div>{{ user.name }}</div>
    }
    
    <!-- Condicional con signal -->
    @if (loading()) {
      <p>Cargando...</p>
    }
  `
})
export class MyComponent {
  count = signal(0);
  users = signal<User[]>([]);
  loading = signal(false);
  
  activeUsers = computed(() => 
    this.users().filter(u => u.active)
  );
}
```

### 3.4 Cuándo usar Signals vs RxJS

| Usar Signals | Usar RxJS (Observables) |
|--------------|-------------------------|
| Estado local del componente | Llamadas HTTP |
| Estado derivado (computed) | WebSockets / Server-Sent Events |
| Valores síncronos | Operaciones asíncronas complejas |
| UI reactiva simple | Streams de eventos complejos |

---

## 4. Servicios y Consumo de APIs

### 4.1 Patrón de Servicios

#### Opción A: Servicio por Entidad (Recomendado)

```typescript
// src/app/features/users/services/user.service.ts
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { User } from '../models/user.model';

@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);
  private readonly baseUrl = '/api/users';
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.baseUrl);
  }
  
  getUserById(id: number): Observable<User> {
    return this.http.get<User>(`${this.baseUrl}/${id}`);
  }
  
  createUser(user: CreateUserDto): Observable<User> {
    return this.http.post<User>(this.baseUrl, user);
  }
  
  updateUser(id: number, user: UpdateUserDto): Observable<User> {
    return this.http.put<User>(`${this.baseUrl}/${id}`, user);
  }
  
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.baseUrl}/${id}`);
  }
  
  // Métodos específicos del dominio
  promoteToAdmin(userId: number): Observable<User> {
    return this.http.patch<User>(`${this.baseUrl}/${userId}/promote`, {});
  }
}
```

#### Opción B: CRUD Genérico (Para proyectos grandes)

```typescript
// src/app/core/services/crud.service.ts
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable()
export class CrudService<T> {
  private http = inject(HttpClient);
  
  constructor(private endpoint: string) {}
  
  getAll(): Observable<T[]> {
    return this.http.get<T[]>(this.endpoint);
  }
  
  getById(id: string | number): Observable<T> {
    return this.http.get<T>(`${this.endpoint}/${id}`);
  }
  
  create(item: Partial<T>): Observable<T> {
    return this.http.post<T>(this.endpoint, item);
  }
  
  update(id: string | number, item: Partial<T>): Observable<T> {
    return this.http.put<T>(`${this.endpoint}/${id}`, item);
  }
  
  delete(id: string | number): Observable<void> {
    return this.http.delete<void>(`${this.endpoint}/${id}`);
  }
}

// Servicio específico que usa el genérico
@Injectable({ providedIn: 'root' })
export class UserService {
  private crudService = new CrudService<User>('/api/users');
  
  getUsers() {
    return this.crudService.getAll();
  }
  
  createUser(user: CreateUserDto) {
    return this.crudService.create(user);
  }
  
  // Métodos específicos
  getActiveUsers(): Observable<User[]> {
    return this.crudService.getAll().pipe(
      map(users => users.filter(u => u.active))
    );
  }
}
```

### 4.2 Consumir APIs en Componentes

```typescript
export class UsersComponent implements OnInit {
  private userService = inject(UserService);
  
  users = signal<User[]>([]);
  loading = signal(false);
  error = signal<string | null>(null);
  
  ngOnInit() {
    this.loadUsers();
  }
  
  loadUsers() {
    this.loading.set(true);
    this.error.set(null);
    
    this.userService.getUsers().subscribe({
      next: (users) => {
        this.users.set(users);
        this.loading.set(false);
      },
      error: (error) => {
        this.error.set('Error al cargar usuarios');
        this.loading.set(false);
        console.error(error);
      }
    });
  }
  
  createUser(userData: CreateUserDto) {
    this.userService.createUser(userData).subscribe({
      next: (newUser) => {
        // Agregar a la lista local
        this.users.update(users => [...users, newUser]);
      },
      error: (error) => {
        console.error('Error creating user:', error);
      }
    });
  }
  
  deleteUser(id: number) {
    this.userService.deleteUser(id).subscribe({
      next: () => {
        // Remover de la lista local
        this.users.update(users => users.filter(u => u.id !== id));
      },
      error: (error) => {
        console.error('Error deleting user:', error);
      }
    });
  }
}
```

---

## 5. Control Flow Nativo

### 5.1 @if (Condicionales)

#### Sintaxis Nueva
```typescript
@Component({
  template: `
    <!-- If simple -->
    @if (isLoggedIn()) {
      <p>Bienvenido usuario</p>
    }
    
    <!-- If / Else -->
    @if (loading()) {
      <mat-spinner></mat-spinner>
    } @else {
      <div>Contenido cargado</div>
    }
    
    <!-- If / Else If / Else -->
    @if (status() === 'loading') {
      <p>Cargando...</p>
    } @else if (status() === 'error') {
      <p>Error al cargar</p>
    } @else {
      <p>Datos cargados correctamente</p>
    }
    
    <!-- If con signal anidado -->
    @if (user()) {
      <p>Hola {{ user()!.name }}</p>
    } @else {
      <p>No hay usuario</p>
    }
  `
})
export class MyComponent {
  isLoggedIn = signal(true);
  loading = signal(false);
  status = signal<'loading' | 'error' | 'success'>('loading');
  user = signal<User | null>(null);
}
```

#### Sintaxis Antigua (NO usar)
```html
<!-- Antigua: *ngIf -->
<p *ngIf="isLoggedIn()">Bienvenido usuario</p>
<div *ngIf="loading(); else content">Cargando...</div>
<ng-template #content>Contenido</ng-template>
```

### 5.2 @for (Loops)

#### Sintaxis Nueva
```typescript
@Component({
  template: `
    <!-- For básico con track -->
    @for (user of users(); track user.id) {
      <div>{{ user.name }}</div>
    }
    
    <!-- For con index -->
    @for (user of users(); track user.id; let i = $index) {
      <div>{{ i + 1 }}. {{ user.name }}</div>
    }
    
    <!-- For con first/last/even/odd -->
    @for (user of users(); track user.id; let first = $first; let last = $last) {
      <div [class.first]="first" [class.last]="last">
        {{ user.name }}
      </div>
    }
    
    <!-- For con empty -->
    @for (user of users(); track user.id) {
      <div>{{ user.name }}</div>
    } @empty {
      <p>No hay usuarios disponibles</p>
    }
    
    <!-- For anidado -->
    @for (category of categories(); track category.id) {
      <h3>{{ category.name }}</h3>
      @for (product of category.products; track product.id) {
        <p>{{ product.name }}</p>
      }
    }
  `
})
export class MyComponent {
  users = signal<User[]>([]);
  categories = signal<Category[]>([]);
}
```

#### Sintaxis Antigua (NO usar)
```html
<!-- Antigua: *ngFor -->
<div *ngFor="let user of users(); trackBy: trackById">
  {{ user.name }}
</div>
```

### 5.3 @switch (Switch Cases)

#### Sintaxis Nueva
```typescript
@Component({
  template: `
    @switch (userRole()) {
      @case ('admin') {
        <admin-dashboard />
      }
      @case ('editor') {
        <editor-dashboard />
      }
      @case ('viewer') {
        <viewer-dashboard />
      }
      @default {
        <guest-dashboard />
      }
    }
  `
})
export class DashboardComponent {
  userRole = signal<'admin' | 'editor' | 'viewer' | 'guest'>('guest');
}
```

#### Sintaxis Antigua (NO usar)
```html
<!-- Antigua: *ngSwitch -->
<div [ngSwitch]="userRole()">
  <admin-dashboard *ngSwitchCase="'admin'"></admin-dashboard>
  <editor-dashboard *ngSwitchCase="'editor'"></editor-dashboard>
  <guest-dashboard *ngSwitchDefault></guest-dashboard>
</div>
```

---

## 6. Componentes

### 6.1 Estructura de un Componente Standalone

```typescript
import { Component, signal, computed, input, output } from '@angular/core';
import { CommonModule } from '@angular/common';
import { MatButtonModule } from '@angular/material/button';

@Component({
  selector: 'app-user-card',
  standalone: true, // Por defecto en Angular 19+
  imports: [
    CommonModule,
    MatButtonModule
  ],
  template: `
    <div class="user-card">
      <h3>{{ user().name }}</h3>
      <p>{{ user().email }}</p>
      
      <!-- Computed signal -->
      <p>Status: {{ userStatus() }}</p>
      
      <!-- Emitir evento -->
      <button mat-raised-button (click)="handleClick()">
        Ver detalles
      </button>
    </div>
  `,
  styles: [`
    .user-card {
      padding: 16px;
      border: 1px solid #ddd;
      border-radius: 8px;
    }
  `]
})
export class UserCardComponent {
  // Input (props)
  user = input.required<User>(); // Input obligatorio
  showEmail = input(true);       // Input opcional con default
  
  // Output (eventos)
  userClick = output<User>();
  
  // Estado local
  isExpanded = signal(false);
  
  // Computed
  userStatus = computed(() => 
    this.user().active ? 'Activo' : 'Inactivo'
  );
  
  handleClick() {
    this.userClick.emit(this.user());
  }
}
```

### 6.2 Uso de Componentes

```typescript
@Component({
  template: `
    <!-- Usar componente hijo -->
    @for (user of users(); track user.id) {
      <app-user-card 
        [user]="user"
        [showEmail]="true"
        (userClick)="onUserClick($event)"
      />
    }
  `
})
export class UserListComponent {
  users = signal<User[]>([]);
  
  onUserClick(user: User) {
    console.log('Clicked user:', user);
  }
}
```

### 6.3 Ciclo de Vida

```typescript
import { Component, OnInit, OnDestroy, AfterViewInit } from '@angular/core';

export class MyComponent implements OnInit, OnDestroy, AfterViewInit {
  
  // Cuando el componente se inicializa
  ngOnInit() {
    console.log('Component initialized');
    this.loadData();
  }
  
  // Después de renderizar la vista
  ngAfterViewInit() {
    console.log('View initialized');
  }
  
  // Antes de destruir el componente
  ngOnDestroy() {
    console.log('Component destroyed');
    // Limpiar subscripciones, timers, etc
  }
}
```

---

## 7. Formularios Reactivos

### 7.1 Setup Básico

```typescript
import { Component, inject } from '@angular/core';
import { FormBuilder, FormGroup, Validators, ReactiveFormsModule } from '@angular/forms';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatButtonModule } from '@angular/material/button';

@Component({
  selector: 'app-user-form',
  standalone: true,
  imports: [
    ReactiveFormsModule,
    MatFormFieldModule,
    MatInputModule,
    MatButtonModule
  ],
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      
      <!-- Input de texto -->
      <mat-form-field appearance="outline">
        <mat-label>Nombre</mat-label>
        <input matInput formControlName="name" placeholder="Juan Pérez">
        @if (userForm.get('name')?.invalid && userForm.get('name')?.touched) {
          <mat-error>El nombre es requerido</mat-error>
        }
      </mat-form-field>

      <!-- Email con validación -->
      <mat-form-field appearance="outline">
        <mat-label>Email</mat-label>
        <input matInput formControlName="email" type="email">
        @if (userForm.get('email')?.invalid && userForm.get('email')?.touched) {
          <mat-error>
            @if (userForm.get('email')?.errors?.['required']) {
              El email es requerido
            }
            @if (userForm.get('email')?.errors?.['email']) {
              Email inválido
            }
          </mat-error>
        }
      </mat-form-field>

      <!-- Submit -->
      <button mat-raised-button color="primary" type="submit" [disabled]="userForm.invalid">
        Guardar
      </button>
    </form>
  `
})
export class UserFormComponent {
  private fb = inject(FormBuilder);
  
  userForm: FormGroup;
  
  constructor() {
    this.userForm = this.fb.group({
      name: ['', [Validators.required, Validators.minLength(3)]],
      email: ['', [Validators.required, Validators.email]],
      age: [18, [Validators.min(18), Validators.max(100)]],
      active: [true]
    });
  }
  
  onSubmit() {
    if (this.userForm.valid) {
      const formValue = this.userForm.value;
      console.log('Form submitted:', formValue);
      
      // Llamar al servicio
      // this.userService.createUser(formValue).subscribe(...)
    }
  }
  
  // Precargar datos (modo edición)
  loadUser(user: User) {
    this.userForm.patchValue({
      name: user.name,
      email: user.email,
      age: user.age,
      active: user.active
    });
  }
}
```

### 7.2 Validaciones Personalizadas

```typescript
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

// Validador personalizado
export function urlValidator(): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const urlPattern = /^https?:\/\/.+/;
    const valid = urlPattern.test(control.value);
    return valid ? null : { invalidUrl: true };
  };
}

// Usar en el form
this.urlForm = this.fb.group({
  link: ['', [Validators.required, urlValidator()]]
});
```

---

## 8. Routing y Navegación

### 8.1 Configuración de Rutas

```typescript
// src/app/app.routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    redirectTo: '/home',
    pathMatch: 'full'
  },
  {
    path: 'home',
    component: HomeComponent
  },
  {
    path: 'about',
    component: AboutComponent
  },
  // Lazy loading (recomendado)
  {
    path: 'users',
    loadChildren: () => import('./features/users/user.routes').then(r => r.urlRoutes)
  },
  // Ruta con parámetro
  {
    path: 'users/:id',
    component: UserDetailComponent
  },
  // 404
  {
    path: '**',
    component: NotFoundComponent
  }
];
```

### 8.2 Rutas Modulares (Feature Routes)

```typescript
// src/app/features/users/user.routes.ts
import { Routes } from '@angular/router';

export const userRoutes: Routes = [
  {
    path: '',
    component: UserListPageComponent
  },
  {
    path: 'create',
    component: UserFormPageComponent
  },
  {
    path: ':id',
    component: UserDetailPageComponent
  },
  {
    path: ':id/edit',
    component: UserFormPageComponent
  }
];
```

### 8.3 Navegación

```typescript
import { Component, inject } from '@angular/core';
import { Router, RouterLink } from '@angular/router';

@Component({
  template: `
    <!-- Navegación declarativa -->
    <a routerLink="/users">Usuarios</a>
    <a [routerLink]="['/users', userId]">Ver usuario</a>
    <a routerLink="/users/create">Nuevo usuario</a>
    
    <!-- Navegación programática -->
    <button (click)="goToUsers()">Ir a usuarios</button>
  `
})
export class MyComponent {
  private router = inject(Router);
  userId = 123;
  
  goToUsers() {
    this.router.navigate(['/users']);
  }
  
  goToUserDetail(id: number) {
    this.router.navigate(['/users', id]);
  }
  
  goToEditWithQueryParams(id: number) {
    this.router.navigate(['/users', id, 'edit'], {
      queryParams: { returnUrl: '/users' }
    });
  }
}
```

### 8.4 Leer Parámetros de Ruta

```typescript
import { Component, OnInit, inject } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

export class UserDetailComponent implements OnInit {
  private route = inject(ActivatedRoute);
  
  userId: number | null = null;
  
  ngOnInit() {
    // Leer parámetro de ruta
    const id = this.route.snapshot.paramMap.get('id');
    if (id) {
      this.userId = +id;
      this.loadUser(this.userId);
    }
    
    // Leer query params
    const returnUrl = this.route.snapshot.queryParamMap.get('returnUrl');
    console.log('Return URL:', returnUrl);
  }
}
```

---

## 9. Angular Material

### 9.1 Instalación

```bash
ng add @angular/material
```

### 9.2 Componentes Principales

#### Tabla (mat-table)
```typescript
import { MatTableModule } from '@angular/material/table';

@Component({
  imports: [MatTableModule],
  template: `
    <mat-table [dataSource]="users()">
      
      <ng-container matColumnDef="name">
        <mat-header-cell *matHeaderCellDef>Nombre</mat-header-cell>
        <mat-cell *matCellDef="let user">{{ user.name }}</mat-cell>
      </ng-container>
      
      <ng-container matColumnDef="email">
        <mat-header-cell *matHeaderCellDef>Email</mat-header-cell>
        <mat-cell *matCellDef="let user">{{ user.email }}</mat-cell>
      </ng-container>
      
      <mat-header-row *matHeaderRowDef="displayedColumns"></mat-header-row>
      <mat-row *matRowDef="let row; columns: displayedColumns;"></mat-row>
    </mat-table>
  `
})
export class UsersComponent {
  users = signal<User[]>([]);
  displayedColumns = ['name', 'email'];
}
```

#### Cards
```typescript
import { MatCardModule } from '@angular/material/card';

@Component({
  imports: [MatCardModule],
  template: `
    <mat-card>
      <mat-card-header>
        <mat-card-title>Título</mat-card-title>
        <mat-card-subtitle>Subtítulo</mat-card-subtitle>
      </mat-card-header>
      <mat-card-content>
        Contenido del card
      </mat-card-content>
      <mat-card-actions>
        <button mat-button>Acción</button>
      </mat-card-actions>
    </mat-card>
  `
})
```

#### Sidebar (Sidenav)
```typescript
import { MatSidenavModule } from '@angular/material/sidenav';
import { MatToolbarModule } from '@angular/material/toolbar';

@Component({
  imports: [MatSidenavModule, MatToolbarModule],
  template: `
    <mat-sidenav-container style="height: 100vh;">
      <mat-sidenav #drawer mode="side" [opened]="true">
        Contenido del sidebar
      </mat-sidenav>
      
      <mat-sidenav-content>
        <mat-toolbar color="primary">
          <button mat-icon-button (click)="drawer.toggle()">
            <mat-icon>menu</mat-icon>
          </button>
          <span>Mi App</span>
        </mat-toolbar>
        
        <div style="padding: 20px;">
          <router-outlet />
        </div>
      </mat-sidenav-content>
    </mat-sidenav-container>
  `
})
```

#### Botones
```typescript
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';

@Component({
  imports: [MatButtonModule, MatIconModule],
  template: `
    <!-- Botones básicos -->
    <button mat-button>Basic</button>
    <button mat-raised-button>Raised</button>
    <button mat-raised-button color="primary">Primary</button>
    <button mat-raised-button color="accent">Accent</button>
    <button mat-raised-button color="warn">Warn</button>
    
    <!-- Con iconos -->
    <button mat-icon-button>
      <mat-icon>favorite</mat-icon>
    </button>
    
    <button mat-raised-button>
      <mat-icon>save</mat-icon>
      Guardar
    </button>
  `
})
```

#### Diálogos
```typescript
import { MatDialog, MatDialogModule } from '@angular/material/dialog';

@Component({
  template: `
    <button mat-raised-button (click)="openDialog()">
      Abrir diálogo
    </button>
  `
})
export class MyComponent {
  private dialog = inject(MatDialog);
  
  openDialog() {
    const dialogRef = this.dialog.open(ConfirmDialogComponent, {
      width: '400px',
      data: { message: '¿Estás seguro?' }
    });
    
    dialogRef.afterClosed().subscribe(result => {
      if (result) {
        console.log('Usuario confirmó');
      }
    });
  }
}

// Componente del diálogo
@Component({
  template: `
    <h2 mat-dialog-title>Confirmar acción</h2>
    <mat-dialog-content>
      {{ data.message }}
    </mat-dialog-content>
    <mat-dialog-actions align="end">
      <button mat-button mat-dialog-close>Cancelar</button>
      <button mat-raised-button color="primary" [mat-dialog-close]="true">
        Confirmar
      </button>
    </mat-dialog-actions>
  `
})
export class ConfirmDialogComponent {
  data = inject(MAT_DIALOG_DATA);
}
```

---

## 10. Ejemplo CRUD Completo

### 10.1 Modelos

```typescript
// src/app/features/urls/models/url.model.ts

export interface Url {
  id: number;
  name: string | null;
  link: string | null;
  isEnable: boolean;
  id_UrlGrp: number;
}

export interface CreateUrlDto {
  name: string;
  link: string;
  isEnable: boolean;
  id_UrlGrp: number;
}

export interface UpdateUrlDto {
  id: number;
  name?: string;
  link?: string;
  isEnable?: boolean;
  id_UrlGrp?: number;
}
```

### 10.2 Servicio

```typescript
// src/app/features/urls/services/url.service.ts

import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { Url, CreateUrlDto, UpdateUrlDto } from '../models/url.model';

@Injectable({ providedIn: 'root' })
export class UrlService {
  private http = inject(HttpClient);
  private readonly baseUrl = '/api/urls';

  getAllUrls(): Observable<Url[]> {
    return this.http.get<Url[]>(this.baseUrl);
  }

  getUrlById(id: number): Observable<Url> {
    return this.http.get<Url>(`${this.baseUrl}/${id}`);
  }

  createUrl(url: CreateUrlDto): Observable<Url> {
    return this.http.post<Url>(this.baseUrl, url);
  }

  updateUrl(id: number, url: UpdateUrlDto): Observable<Url> {
    return this.http.put<Url>(`${this.baseUrl}/${id}`, url);
  }

  deleteUrl(id: number): Observable<void> {
    return this.http.delete<void>(`${this.baseUrl}/${id}`);
  }

  toggleUrlStatus(id: number): Observable<Url> {
    return this.http.patch<Url>(`${this.baseUrl}/${id}/toggle`, {});
  }
}
```

### 10.3 Lista (READ)

```typescript
// src/app/features/urls/pages/url-list-page/url-list-page.component.ts

import { Component, OnInit, signal, inject } from '@angular/core';
import { MatTableModule } from '@angular/material/table';
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';
import { RouterLink } from '@angular/router';
import { UrlService } from '../../services/url.service';
import { Url } from '../../models/url.model';

@Component({
  selector: 'app-url-list-page',
  standalone: true,
  imports: [MatTableModule, MatButtonModule, MatIconModule, RouterLink],
  template: `
    <div style="padding: 20px;">
      <button mat-raised-button color="primary" routerLink="/urls/create">
        <mat-icon>add</mat-icon>
        Nueva URL
      </button>

      @if (loading()) {
        <p>Cargando...</p>
      } @else {
        <mat-table [dataSource]="urls()" class="mat-elevation-1">
          
          <ng-container matColumnDef="id">
            <mat-header-cell *matHeaderCellDef>ID</mat-header-cell>
            <mat-cell *matCellDef="let url">{{ url.id }}</mat-cell>
          </ng-container>

          <ng-container matColumnDef="name">
            <mat-header-cell *matHeaderCellDef>Nombre</mat-header-cell>
            <mat-cell *matCellDef="let url">{{ url.name }}</mat-cell>
          </ng-container>

          <ng-container matColumnDef="actions">
            <mat-header-cell *matHeaderCellDef>Acciones</mat-header-cell>
            <mat-cell *matCellDef="let url">
              <button mat-icon-button [routerLink]="['/urls', url.id]">
                <mat-icon>visibility</mat-icon>
              </button>
              <button mat-icon-button [routerLink]="['/urls', url.id, 'edit']">
                <mat-icon>edit</mat-icon>
              </button>
              <button mat-icon-button color="warn" (click)="deleteUrl(url.id)">
                <mat-icon>delete</mat-icon>
              </button>
            </mat-cell>
          </ng-container>

          <mat-header-row *matHeaderRowDef="displayedColumns"></mat-header-row>
          <mat-row *matRowDef="let row; columns: displayedColumns;"></mat-row>
        </mat-table>
      }
    </div>
  `
})
export class UrlListPageComponent implements OnInit {
  private urlService = inject(UrlService);
  
  urls = signal<Url[]>([]);
  loading = signal(false);
  displayedColumns = ['id', 'name', 'actions'];

  ngOnInit() {
    this.loadUrls();
  }

  loadUrls() {
    this.loading.set(true);
    this.urlService.getAllUrls().subscribe({
      next: (urls) => {
        this.urls.set(urls);
        this.loading.set(false);
      },
      error: (error) => {
        console.error('Error:', error);
        this.loading.set(false);
      }
    });
  }

  deleteUrl(id: number) {
    if (confirm('¿Eliminar esta URL?')) {
      this.urlService.deleteUrl(id).subscribe({
        next: () => {
          this.urls.update(urls => urls.filter(u => u.id !== id));
        }
      });
    }
  }
}
```

### 10.4 Formulario (CREATE/UPDATE)

```typescript
// src/app/features/urls/pages/url-form-page/url-form-page.component.ts

import { Component, OnInit, signal, inject } from '@angular/core';
import { FormBuilder, FormGroup, Validators, ReactiveFormsModule } from '@angular/forms';
import { Router, ActivatedRoute } from '@angular/router';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatButtonModule } from '@angular/material/button';
import { UrlService } from '../../services/url.service';

@Component({
  selector: 'app-url-form-page',
  standalone: true,
  imports: [
    ReactiveFormsModule,
    MatFormFieldModule,
    MatInputModule,
    MatButtonModule
  ],
  template: `
    <div style="padding: 20px;">
      <h2>{{ isEditMode() ? 'Editar URL' : 'Nueva URL' }}</h2>
      
      <form [formGroup]="urlForm" (ngSubmit)="onSubmit()">
        <mat-form-field appearance="outline">
          <mat-label>Nombre</mat-label>
          <input matInput formControlName="name">
        </mat-form-field>

        <mat-form-field appearance="outline">
          <mat-label>Enlace</mat-label>
          <input matInput formControlName="link" type="url">
        </mat-form-field>

        <mat-form-field appearance="outline">
          <mat-label>ID Grupo</mat-label>
          <input matInput formControlName="id_UrlGrp" type="number">
        </mat-form-field>

        <button mat-raised-button color="primary" type="submit" [disabled]="urlForm.invalid">
          {{ isEditMode() ? 'Actualizar' : 'Crear' }}
        </button>
        
        <button mat-button type="button" (click)="onCancel()">
          Cancelar
        </button>
      </form>
    </div>
  `
})
export class UrlFormPageComponent implements OnInit {
  private fb = inject(FormBuilder);
  private router = inject(Router);
  private route = inject(ActivatedRoute);
  private urlService = inject(UrlService);

  urlForm: FormGroup;
  isEditMode = signal(false);
  urlId = signal<number | null>(null);

  constructor() {
    this.urlForm = this.fb.group({
      name: ['', Validators.required],
      link: ['', [Validators.required, Validators.pattern(/^https?:\/\/.+/)]],
      id_UrlGrp: [1, [Validators.required, Validators.min(1)]],
      isEnable: [true]
    });
  }

  ngOnInit() {
    const id = this.route.snapshot.paramMap.get('id');
    if (id) {
      this.urlId.set(+id);
      this.isEditMode.set(true);
      this.loadUrl(+id);
    }
  }

  loadUrl(id: number) {
    this.urlService.getUrlById(id).subscribe({
      next: (url) => {
        this.urlForm.patchValue(url);
      }
    });
  }

  onSubmit() {
    if (this.urlForm.valid) {
      const formValue = this.urlForm.value;
      
      if (this.isEditMode()) {
        this.urlService.updateUrl(this.urlId()!, formValue).subscribe({
          next: () => this.router.navigate(['/urls'])
        });
      } else {
        this.urlService.createUrl(formValue).subscribe({
          next: () => this.router.navigate(['/urls'])
        });
      }
    }
  }

  onCancel() {
    this.router.navigate(['/urls']);
  }
}
```

### 10.5 Detalle (READ específico)

```typescript
// src/app/features/urls/pages/url-detail-page/url-detail-page.component.ts

import { Component, OnInit, signal, inject } from '@angular/core';
import { ActivatedRoute, Router, RouterLink } from '@angular/router';
import { MatCardModule } from '@angular/material/card';
import { MatButtonModule } from '@angular/material/button';
import { UrlService } from '../../services/url.service';
import { Url } from '../../models/url.model';

@Component({
  selector: 'app-url-detail-page',
  standalone: true,
  imports: [MatCardModule, MatButtonModule, RouterLink],
  template: `
    @if (url()) {
      <mat-card>
        <mat-card-header>
          <mat-card-title>{{ url()!.name }}</mat-card-title>
        </mat-card-header>
        <mat-card-content>
          <p><strong>ID:</strong> {{ url()!.id }}</p>
          <p><strong>Link:</strong> {{ url()!.link }}</p>
          <p><strong>Estado:</strong> {{ url()!.isEnable ? 'Activo' : 'Inactivo' }}</p>
          <p><strong>Grupo:</strong> {{ url()!.id_UrlGrp }}</p>
        </mat-card-content>
        <mat-card-actions>
          <button mat-button routerLink="/urls">Volver</button>
          <button mat-raised-button color="accent" [routerLink]="['/urls', url()!.id, 'edit']">
            Editar
          </button>
          <button mat-raised-button color="warn" (click)="deleteUrl()">
            Eliminar
          </button>
        </mat-card-actions>
      </mat-card>
    }
  `
})
export class UrlDetailPageComponent implements OnInit {
  private route = inject(ActivatedRoute);
  private router = inject(Router);
  private urlService = inject(UrlService);

  url = signal<Url | null>(null);

  ngOnInit() {
    const id = this.route.snapshot.paramMap.get('id');
    if (id) {
      this.urlService.getUrlById(+id).subscribe({
        next: (url) => this.url.set(url)
      });
    }
  }

  deleteUrl() {
    if (confirm('¿Eliminar esta URL?')) {
      this.urlService.deleteUrl(this.url()!.id).subscribe({
        next: () => this.router.navigate(['/urls'])
      });
    }
  }
}
```

### 10.6 Rutas

```typescript
// src/app/features/urls/url.routes.ts

import { Routes } from '@angular/router';
import { UrlListPageComponent } from './pages/url-list-page/url-list-page.component';
import { UrlFormPageComponent } from './pages/url-form-page/url-form-page.component';
import { UrlDetailPageComponent } from './pages/url-detail-page/url-detail-page.component';

export const urlRoutes: Routes = [
  { path: '', component: UrlListPageComponent },
  { path: 'create', component: UrlFormPageComponent },
  { path: ':id', component: UrlDetailPageComponent },
  { path: ':id/edit', component: UrlFormPageComponent }
];
```

---

## 11. Mejores Prácticas

### 11.1 Componentes

✅ **HACER:**
- Usar componentes standalone (sin NgModules)
- NO poner `standalone: true` en decoradores (es el default)
- Usar `input()` y `output()` en lugar de decoradores
- Usar `computed()` para estado derivado
- Mantener componentes pequeños y enfocados
- `changeDetection: ChangeDetectionStrategy.OnPush`

❌ **NO HACER:**
- No usar `@Input()` y `@Output()` decorators
- No usar `@HostBinding` y `@HostListener`
- No crear componentes gigantes con múltiples responsabilidades

### 11.2 Templates

✅ **HACER:**
- Usar control flow nativo (`@if`, `@for`, `@switch`)
- Usar `class` bindings en lugar de `ngClass`
- Usar `style` bindings en lugar de `ngStyle`
- Mantener templates simples, sin lógica compleja

❌ **NO HACER:**
```html
<!-- NO usar *ngIf, *ngFor, *ngSwitch -->
<div *ngIf="condition">...</div>

<!-- NO usar ngClass -->
<div [ngClass]="{'active': isActive}">...</div>

<!-- NO usar ngStyle -->
<div [ngStyle]="{'color': color}">...</div>
```

✅ **SÍ usar:**
```html
<!-- Usar @if -->
@if (condition) {
  <div>...</div>
}

<!-- Usar [class] binding -->
<div [class.active]="isActive">...</div>

<!-- Usar [style] binding -->
<div [style.color]="color">...</div>
```

### 11.3 Estado

✅ **HACER:**
- Usar signals para estado local
- Usar `computed()` para estado derivado
- Mantener transformaciones puras y predecibles
- Usar `update()` o `set()`, NO `mutate()`

```typescript
// ✅ BIEN
users = signal<User[]>([]);
activeUsers = computed(() => this.users().filter(u => u.active));

addUser(user: User) {
  this.users.update(users => [...users, user]);
}

// ❌ MAL
users = signal<User[]>([]);
addUser(user: User) {
  this.users().push(user); // NO HACER - muta directamente
}
```

### 11.4 Servicios

✅ **HACER:**
- Diseñar servicios con una sola responsabilidad
- Usar `providedIn: 'root'` para singleton
- Usar función `inject()` en lugar de constructor injection
- Retornar Observables desde los servicios

```typescript
// ✅ BIEN
@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users');
  }
}

// ❌ MAL
@Injectable({ providedIn: 'root' })
export class UserService {
  constructor(private http: HttpClient) {} // Antigua forma
}
```

### 11.5 TypeScript

✅ **HACER:**
- Usar strict type checking
- Preferir inferencia de tipos cuando sea obvio
- Evitar `any`, usar `unknown` si no conoces el tipo
- Definir interfaces para todos los modelos

```typescript
// ✅ BIEN
interface User {
  id: number;
  name: string;
  email: string;
}

users = signal<User[]>([]);

// ❌ MAL
users = signal<any[]>([]); // Evitar any
```

### 11.6 Optimización

✅ **HACER:**
- Implementar lazy loading para rutas
- Usar `NgOptimizedImage` para imágenes estáticas
- Usar `trackBy` en loops (o `track` en `@for`)
- OnPush change detection

```typescript
// ✅ BIEN - track en @for
@for (user of users(); track user.id) {
  <div>{{ user.name }}</div>
}

// ✅ BIEN - OnPush
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
```

### 11.7 Organización

✅ **HACER:**
- Un feature por carpeta
- Rutas modulares con lazy loading
- Separar páginas de componentes
- Servicios específicos por feature

```
features/
├── users/
│   ├── pages/       # Componentes con ruta
│   ├── components/  # Componentes reutilizables en users
│   ├── services/    # UserService
│   ├── models/      # User interfaces
│   └── user.routes.ts
```

---

## 12. Comandos Útiles

### CLI Commands

```bash
# Crear componente
ng generate component features/users/components/user-card

# Crear servicio
ng generate service features/users/services/user

# Crear guard
ng generate guard core/guards/auth

# Crear interceptor
ng generate interceptor core/interceptors/auth

# Servir aplicación
ng serve

# Build para producción
ng build --configuration production

# Ejecutar tests
ng test

# Lint
ng lint
```

---

## 13. Recursos Adicionales

### Documentación Oficial
- [Angular.dev](https://angular.dev) - Documentación oficial
- [Angular Material](https://material.angular.io) - Componentes Material
- [RxJS](https://rxjs.dev) - Programación reactiva

### Guías de Estilo
- [Angular Style Guide](https://angular.dev/style-guide)
- [Angular AI Guidelines](https://angular.dev/ai/develop-with-ai)

---

## Resumen Final

**Angular 19/20+ en pocas palabras:**

1. **Signals** para estado reactivo
2. **Standalone components** sin módulos
3. **Control flow nativo** (`@if`, `@for`)
4. **inject()** para inyección de dependencias
5. **Formularios reactivos** con validaciones
6. **Lazy loading** para optimización
7. **Angular Material** para UI consistente
8. **Estructura modular** por features

**Flujo típico:**
```
Usuario → Componente → Servicio → HTTP → API
         ↓           ↓
      Template    Signal/Observable
```

**Patrón CRUD:**
- **Lista** (READ): Tabla + acciones
- **Detalle** (READ): Vista completa
- **Formulario** (CREATE/UPDATE): Validaciones
- **Eliminar** (DELETE): Confirmación

---

*Esta guía cubre los fundamentos esenciales de Angular 19/20+ con enfoque en zoneless y sin SSR. Para temas avanzados consulta la documentación oficial.*