# Angular CLI: Comandos `ng generate` (v20+)

Guía completa de los comandos de generación en Angular, con **casos de uso reales** y ejemplos prácticos.

---

## 📌 Tabla de Contenidos
1. [Features](#-features)
2. [Componentes](#-componentes)
3. [Directivas](#-directivas)
4. [Pipes](#-pipes)
5. [Servicios](#-servicios)
6. [Guards](#-guards)
7. [Interceptors](#-interceptors)
8. [Módulos](#-módulos)
9. [Casos de Uso Completos](#-casos-de-uso-completos)

---

## 🛠️ Comandos con Casos Reales

### 🔹 Features

#### **Caso: Módulo completo de gestión de productos**
**Comando:**  
```bash
ng generate feature products --standalone --routing
# O también:
ng generate feature features/user-management --standalone --routing
ng generate feature features/e-commerce --standalone --routing
```

**Estructura generada:**

```
src/app/features/products/
├── components/
│   ├── product-list/
│   ├── product-detail/
│   ├── product-form/
│   └── product-card/
├── services/
│   ├── product.service.ts
│   └── product-state.service.ts
├── models/
│   └── product.model.ts
├── guards/
│   └── product-access.guard.ts
└── products.routes.ts
```

**Código generado:**

```typescript
// products.routes.ts - Rutas del feature
import { Routes } from '@angular/router';
import { productAccessGuard } from './guards/product-access.guard';

export const PRODUCTS_ROUTES: Routes = [
  {
    path: '',
    children: [
      {
        path: '',
        loadComponent: () => import('./components/product-list/product-list.component')
          .then(m => m.ProductListComponent),
        canActivate: [productAccessGuard]
      },
      {
        path: 'new',
        loadComponent: () => import('./components/product-form/product-form.component')
          .then(m => m.ProductFormComponent),
        data: { mode: 'create', roles: ['admin', 'manager'] }
      },
      {
        path: ':id',
        loadComponent: () => import('./components/product-detail/product-detail.component')
          .then(m => m.ProductDetailComponent)
      },
      {
        path: ':id/edit',
        loadComponent: () => import('./components/product-form/product-form.component')
          .then(m => m.ProductFormComponent),
        data: { mode: 'edit', roles: ['admin', 'manager'] }
      }
    ]
  }
];
```

```typescript
// product.service.ts - Servicio del feature
import { Injectable, inject, signal } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Product } from '../models/product.model';
import { BehaviorSubject, Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class ProductService {
  private http = inject(HttpClient);
  private apiUrl = '/api/products';
  
  // Estado con signals
  products = signal<Product[]>([]);
  loading = signal(false);
  error = signal<string | null>(null);

  // Stream para productos en tiempo real
  private productsSubject = new BehaviorSubject<Product[]>([]);
  products$ = this.productsSubject.asObservable();

  getProducts(): Observable<Product[]> {
    this.loading.set(true);
    return this.http.get<Product[]>(this.apiUrl).pipe(
      tap(products => {
        this.products.set(products);
        this.productsSubject.next(products);
        this.loading.set(false);
      }),
      catchError(error => {
        this.error.set('Error cargando productos');
        this.loading.set(false);
        return throwError(() => error);
      })
    );
  }

  getProduct(id: number): Observable<Product> {
    return this.http.get<Product>(`${this.apiUrl}/${id}`);
  }

  createProduct(product: Partial<Product>): Observable<Product> {
    return this.http.post<Product>(this.apiUrl, product).pipe(
      tap(newProduct => {
        const currentProducts = this.products();
        this.products.set([...currentProducts, newProduct]);
        this.productsSubject.next([...currentProducts, newProduct]);
      })
    );
  }

  updateProduct(id: number, product: Partial<Product>): Observable<Product> {
    return this.http.put<Product>(`${this.apiUrl}/${id}`, product).pipe(
      tap(updatedProduct => {
        const currentProducts = this.products();
        const index = currentProducts.findIndex(p => p.id === id);
        if (index > -1) {
          const newProducts = [...currentProducts];
          newProducts[index] = updatedProduct;
          this.products.set(newProducts);
          this.productsSubject.next(newProducts);
        }
      })
    );
  }

  deleteProduct(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`).pipe(
      tap(() => {
        const currentProducts = this.products().filter(p => p.id !== id);
        this.products.set(currentProducts);
        this.productsSubject.next(currentProducts);
      })
    );
  }
}
```

```typescript
// product.model.ts
export interface Product {
  id: number;
  name: string;
  description: string;
  price: number;
  category: string;
  imageUrl: string;
  stock: number;
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;
}

export interface ProductFilters {
  category?: string;
  minPrice?: number;
  maxPrice?: number;
  inStock?: boolean;
  search?: string;
}

export interface ProductFormData {
  name: string;
  description: string;
  price: number;
  category: string;
  imageUrl: string;
  stock: number;
  isActive: boolean;
}
```

```typescript
// product-list.component.ts
import { Component, OnInit, inject, signal, computed } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';
import { FormsModule } from '@angular/forms';
import { ProductService } from '../../services/product.service';
import { ProductCardComponent } from '../product-card/product-card.component';
import { Product, ProductFilters } from '../../models/product.model';

@Component({
  standalone: true,
  imports: [CommonModule, RouterModule, FormsModule, ProductCardComponent],
  selector: 'app-product-list',
  template: `
    <div class="product-list-container">
      <div class="filters-section">
        <h2>Gestión de Productos</h2>
        
        <div class="filters">
          <input 
            [(ngModel)]="searchTerm" 
            placeholder="Buscar productos..."
            (input)="onSearch()">
          
          <select [(ngModel)]="selectedCategory" (change)="onCategoryChange()">
            <option value="">Todas las categorías</option>
            <option *ngFor="let cat of categories()" [value]="cat">{{ cat }}</option>
          </select>
          
          <label>
            <input type="checkbox" [(ngModel)]="showOnlyInStock" (change)="onStockFilter()">
            Solo en stock
          </label>
          
          <button routerLink="/products/new" class="btn-primary">
            Nuevo Producto
          </button>
        </div>
      </div>

      <div class="products-grid" *ngIf="!productService.loading(); else loadingTemplate">
        <app-product-card 
          *ngFor="let product of filteredProducts()" 
          [product]="product"
          (edit)="onEdit($event)"
          (delete)="onDelete($event)">
        </app-product-card>
      </div>

      <ng-template #loadingTemplate>
        <div class="loading">Cargando productos...</div>
      </ng-template>

      <div *ngIf="productService.error()" class="error">
        {{ productService.error() }}
      </div>
    </div>
  `,
  styleUrls: ['./product-list.component.scss']
})
export class ProductListComponent implements OnInit {
  productService = inject(ProductService);
  
  // Filtros reactivos
  searchTerm = signal('');
  selectedCategory = signal('');
  showOnlyInStock = signal(false);

  // Computed properties
  categories = computed(() => {
    const products = this.productService.products();
    return [...new Set(products.map(p => p.category))];
  });

  filteredProducts = computed(() => {
    let products = this.productService.products();
    
    // Filtro por búsqueda
    const search = this.searchTerm().toLowerCase();
    if (search) {
      products = products.filter(p => 
        p.name.toLowerCase().includes(search) ||
        p.description.toLowerCase().includes(search)
      );
    }
    
    // Filtro por categoría
    const category = this.selectedCategory();
    if (category) {
      products = products.filter(p => p.category === category);
    }
    
    // Filtro por stock
    if (this.showOnlyInStock()) {
      products = products.filter(p => p.stock > 0);
    }
    
    return products;
  });

  ngOnInit() {
    this.loadProducts();
  }

  loadProducts() {
    this.productService.getProducts().subscribe();
  }

  onSearch() {
    // Los computed se actualizan automáticamente
  }

  onCategoryChange() {
    // Los computed se actualizan automáticamente
  }

  onStockFilter() {
    // Los computed se actualizan automáticamente
  }

  onEdit(product: Product) {
    // Navegar a edición se maneja en el componente hijo
  }

  onDelete(product: Product) {
    if (confirm(`¿Estás seguro de eliminar "${product.name}"?`)) {
      this.productService.deleteProduct(product.id).subscribe();
    }
  }
}
```

#### **Caso: Feature de User Management**
**Comando:**
```bash
ng generate feature features/user-management --standalone --routing
```

**Integración en app.routes.ts:**
```typescript
// app.routes.ts - Lazy loading de features
export const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'login', component: LoginComponent, canActivate: [authGuard] },
  { path: 'dashboard', component: DashboardComponent, canActivate: [authGuard] },
  
  // Features con lazy loading
  {
    path: 'products',
    loadChildren: () => import('./features/products/products.routes').then(m => m.PRODUCTS_ROUTES),
    canActivate: [authGuard, roleGuard],
    data: { roles: ['admin', 'manager', 'employee'] }
  },
  {
    path: 'users',
    loadChildren: () => import('./features/user-management/user-management.routes').then(m => m.USER_MANAGEMENT_ROUTES),
    canActivate: [authGuard, roleGuard],
    data: { roles: ['admin'] }
  },
  {
    path: 'reports',
    loadChildren: () => import('./features/reports/reports.routes').then(m => m.REPORTS_ROUTES),
    canActivate: [authGuard, roleGuard],
    data: { roles: ['admin', 'manager'] }
  }
];
```

### 🔹 Componentes

#### **Caso: Sistema de Login/Dashboard**
**Comando:**  
```bash
ng generate component auth/login --standalone --style=scss
ng generate component dashboard --standalone --style=scss
ng generate component shared/navbar --standalone --style=scss
```

**Código generado:**

```typescript
// login.component.ts
import { Component, inject } from '@angular/core';
import { FormBuilder, ReactiveFormsModule, Validators } from '@angular/forms';
import { Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

@Component({
  standalone: true,
  imports: [ReactiveFormsModule],
  selector: 'app-login',
  template: `
    <form [formGroup]="loginForm" (ngSubmit)="onLogin()">
      <input formControlName="email" placeholder="Email" type="email">
      <input formControlName="password" placeholder="Password" type="password">
      <button type="submit" [disabled]="loginForm.invalid">Iniciar Sesión</button>
    </form>
  `,
  styleUrls: ['./login.component.scss']
})
export class LoginComponent {
  private fb = inject(FormBuilder);
  private authService = inject(AuthService);
  private router = inject(Router);

  loginForm = this.fb.group({
    email: ['', [Validators.required, Validators.email]],
    password: ['', [Validators.required, Validators.minLength(6)]]
  });

  onLogin() {
    if (this.loginForm.valid) {
      this.authService.login(this.loginForm.value).subscribe({
        next: () => this.router.navigate(['/dashboard']),
        error: (err) => console.error('Error de login:', err)
      });
    }
  }
}
```

```typescript
// dashboard.component.ts
@Component({
  standalone: true,
  imports: [NavbarComponent],
  selector: 'app-dashboard',
  template: `
    <app-navbar></app-navbar>
    <div class="dashboard-content">
      <h1>Bienvenido, {{ userName }}!</h1>
      <div class="stats-grid">
        <div class="stat-card">Usuarios: {{ userCount }}</div>
        <div class="stat-card">Ventas: {{ salesCount }}</div>
      </div>
    </div>
  `,
  styleUrls: ['./dashboard.component.scss']
})
export class DashboardComponent implements OnInit {
  userName = '';
  userCount = 0;
  salesCount = 0;

  ngOnInit() {
    // Cargar datos del dashboard
    this.loadDashboardData();
  }
}
```

### 🔹 Directivas

#### **Caso: Tooltip personalizado y validación visual**
**Comando:**
```bash
ng generate directive shared/tooltip --standalone
ng generate directive shared/highlight-errors --standalone
```

**Código generado:**

```typescript
// tooltip.directive.ts
import { Directive, ElementRef, HostListener, Input, inject } from '@angular/core';

@Directive({
  standalone: true,
  selector: '[appTooltip]'
})
export class TooltipDirective {
  @Input() appTooltip = '';
  private el = inject(ElementRef);
  private tooltipElement: HTMLElement | null = null;

  @HostListener('mouseenter') onMouseEnter() {
    this.showTooltip();
  }

  @HostListener('mouseleave') onMouseLeave() {
    this.hideTooltip();
  }

  private showTooltip() {
    this.tooltipElement = document.createElement('div');
    this.tooltipElement.className = 'custom-tooltip';
    this.tooltipElement.textContent = this.appTooltip;
    document.body.appendChild(this.tooltipElement);
    
    const rect = this.el.nativeElement.getBoundingClientRect();
    this.tooltipElement.style.left = `${rect.left + rect.width / 2}px`;
    this.tooltipElement.style.top = `${rect.top - 30}px`;
  }

  private hideTooltip() {
    if (this.tooltipElement) {
      document.body.removeChild(this.tooltipElement);
      this.tooltipElement = null;
    }
  }
}
```

```typescript
// highlight-errors.directive.ts - Para formularios
@Directive({
  standalone: true,
  selector: '[appHighlightErrors]'
})
export class HighlightErrorsDirective implements OnInit {
  @Input() appHighlightErrors: AbstractControl | null = null;
  private el = inject(ElementRef);

  ngOnInit() {
    if (this.appHighlightErrors) {
      this.appHighlightErrors.statusChanges.subscribe(status => {
        if (status === 'INVALID' && this.appHighlightErrors?.touched) {
          this.el.nativeElement.classList.add('error-highlight');
        } else {
          this.el.nativeElement.classList.remove('error-highlight');
        }
      });
    }
  }
}
```

### 🔹 Pipes

#### **Caso: Formateo de datos de usuario**
**Comando:**
```bash
ng generate pipe shared/phone-format --standalone
ng generate pipe shared/time-ago --standalone
ng generate pipe shared/currency-local --standalone
```

**Código generado:**

```typescript
// phone-format.pipe.ts - Formatear números de teléfono
@Pipe({
  standalone: true,
  name: 'phoneFormat'
})
export class PhoneFormatPipe implements PipeTransform {
  transform(value: string): string {
    if (!value) return '';
    
    // Formato: +56 9 1234 5678
    const cleaned = value.replace(/\D/g, '');
    if (cleaned.length === 11 && cleaned.startsWith('569')) {
      return `+56 9 ${cleaned.slice(3, 7)} ${cleaned.slice(7)}`;
    }
    return value;
  }
}
```

```typescript
// time-ago.pipe.ts - "hace 2 horas", "hace 3 días"
@Pipe({
  standalone: true,
  name: 'timeAgo'
})
export class TimeAgoPipe implements PipeTransform {
  transform(value: Date | string): string {
    const now = new Date();
    const date = new Date(value);
    const diff = now.getTime() - date.getTime();
    
    const minutes = Math.floor(diff / 60000);
    const hours = Math.floor(minutes / 60);
    const days = Math.floor(hours / 24);
    
    if (minutes < 1) return 'hace un momento';
    if (minutes < 60) return `hace ${minutes} minutos`;
    if (hours < 24) return `hace ${hours} horas`;
    return `hace ${days} días`;
  }
}
```

### 🔹 Servicios

#### **Caso: Sistema completo de autenticación y datos**
**Comando:**
```bash
ng generate service core/auth
ng generate service core/user
ng generate service core/notification
```

**Código generado:**

```typescript
// auth.service.ts
import { Injectable, inject, signal } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Router } from '@angular/router';
import { BehaviorSubject, tap } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class AuthService {
  private http = inject(HttpClient);
  private router = inject(Router);
  
  private isLoggedInSubject = new BehaviorSubject<boolean>(this.hasToken());
  isLoggedIn$ = this.isLoggedInSubject.asObservable();
  
  // Signal para el estado actual
  currentUser = signal<any>(null);

  login(credentials: {email: string, password: string}) {
    return this.http.post<{token: string, user: any}>('/api/auth/login', credentials)
      .pipe(
        tap(response => {
          localStorage.setItem('token', response.token);
          this.currentUser.set(response.user);
          this.isLoggedInSubject.next(true);
        })
      );
  }

  logout() {
    localStorage.removeItem('token');
    this.currentUser.set(null);
    this.isLoggedInSubject.next(false);
    this.router.navigate(['/login']);
  }

  isAuthenticated(): boolean {
    return this.hasToken();
  }

  private hasToken(): boolean {
    return !!localStorage.getItem('token');
  }
}
```

```typescript
// notification.service.ts - Sistema de notificaciones
@Injectable({ providedIn: 'root' })
export class NotificationService {
  private notifications = signal<Array<{id: number, message: string, type: 'success' | 'error' | 'info'}>>([]);
  
  notifications$ = computed(() => this.notifications());

  showSuccess(message: string) {
    this.addNotification(message, 'success');
  }

  showError(message: string) {
    this.addNotification(message, 'error');
  }

  private addNotification(message: string, type: 'success' | 'error' | 'info') {
    const id = Date.now();
    this.notifications.update(current => [...current, { id, message, type }]);
    
    // Auto-remove después de 5 segundos
    setTimeout(() => this.removeNotification(id), 5000);
  }

  removeNotification(id: number) {
    this.notifications.update(current => current.filter(n => n.id !== id));
  }
}
```

### 🔹 Guards

#### **Caso: Sistema de rutas protegidas con redirección inteligente**
**Comando:**
```bash
ng generate guard core/auth --implements=CanActivate
ng generate guard core/role --implements=CanActivate
ng generate guard core/unsaved-changes --implements=CanDeactivate
```

**Código generado:**

```typescript
// auth.guard.ts - Redirección inteligente según estado de login
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from './auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    // Si ya está logueado y trata de acceder al login, redirigir al dashboard
    if (state.url === '/login') {
      router.navigate(['/dashboard']);
      return false;
    }
    return true;
  } else {
    // Si no está logueado y trata de acceder a ruta protegida
    if (state.url !== '/login') {
      // Guardar la URL a la que quería ir para redirigir después del login
      localStorage.setItem('redirectUrl', state.url);
      router.navigate(['/login']);
      return false;
    }
    return true;
  }
};
```

```typescript
// role.guard.ts - Control por roles de usuario
export const roleGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);
  const notificationService = inject(NotificationService);

  const currentUser = authService.currentUser();
  const requiredRoles = route.data['roles'] as string[];

  if (!currentUser || !requiredRoles.some(role => currentUser.roles.includes(role))) {
    notificationService.showError('No tienes permisos para acceder a esta sección');
    router.navigate(['/dashboard']);
    return false;
  }

  return true;
};
```

```typescript
// unsaved-changes.guard.ts - Prevenir pérdida de datos
export const unsavedChangesGuard: CanDeactivateFn<any> = (component) => {
  if (component.hasUnsavedChanges && component.hasUnsavedChanges()) {
    return confirm('¿Estás seguro de que quieres salir? Los cambios no guardados se perderán.');
  }
  return true;
};
```

### 🔹 Interceptors

#### **Caso: Manejo completo de HTTP con autenticación y errores**
**Comando:**
```bash
ng generate interceptor core/auth --functional
ng generate interceptor core/error-handler --functional
ng generate interceptor core/loading --functional
```

**Código generado:**

```typescript
// auth.interceptor.ts - Agregar token automáticamente
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = localStorage.getItem('token');
  
  if (token && !req.url.includes('/auth/')) {
    const authReq = req.clone({
      headers: req.headers.set('Authorization', `Bearer ${token}`)
    });
    return next(authReq);
  }
  
  return next(req);
};
```

```typescript
// error-handler.interceptor.ts - Manejo centralizado de errores
export const errorHandlerInterceptor: HttpInterceptorFn = (req, next) => {
  const notificationService = inject(NotificationService);
  const authService = inject(AuthService);
  const router = inject(Router);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      switch (error.status) {
        case 401:
          notificationService.showError('Sesión expirada. Por favor, inicia sesión nuevamente.');
          authService.logout();
          break;
        case 403:
          notificationService.showError('No tienes permisos para realizar esta acción.');
          break;
        case 404:
          notificationService.showError('Recurso no encontrado.');
          break;
        case 500:
          notificationService.showError('Error interno del servidor. Inténtalo más tarde.');
          break;
        default:
          notificationService.showError('Ha ocurrido un error inesperado.');
      }
      
      return throwError(() => error);
    })
  );
};
```

```typescript
// loading.interceptor.ts - Indicador de carga global
export const loadingInterceptor: HttpInterceptorFn = (req, next) => {
  const loadingService = inject(LoadingService);
  
  loadingService.show();
  
  return next(req).pipe(
    finalize(() => loadingService.hide())
  );
};
```

## 📂 Estructura de Proyecto Real

```
src/
├── app/
│   ├── core/
│   │   ├── guards/
│   │   │   ├── auth.guard.ts
│   │   │   ├── role.guard.ts
│   │   │   └── unsaved-changes.guard.ts
│   │   ├── interceptors/
│   │   │   ├── auth.interceptor.ts
│   │   │   ├── error-handler.interceptor.ts
│   │   │   └── loading.interceptor.ts
│   │   └── services/
│   │       ├── auth.service.ts
│   │       ├── user.service.ts
│   │       └── notification.service.ts
│   ├── features/                    # 🆕 Features modulares
│   │   ├── products/
│   │   │   ├── components/
│   │   │   │   ├── product-list/
│   │   │   │   ├── product-detail/
│   │   │   │   ├── product-form/
│   │   │   │   └── product-card/
│   │   │   ├── services/
│   │   │   │   └── product.service.ts
│   │   │   ├── models/
│   │   │   │   └── product.model.ts
│   │   │   ├── guards/
│   │   │   │   └── product-access.guard.ts
│   │   │   └── products.routes.ts
│   │   ├── user-management/
│   │   │   ├── components/
│   │   │   ├── services/
│   │   │   ├── models/
│   │   │   └── user-management.routes.ts
│   │   └── reports/
│   │       ├── components/
│   │       ├── services/
│   │       └── reports.routes.ts
│   ├── auth/
│   │   ├── login/
│   │   │   └── login.component.ts
│   │   └── register/
│   │       └── register.component.ts
│   ├── dashboard/
│   │   └── dashboard.component.ts
│   └── shared/
│       ├── directives/
│       │   ├── tooltip.directive.ts
│       │   └── highlight-errors.directive.ts
│       └── pipes/
│           ├── phone-format.pipe.ts
│           ├── time-ago.pipe.ts
│           └── currency-local.pipe.ts
```

## 🔥 Casos de Uso Completos

### **1. Sistema de Autenticación Completo**

**app.routes.ts:**
```typescript
export const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { 
    path: 'login', 
    component: LoginComponent,
    canActivate: [authGuard] // Redirige al dashboard si ya está logueado
  },
  { 
    path: 'dashboard', 
    component: DashboardComponent,
    canActivate: [authGuard] // Requiere estar logueado
  },
  {
    path: 'admin',
    loadComponent: () => import('./admin/admin.component').then(m => m.AdminComponent),
    canActivate: [authGuard, roleGuard],
    data: { roles: ['admin'] } // Solo administradores
  },
  {
    path: 'profile',
    component: ProfileComponent,
    canActivate: [authGuard],
    canDeactivate: [unsavedChangesGuard] // Previene pérdida de datos
  }
];
```

### **2. Uso en Template con Pipes y Directivas**

```typescript
// user-list.component.ts
@Component({
  standalone: true,
  imports: [PhoneFormatPipe, TimeAgoPipe, TooltipDirective, HighlightErrorsDirective],
  template: `
    <div class="user-card" *ngFor="let user of users">
      <h3 appTooltip="Click para ver perfil">{{ user.name }}</h3>
      <p>Teléfono: {{ user.phone | phoneFormat }}</p>
      <p>Último acceso: {{ user.lastLogin | timeAgo }}</p>
      
      <input 
        [(ngModel)]="user.email" 
        [appHighlightErrors]="emailControl"
        placeholder="Email">
    </div>
  `
})
export class UserListComponent {
  users = [
    { name: 'Juan Pérez', phone: '56912345678', lastLogin: new Date('2024-01-15T10:30:00') },
    { name: 'María González', phone: '56987654321', lastLogin: new Date('2024-01-14T15:45:00') }
  ];
  
  emailControl = new FormControl('', [Validators.email]);
}
```

### **3. Configuración de Interceptors en main.ts**

```typescript
// main.ts
bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(
      withInterceptors([
        loadingInterceptor,
        authInterceptor,
        errorHandlerInterceptor
      ])
    ),
    // ... otros providers
  ]
});
```

## 📊 Comparación de Opciones

| Comando | ¿Standalone? | ¿Genera Template? | ¿Inyectable? | Uso Principal |
|---------|--------------|-------------------|--------------|---------------|
| **feature** | ✅ | ✅ | ❌ | **Módulos completos con rutas** |
| component | ✅ | ✅ | ❌ | UI, páginas, formularios |
| directive | ✅ | ❌ | ❌ | Comportamiento DOM |
| service | ❌ | ❌ | ✅ | Lógica de negocio, HTTP |
| guard | ✅ | ❌ | ✅ | Protección de rutas |
| interceptor | ✅ | ❌ | ✅ | Interceptar HTTP requests |
| pipe | ✅ | ❌ | ❌ | Transformación de datos |

## 📌 Tips Avanzados

- **Usa `--dry-run`** para simular la generación: `ng g c mi-componente --dry-run`
- **Genera múltiples archivos**: `ng g c auth/login auth/register auth/forgot-password`
- **Estructura modular**: Agrupa por funcionalidad, no por tipo de archivo
- **Signals + RxJS**: Combina signals para estado local y RxJS para streams complejos
- **Guards funcionales**: Usa la nueva sintaxis funcional en lugar de clases
- **Interceptors funcionales**: Más simples y fáciles de testear

---

### 🚀 Flujo Completo de Desarrollo

1. **Generar estructura base**: `ng g c auth/login --standalone`
2. **Crear servicios**: `ng g s core/auth`
3. **Agregar guards**: `ng g guard core/auth --implements=CanActivate`
4. **Configurar interceptors**: `ng g interceptor core/auth --functional`
5. **Crear pipes y directivas**: `ng g pipe shared/format --standalone`

¡Con estos ejemplos reales tendrás todo lo necesario para construir aplicaciones Angular robustas y escalables! 🎯