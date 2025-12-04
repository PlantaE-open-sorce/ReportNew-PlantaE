# Routing Guide

## Overview

This document provides comprehensive information about the routing configuration, navigation patterns, and route protection strategies implemented in the PlantaE Frontend Application.

---

## Route Configuration

### Main Routes File

**Location**: `src/app/app.routes.ts`

#### Route Structure

```typescript
export const routes: Routes = [
  // Public routes (no layout)
  { path: 'login', component: LoginViewComponent },
  { path: 'register', component: RegisterViewComponent },
  { path: 'forgot-password', component: ForgotPasswordViewComponent },
  
  // Protected routes (with layout)
  {
    path: '',
    component: LayoutComponent,
    children: [
      // Public pages
      { path: '', component: HomeViewComponent },
      { path: 'about', component: AboutViewComponent },
      
      // Protected pages
      {
        path: 'dashboard',
        component: DashboardViewComponent,
        canActivate: [authGuard, accountTypeGuard],
        data: { accountTypes: ['HOME'] }
      },
      {
        path: 'plants',
        component: PlantListViewComponent,
        canActivate: [authGuard]
      },
      {
        path: 'plants/new',
        component: AddPlantViewComponent,
        canActivate: [authGuard]
      },
      {
        path: 'plants/:id',
        component: PlantDetailViewComponent,
        canActivate: [authGuard]
      },
      {
        path: 'management',
        component: PlantManagementViewComponent,
        canActivate: [authGuard, accountTypeGuard],
        data: { accountTypes: ['HOME', 'VIVERO_FORESTAL'] }
      },
      {
        path: 'nursery',
        component: NurseryDashboardViewComponent,
        canActivate: [authGuard, accountTypeGuard],
        data: { accountTypes: ['VIVERO_FORESTAL'] }
      },
      {
        path: 'sensors',
        component: SensorsViewComponent,
        canActivate: [authGuard, accountTypeGuard],
        data: { accountTypes: ['HOME', 'VIVERO_FORESTAL'] }
      },
      {
        path: 'devices',
        component: DevicesViewComponent,
        canActivate: [authGuard, accountTypeGuard],
        data: { accountTypes: ['HOME', 'VIVERO_FORESTAL'] }
      },
      {
        path: 'reports',
        component: ReportsViewComponent,
        canActivate: [authGuard]
      },
      {
        path: 'alerts',
        component: AlertsViewComponent,
        canActivate: [authGuard]
      },
      {
        path: 'profile',
        component: ProfileViewComponent,
        canActivate: [authGuard]
      },
      {
        path: 'profile/change-password',
        component: ChangePasswordViewComponent,
        canActivate: [authGuard]
      },
      {
        path: 'settings',
        component: SettingsViewComponent,
        canActivate: [authGuard]
      }
    ]
  },
  
  // Public profile (outside layout)
  { path: 'u/:slug', component: PublicProfilePageComponent },
  
  // Fallback route
  { path: '**', component: PageNotFoundViewComponent }
];
```

---

## Route Categories

### 1. Public Routes

Routes accessible without authentication.

| Path | Component | Description |
|------|-----------|-------------|
| `/login` | LoginViewComponent | User authentication |
| `/register` | RegisterViewComponent | New user registration |
| `/forgot-password` | ForgotPasswordViewComponent | Password recovery |
| `/` | HomeViewComponent | Landing page |
| `/about` | AboutViewComponent | About page |
| `/u/:slug` | PublicProfilePageComponent | Public user profile |

### 2. Protected Routes

Routes requiring authentication (via `authGuard`).

| Path | Component | Description |
|------|-----------|-------------|
| `/plants` | PlantListViewComponent | Plant inventory |
| `/plants/new` | AddPlantViewComponent | Create new plant |
| `/plants/:id` | PlantDetailViewComponent | Plant details |
| `/reports` | ReportsViewComponent | Reports dashboard |
| `/alerts` | AlertsViewComponent | Alert notifications |
| `/profile` | ProfileViewComponent | User profile |
| `/profile/change-password` | ChangePasswordViewComponent | Change password |
| `/settings` | SettingsViewComponent | App settings |

### 3. Role-Based Routes

Routes with account type restrictions (via `accountTypeGuard`).

#### HOME Account Routes
| Path | Component | Description |
|------|-----------|-------------|
| `/dashboard` | DashboardViewComponent | Home user dashboard |
| `/management` | PlantManagementViewComponent | Plant management (shared) |
| `/sensors` | SensorsViewComponent | Sensor monitoring (shared) |
| `/devices` | DevicesViewComponent | Device config (shared) |

#### VIVERO_FORESTAL Account Routes
| Path | Component | Description |
|------|-----------|-------------|
| `/nursery` | NurseryDashboardViewComponent | Nursery dashboard |
| `/management` | PlantManagementViewComponent | Plant management (shared) |
| `/sensors` | SensorsViewComponent | Sensor monitoring (shared) |
| `/devices` | DevicesViewComponent | Device config (shared) |

---

## Route Guards

### 1. Authentication Guard

**Location**: `src/app/plant-care/presentation/guards/auth.guard.ts`

**Purpose**: Protect routes from unauthenticated access.

#### Implementation

```typescript
export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  const router = inject(Router);

  if (auth.isAuthenticated()) {
    return true;
  }

  router.navigate(['/login']);
  return false;
};
```

#### Behavior
- **Authenticated users**: Allow access
- **Unauthenticated users**: Redirect to `/login`

#### Usage

```typescript
{
  path: 'protected',
  component: ProtectedComponent,
  canActivate: [authGuard]
}
```

---

### 2. Account Type Guard

**Location**: `src/app/plant-care/presentation/guards/account-type.guard.ts`

**Purpose**: Restrict routes based on user account type.

#### Implementation

```typescript
export const accountTypeGuard: CanActivateFn = (route) => {
  const allowed = route.data?.['accountTypes'] as string[] | undefined;
  
  // No restriction if accountTypes not specified
  if (!allowed || allowed.length === 0) {
    return true;
  }
  
  const auth = inject(AuthService);
  const router = inject(Router);
  const accountType = auth.getAccountType();
  
  // Check if user's account type is allowed
  if (accountType && allowed.includes(accountType)) {
    return true;
  }
  
  // Redirect to user's default route
  router.navigate([auth.getDefaultRoute()]);
  return false;
};
```

#### Behavior
- **Allowed account types**: Grant access
- **Other account types**: Redirect to default route
- **No account types specified**: Allow all authenticated users

#### Usage

```typescript
{
  path: 'nursery',
  component: NurseryDashboardComponent,
  canActivate: [authGuard, accountTypeGuard],
  data: { accountTypes: ['VIVERO_FORESTAL'] }
}
```

#### Account Types

- `HOME`: Standard user account
- `VIVERO_FORESTAL`: Nursery grower account

---

### 3. Guard Composition

Guards can be combined and execute in sequence:

```typescript
{
  path: 'protected-route',
  component: SomeComponent,
  canActivate: [authGuard, accountTypeGuard, customGuard],
  data: { accountTypes: ['HOME'] }
}
```

**Execution Order**:
1. `authGuard` - Check authentication
2. `accountTypeGuard` - Check account type
3. `customGuard` - Additional checks

**Important**: If any guard returns `false`, navigation is blocked.

---

## Navigation Patterns

### 1. Router Service Navigation

#### Programmatic Navigation

```typescript
import { Router } from '@angular/router';

export class SomeComponent {
  constructor(private router: Router) {}

  // Simple navigation
  goToPlants() {
    this.router.navigate(['/plants']);
  }

  // Navigation with parameters
  viewPlant(id: string) {
    this.router.navigate(['/plants', id]);
  }

  // Navigation with query parameters
  searchPlants(query: string) {
    this.router.navigate(['/plants'], {
      queryParams: { search: query, page: 1 }
    });
  }

  // Navigation with state
  goToEdit(plant: Plant) {
    this.router.navigate(['/plants', plant.id, 'edit'], {
      state: { plant }
    });
  }

  // Relative navigation
  goToSibling() {
    this.router.navigate(['../sibling'], {
      relativeTo: this.route
    });
  }
}
```

---

### 2. RouterLink Directive

#### Template Navigation

```html
<!-- Simple link -->
<a routerLink="/plants">Plants</a>

<!-- Link with parameters -->
<a [routerLink]="['/plants', plant.id]">View Plant</a>

<!-- Link with query params -->
<a [routerLink]="['/plants']" 
   [queryParams]="{ search: 'rose', status: 'active' }">
  Search
</a>

<!-- Active link styling -->
<a routerLink="/dashboard" 
   routerLinkActive="active"
   [routerLinkActiveOptions]="{ exact: true }">
  Dashboard
</a>

<!-- Fragment navigation -->
<a [routerLink]="['/plants']" fragment="section-1">
  Jump to Section
</a>
```

---

### 3. Route Parameters

#### Path Parameters

**Route Definition**:
```typescript
{ path: 'plants/:id', component: PlantDetailViewComponent }
```

**Access in Component**:
```typescript
export class PlantDetailViewComponent implements OnInit {
  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    // Snapshot (for one-time access)
    const id = this.route.snapshot.params['id'];

    // Observable (for reactive updates)
    this.route.params.subscribe(params => {
      const id = params['id'];
      this.loadPlant(id);
    });

    // Modern approach with RxJS
    this.route.paramMap.subscribe(params => {
      const id = params.get('id');
      if (id) {
        this.loadPlant(id);
      }
    });
  }
}
```

**Navigate with Parameters**:
```typescript
this.router.navigate(['/plants', plantId]);
```

---

#### Query Parameters

**Route Definition**:
```typescript
{ path: 'plants', component: PlantListViewComponent }
```

**Access in Component**:
```typescript
export class PlantListViewComponent implements OnInit {
  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    // Snapshot
    const search = this.route.snapshot.queryParams['search'];
    const page = this.route.snapshot.queryParams['page'];

    // Observable
    this.route.queryParams.subscribe(params => {
      const search = params['search'];
      const page = params['page'] || 1;
      this.loadPlants({ search, page });
    });

    // Modern approach
    this.route.queryParamMap.subscribe(params => {
      const search = params.get('search');
      const page = Number(params.get('page')) || 1;
      this.loadPlants({ search, page });
    });
  }
}
```

**Navigate with Query Parameters**:
```typescript
// Set query params
this.router.navigate(['/plants'], {
  queryParams: { search: 'rose', page: 2 }
});

// Merge with existing params
this.router.navigate([], {
  relativeTo: this.route,
  queryParams: { page: 3 },
  queryParamsHandling: 'merge'
});

// Preserve existing params
this.router.navigate(['/plants'], {
  queryParamsHandling: 'preserve'
});
```

---

### 4. Route Data

#### Static Data

**Route Definition**:
```typescript
{
  path: 'plants',
  component: PlantListViewComponent,
  data: {
    title: 'Plant Management',
    breadcrumb: 'Plants',
    showBackButton: true
  }
}
```

**Access in Component**:
```typescript
export class PlantListViewComponent implements OnInit {
  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    const title = this.route.snapshot.data['title'];
    const breadcrumb = this.route.snapshot.data['breadcrumb'];
  }
}
```

---

### 5. Navigation Extras

#### Complete Options

```typescript
this.router.navigate(['/plants'], {
  // Query parameters
  queryParams: { page: 1, search: 'rose' },
  queryParamsHandling: 'merge', // or 'preserve'
  
  // Fragment (hash)
  fragment: 'section-1',
  
  // State (doesn't appear in URL)
  state: { fromDashboard: true },
  
  // Relative to current route
  relativeTo: this.route,
  
  // Skip location change (don't update browser URL)
  skipLocationChange: false,
  
  // Replace URL in history
  replaceUrl: false
});
```

---

## Route Resolution

### 1. Resolvers

**Purpose**: Pre-fetch data before activating route.

#### Implementation

```typescript
export const plantResolver: ResolveFn<Plant> = (route) => {
  const plantFacade = inject(PlantFacade);
  const id = route.paramMap.get('id');
  
  if (!id) {
    return throwError(() => new Error('Plant ID required'));
  }
  
  return plantFacade.loadPlant(id);
};
```

#### Route Configuration

```typescript
{
  path: 'plants/:id',
  component: PlantDetailViewComponent,
  resolve: { plant: plantResolver }
}
```

#### Access Resolved Data

```typescript
export class PlantDetailViewComponent implements OnInit {
  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    const plant = this.route.snapshot.data['plant'];
    console.log('Resolved plant:', plant);
  }
}
```

---

## Default Routes by Account Type

### AuthService Default Route Logic

```typescript
getDefaultRoute(): string {
  const accountType = this.getAccountType();
  
  switch (accountType) {
    case 'HOME':
      return '/dashboard';
    case 'VIVERO_FORESTAL':
      return '/nursery';
    default:
      return '/';
  }
}
```

### Post-Login Navigation

```typescript
login(credentials: LoginRequest) {
  return this.authService.login(credentials)
    .subscribe({
      next: () => {
        const defaultRoute = this.authService.getDefaultRoute();
        this.router.navigate([defaultRoute]);
      }
    });
}
```

---

## Nested Routes

### Layout Component Pattern

**Parent Route** (Layout):
```typescript
{
  path: '',
  component: LayoutComponent,
  children: [
    { path: 'dashboard', component: DashboardComponent },
    { path: 'plants', component: PlantListComponent }
  ]
}
```

**Layout Component Template**:
```html
<div class="app-layout">
  <aside class="sidebar">
    <nav>
      <a routerLink="/dashboard" routerLinkActive="active">Dashboard</a>
      <a routerLink="/plants" routerLinkActive="active">Plants</a>
    </nav>
  </aside>
  <main>
    <router-outlet></router-outlet>
  </main>
</div>
```

**Result**:
- `/dashboard` → LayoutComponent + DashboardComponent
- `/plants` → LayoutComponent + PlantListComponent

---

## Route Wildcards

### 404 Not Found

**Route Configuration** (must be last):
```typescript
{
  path: '**',
  component: PageNotFoundViewComponent
}
```

**PageNotFoundViewComponent**:
```typescript
@Component({
  template: `
    <div class="not-found">
      <h1>404 - Page Not Found</h1>
      <p>The page you're looking for doesn't exist.</p>
      <a routerLink="/">Go Home</a>
    </div>
  `
})
export class PageNotFoundViewComponent {}
```

---

## URL Strategies

### Hash-Based vs. PathLocationStrategy

#### PathLocationStrategy (Default, Recommended)

```typescript
// No configuration needed - default strategy
// URLs: http://example.com/plants
```

**Benefits**:
- Clean URLs
- Better SEO
- Modern approach

**Server Requirement**:
- Server must redirect all routes to `index.html`

#### HashLocationStrategy

```typescript
import { provideRouter, withHashLocation } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withHashLocation())
  ]
};
```

**URLs**: `http://example.com/#/plants`

**Benefits**:
- No server configuration needed
- Works with any server

**Drawbacks**:
- Less clean URLs
- Not ideal for SEO

---

## Navigation Events

### Router Events

```typescript
export class AppComponent implements OnInit {
  constructor(private router: Router) {}

  ngOnInit() {
    this.router.events.subscribe(event => {
      if (event instanceof NavigationStart) {
        console.log('Navigation started:', event.url);
      }
      
      if (event instanceof NavigationEnd) {
        console.log('Navigation ended:', event.url);
      }
      
      if (event instanceof NavigationError) {
        console.error('Navigation error:', event.error);
      }
      
      if (event instanceof NavigationCancel) {
        console.log('Navigation cancelled');
      }
    });
  }
}
```

### Loading Indicator Example

```typescript
export class LoadingService {
  private loadingState = signal(false);
  readonly isLoading = this.loadingState.asReadonly();

  constructor(private router: Router) {
    this.router.events.subscribe(event => {
      if (event instanceof NavigationStart) {
        this.loadingState.set(true);
      }
      if (event instanceof NavigationEnd || 
          event instanceof NavigationCancel || 
          event instanceof NavigationError) {
        this.loadingState.set(false);
      }
    });
  }
}
```

---

## Lazy Loading Routes

### Feature Module Lazy Loading

```typescript
{
  path: 'admin',
  loadChildren: () => import('./admin/admin.routes')
    .then(m => m.ADMIN_ROUTES)
}
```

### Component Lazy Loading

```typescript
{
  path: 'reports',
  loadComponent: () => import('./reports/reports.view')
    .then(m => m.ReportsViewComponent)
}
```

**Benefits**:
- Reduced initial bundle size
- Faster initial load
- Code splitting
- Load on demand

---

## Router Best Practices

### 1. Route Organization

✅ **Do**:
```typescript
// Organize by feature
const routes: Routes = [
  // Public routes first
  { path: 'login', ... },
  { path: 'register', ... },
  
  // Protected routes in layout
  {
    path: '',
    component: LayoutComponent,
    children: [...]
  },
  
  // Wildcard last
  { path: '**', ... }
];
```

❌ **Don't**:
```typescript
// Mixed organization
const routes: Routes = [
  { path: '**', ... },  // Wildcard too early
  { path: 'login', ... },
  { path: 'protected', canActivate: [authGuard], ... },
  { path: 'public', ... }
];
```

---

### 2. Guard Usage

✅ **Do**:
```typescript
// Use functional guards
export const authGuard: CanActivateFn = () => {
  // Implementation
};

// Compose guards
{
  path: 'protected',
  canActivate: [authGuard, roleGuard]
}
```

❌ **Don't**:
```typescript
// Class-based guards (deprecated)
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate() { ... }
}
```

---

### 3. Parameter Access

✅ **Do**:
```typescript
// Use observables for reactive params
this.route.paramMap.subscribe(params => {
  const id = params.get('id');
});
```

❌ **Don't**:
```typescript
// Snapshot only if params never change
const id = this.route.snapshot.params['id'];
// Component won't update if navigating to same route with different id
```

---

### 4. Type Safety

✅ **Do**:
```typescript
// Type-safe navigation
interface PlantRouteParams {
  id: string;
  mode?: 'view' | 'edit';
}

navigateToPlant(params: PlantRouteParams) {
  this.router.navigate(['/plants', params.id], {
    queryParams: { mode: params.mode }
  });
}
```

---

## Route Testing

### Testing Guard

```typescript
describe('authGuard', () => {
  let authService: jasmine.SpyObj<AuthService>;
  let router: jasmine.SpyObj<Router>;

  beforeEach(() => {
    authService = jasmine.createSpyObj('AuthService', ['isAuthenticated']);
    router = jasmine.createSpyObj('Router', ['navigate']);
    
    TestBed.configureTestingModule({
      providers: [
        { provide: AuthService, useValue: authService },
        { provide: Router, useValue: router }
      ]
    });
  });

  it('should allow authenticated users', () => {
    authService.isAuthenticated.and.returnValue(true);
    
    const result = TestBed.runInInjectionContext(() => authGuard());
    
    expect(result).toBe(true);
  });

  it('should redirect unauthenticated users', () => {
    authService.isAuthenticated.and.returnValue(false);
    
    const result = TestBed.runInInjectionContext(() => authGuard());
    
    expect(result).toBe(false);
    expect(router.navigate).toHaveBeenCalledWith(['/login']);
  });
});
```

### Testing Navigation

```typescript
describe('PlantListComponent', () => {
  let component: PlantListComponent;
  let router: Router;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [PlantListComponent, RouterTestingModule]
    }).compileComponents();

    router = TestBed.inject(Router);
    spyOn(router, 'navigate');
  });

  it('should navigate to plant detail', () => {
    const plantId = '123';
    component.viewPlant(plantId);
    
    expect(router.navigate).toHaveBeenCalledWith(['/plants', plantId]);
  });
});
```

---

## Troubleshooting

### Common Issues

#### 1. Guards Not Executing

**Problem**: Guard not blocking navigation

**Solution**: Ensure guard is registered in route configuration
```typescript
{
  path: 'protected',
  canActivate: [authGuard],  // ← Don't forget!
  component: ProtectedComponent
}
```

---

#### 2. Wildcard Route Matching Everything

**Problem**: 404 page shows for all routes

**Solution**: Place wildcard route last
```typescript
const routes: Routes = [
  { path: 'home', ... },
  { path: 'about', ... },
  { path: '**', component: NotFoundComponent }  // ← Last!
];
```

---

#### 3. Route Parameters Not Updating

**Problem**: Component doesn't update when navigating to same route with different parameters

**Solution**: Use observable instead of snapshot
```typescript
// Bad
const id = this.route.snapshot.params['id'];

// Good
this.route.paramMap.subscribe(params => {
  const id = params.get('id');
  this.loadData(id);
});
```

---

## Related Documentation
- [Component Documentation](./COMPONENT_DOCUMENTATION.md)
- [Module Documentation](./MODULE_DOCUMENTATION.md)
- [Design Patterns](./DESIGN_PATTERNS.md)


