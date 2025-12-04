# Module Documentation

## Overview

This document describes the modular architecture of the PlantaE Frontend Application, including feature modules, shared modules, and their dependencies.

---

## Application Architecture

### Clean Architecture Layers

The application follows **Clean Architecture** principles with clear separation of concerns:

```
src/app/
├── plant-care/              # Feature module
│   ├── application/         # Use cases & facades
│   ├── domain/             # Business logic & entities
│   ├── infrastructure/     # External implementations
│   └── presentation/       # UI components & views
├── shared/                 # Shared module
│   ├── domain/            # Shared models
│   ├── infrastructure/    # Shared services & config
│   └── presentation/      # Shared components & pipes
└── app.routes.ts          # Application routing
```

---

## Core Modules

### 1. Application Module (Main)

**Location**: `src/app/`

**Purpose**: Root application configuration and bootstrapping.

#### Configuration Files

**app.config.ts**
```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideHttpClient(withInterceptors([authInterceptor])),
    
    // Repository providers (Dependency Injection)
    { provide: PlantRepository, useExisting: HttpPlantRepository },
    { provide: ProfileRepository, useExisting: HttpProfileRepository },
    { provide: HomeRepository, useExisting: HttpHomeRepository },
    { provide: SensorRepository, useExisting: HttpSensorRepository },
    { provide: DeviceRepository, useExisting: HttpDeviceRepository },
    { provide: NurseryRepository, useExisting: HttpNurseryRepository }
  ]
};
```

#### Key Features
- **Zone.js optimization** with event coalescing
- **Global error listeners** for error boundary
- **HTTP interceptors** for authentication
- **Dependency Injection** using repository pattern
- **Router configuration** with route guards

#### Dependencies
```json
{
  "@angular/core": "^20.3.0",
  "@angular/common": "^20.3.0",
  "@angular/router": "^20.3.0",
  "@angular/forms": "^20.3.0",
  "@angular/platform-browser": "^20.3.0",
  "rxjs": "~7.8.0",
  "zone.js": "~0.15.0"
}
```

---

### 2. Shared Module

**Location**: `src/app/shared/`

**Purpose**: Reusable components, services, and utilities used across the application.

#### Structure

```
shared/
├── domain/
│   └── models/
│       └── paged-result.model.ts      # Generic pagination model
├── infrastructure/
│   ├── config/
│   │   └── app-config.token.ts        # Configuration tokens
│   ├── interceptors/
│   │   └── auth.interceptor.ts        # JWT token interceptor
│   └── services/
│       ├── api-client.service.ts      # HTTP wrapper
│       ├── auth.service.ts            # Authentication
│       ├── i18n.service.ts            # Internationalization
│       └── notification.service.ts    # Toast notifications
└── presentation/
    ├── components/
    │   ├── layout/                    # Main layout wrapper
    │   ├── footer-content/            # Footer component
    │   ├── language-switcher/         # Language toggle
    │   └── toast-container/           # Notification display
    ├── pipes/
    │   └── translate.pipe.ts          # Translation pipe
    └── views/
        ├── home/                      # Public home page
        ├── about/                     # About page
        └── page-not-found/            # 404 page
```

#### Core Services

**ApiClientService**
- Base HTTP client wrapper
- Handles API URL configuration
- Provides type-safe HTTP methods

**AuthService**
- User authentication state management
- Token storage and retrieval
- Login/Register/Logout operations
- Auto-login on app initialization
- Language preference persistence

**I18nService**
- Multi-language support (English, Spanish)
- Namespace-based translations
- Lazy loading of translation files
- Signal-based reactive translations

**NotificationService**
- Toast notification management
- Auto-dismiss functionality
- Type-safe message types (success, error, info)

#### Shared Models

**PagedResult<T>**
```typescript
export interface PagedResult<T> {
  content: T[];
  totalElements: number;
  totalPages: number;
  size: number;
  number: number;
  first: boolean;
  last: boolean;
  empty: boolean;
}
```

#### Configuration Tokens

**PLANTAE_CONFIG**
```typescript
export interface PlantaeFrontConfig {
  apiBaseUrl: string;
  defaultLanguage: string;
  supportedLanguages: string[];
}

export const PLANTAE_CONFIG = new InjectionToken<PlantaeFrontConfig>(
  'plantae.config'
);
```

---

### 3. Plant Care Feature Module

**Location**: `src/app/plant-care/`

**Purpose**: Main business domain for plant management, sensors, devices, and monitoring.

#### Architecture Layers

##### Domain Layer (`domain/`)

**Purpose**: Pure business logic and entities (framework-agnostic)

**Models**:
- `plant.model.ts`: Plant entity and status enum
- `plant-alert.model.ts`: Alert entity
- `plant-save.model.ts`: DTOs for create/update operations
- `plant-search.model.ts`: Search/filter parameters
- `device.model.ts`: Device entity
- `sensor.model.ts`: Sensor entity

**Repositories** (Interfaces):
- `PlantRepository`: Plant CRUD operations
- `DeviceRepository`: Device management
- `SensorRepository`: Sensor data operations
- `HomeRepository`: Home dashboard data
- `NurseryRepository`: Nursery-specific operations
- `ProfileRepository`: User profile management

**Example Model**:
```typescript
export type PlantStatus = 'ACTIVE' | 'INACTIVE' | 'NEEDS_ATTENTION';

export interface Plant {
  id: string;
  ownerId?: string;
  name: string;
  species: string;
  status?: PlantStatus;
  deviceId?: string;
  sensorId?: string;
  createdAt?: string;
  updatedAt?: string;
  hasAlerts?: boolean;
}
```

##### Infrastructure Layer (`infrastructure/`)

**Purpose**: External service implementations

**HTTP Repositories**:
- `HttpPlantRepository`: REST API implementation for plants
- `HttpDeviceRepository`: REST API implementation for devices
- `HttpSensorRepository`: REST API implementation for sensors
- `HttpHomeRepository`: Dashboard data fetching
- `HttpNurseryRepository`: Nursery operations
- `HttpProfileRepository`: User profile API

**Pattern**: Repository pattern with interface segregation

```typescript
@Injectable({ providedIn: 'root' })
export class HttpPlantRepository extends PlantRepository {
  constructor(private readonly api: ApiClientService) {
    super();
  }

  search(params: PlantSearchParams): Observable<PagedResult<Plant>> {
    return this.api.get<PagedResult<Plant>>('plants', params);
  }

  get(id: string): Observable<Plant> {
    return this.api.get<Plant>(`plants/${id}`);
  }

  create(payload: CreatePlantPayload): Observable<Plant> {
    return this.api.post<Plant>('plants', payload);
  }

  update(id: string, payload: UpdatePlantPayload): Observable<Plant> {
    return this.api.put<Plant>(`plants/${id}`, payload);
  }

  delete(id: string): Observable<void> {
    return this.api.delete<void>(`plants/${id}`);
  }
}
```

##### Application Layer (`application/`)

**Purpose**: Use cases and application logic (facades)

**Facades**:
- `PlantFacade`: Plant management operations
- `DeviceFacade`: Device configuration
- `SensorFacade`: Sensor readings and monitoring
- `AlertFacade`: Alert management
- `HomeFacade`: Dashboard data aggregation
- `NurseryFacade`: Nursery-specific features
- `ProfileFacade`: User profile operations

**Services**:
- `IamService`: Identity and access management
- `PlantOptionsService`: Plant species and options
- `ReportService`: Report generation

**Facade Pattern Example**:
```typescript
@Injectable({ providedIn: 'root' })
export class PlantFacade {
  // State management with Signals
  private readonly resultState = signal<PagedResult<Plant> | null>(null);
  private readonly loadingState = signal(false);
  private readonly selectedPlantState = signal<Plant | null>(null);

  // Public readonly signals
  readonly plants: Signal<Plant[]> = computed(() => 
    this.resultState()?.content ?? []
  );
  readonly loading: Signal<boolean> = this.loadingState.asReadonly();
  readonly selectedPlant: Signal<Plant | null> = 
    this.selectedPlantState.asReadonly();

  constructor(
    private readonly plantRepository: PlantRepository,
    private readonly notifications: NotificationService
  ) {}

  loadPlants(params?: PlantSearchParams) {
    this.loadingState.set(true);
    this.plantRepository.search(params)
      .pipe(finalize(() => this.loadingState.set(false)))
      .subscribe({
        next: (result) => this.resultState.set(result),
        error: () => this.notifications.error('Failed to load plants')
      });
  }

  createPlant(payload: CreatePlantPayload) {
    return this.plantRepository.create(payload).pipe(
      tap(() => {
        this.notifications.success('Plant created successfully');
        this.loadPlants(); // Refresh list
      })
    );
  }
}
```

##### Presentation Layer (`presentation/`)

**Purpose**: UI components and user interaction

**Guards**:
- `auth.guard.ts`: Authentication verification
- `account-type.guard.ts`: Role-based access control

**Views** (Page Components):
- `login/`: Authentication page
- `register/`: User registration
- `dashboard/`: Home user dashboard
- `nursery-dashboard/`: Nursery dashboard
- `plant-list/`: Plant inventory
- `plant-detail/`: Single plant view
- `plant-management/`: Plant CRUD operations
- `add-plant/`: Create new plant
- `devices/`: Device management
- `sensors/`: Sensor configuration
- `alerts/`: Alert notifications
- `reports/`: Report generation
- `profile/`: User profile
- `settings/`: Application settings
- `password/`: Password management

---

## Dependency Injection Strategy

### Repository Pattern

**Interface Definition** (Domain Layer):
```typescript
// domain/repositories/plant.repository.ts
export abstract class PlantRepository {
  abstract search(params: PlantSearchParams): Observable<PagedResult<Plant>>;
  abstract get(id: string): Observable<Plant>;
  abstract create(payload: CreatePlantPayload): Observable<Plant>;
  abstract update(id: string, payload: UpdatePlantPayload): Observable<Plant>;
  abstract delete(id: string): Observable<void>;
}
```

**Implementation** (Infrastructure Layer):
```typescript
// infrastructure/repositories/http-plant.repository.ts
@Injectable({ providedIn: 'root' })
export class HttpPlantRepository extends PlantRepository {
  // Implementation using ApiClientService
}
```

**Registration** (App Config):
```typescript
{ provide: PlantRepository, useExisting: HttpPlantRepository }
```

**Usage** (Application Layer):
```typescript
@Injectable({ providedIn: 'root' })
export class PlantFacade {
  constructor(private readonly plantRepository: PlantRepository) {}
  // PlantRepository is injected as HttpPlantRepository
}
```

### Benefits
- **Testability**: Easy to mock repositories
- **Flexibility**: Can swap implementations (e.g., LocalStorageRepository for offline)
- **Separation of Concerns**: Domain layer independent of framework
- **Single Responsibility**: Each layer has clear purpose

---

## HTTP Interceptors

### AuthInterceptor

**Location**: `src/app/shared/infrastructure/interceptors/auth.interceptor.ts`

**Purpose**: Automatically attach JWT token to outgoing requests

```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const auth = inject(AuthService);
  const token = auth.getToken();
  
  if (!token) {
    return next(req);
  }
  
  const cloned = req.clone({
    setHeaders: {
      Authorization: `Bearer ${token}`
    }
  });
  
  return next(cloned);
};
```

**Features**:
- Functional interceptor (Angular 15+)
- Automatic token injection
- Skips if no token available
- Immutable request cloning

**Registration**:
```typescript
provideHttpClient(withInterceptors([authInterceptor]))
```

---

## State Management Pattern

### Signal-Based Reactive State

The application uses **Angular Signals** (Angular 16+) for reactive state management instead of RxJS BehaviorSubjects.

#### Benefits
- **Better performance**: Fine-grained reactivity
- **Simpler API**: No manual subscription management
- **Type safety**: Full TypeScript support
- **Computed values**: Automatic dependency tracking

#### Pattern Implementation

```typescript
@Injectable({ providedIn: 'root' })
export class SomeFacade {
  // Private writable signals
  private readonly dataState = signal<Data[]>([]);
  private readonly loadingState = signal(false);
  private readonly errorState = signal<string | null>(null);
  
  // Public readonly signals
  readonly data = this.dataState.asReadonly();
  readonly loading = this.loadingState.asReadonly();
  readonly error = this.errorState.asReadonly();
  
  // Computed signals
  readonly isEmpty = computed(() => this.data().length === 0);
  readonly hasError = computed(() => this.error() !== null);
  
  loadData() {
    this.loadingState.set(true);
    this.errorState.set(null);
    
    this.repository.getData()
      .pipe(finalize(() => this.loadingState.set(false)))
      .subscribe({
        next: (data) => this.dataState.set(data),
        error: (err) => this.errorState.set(err.message)
      });
  }
}
```

#### Usage in Components

```typescript
@Component({
  template: `
    <div *ngIf="facade.loading()">Loading...</div>
    <div *ngIf="facade.error()">{{ facade.error() }}</div>
    <ul *ngIf="!facade.isEmpty()">
      <li *ngFor="let item of facade.data()">{{ item.name }}</li>
    </ul>
  `
})
export class SomeComponent {
  constructor(public facade: SomeFacade) {
    facade.loadData();
  }
}
```

---

## Internationalization (i18n)

### Implementation Strategy

**Translation Files**:
- `assets/i18n/en.json`: English translations
- `assets/i18n/es.json`: Spanish translations

**File Structure**:
```json
{
  "namespace": {
    "key": "Translation value",
    "nested.key": "Nested value"
  }
}
```

**Example**:
```json
{
  "iam": {
    "login.title": "Sign in",
    "login.email": "Email",
    "login.password": "Password"
  },
  "common": {
    "actions.save": "Save",
    "actions.cancel": "Cancel"
  }
}
```

### I18nService

**Features**:
- Lazy loading of translation files
- Namespace-based organization
- Fallback support
- Signal-based reactivity
- HTTP caching

**Usage**:
```typescript
// In service
constructor(private i18n: I18nService) {}

const translated = this.i18n.translate('iam:login.title', 'Sign in');

// In template with pipe
{{ 'iam:login.title' | t:'Sign in' }}
```

### Language Switching

**Persistence**: Language preference stored in:
1. User profile (backend)
2. Browser localStorage (frontend)

**Change Flow**:
1. User selects language in LanguageSwitcherComponent
2. AuthService.updateLanguage() called
3. Backend updated (if authenticated)
4. I18nService.setLanguage() triggers re-translation
5. All pipes re-evaluate with new language

---

## Guard System

### Authentication Guard

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

### Account Type Guard

```typescript
export const accountTypeGuard: CanActivateFn = (route) => {
  const allowed = route.data?.['accountTypes'] as string[] | undefined;
  
  if (!allowed || allowed.length === 0) {
    return true;
  }
  
  const auth = inject(AuthService);
  const router = inject(Router);
  const accountType = auth.getAccountType();
  
  if (accountType && allowed.includes(accountType)) {
    return true;
  }
  
  router.navigate([auth.getDefaultRoute()]);
  return false;
};
```

**Usage in Routes**:
```typescript
{
  path: 'dashboard',
  component: DashboardViewComponent,
  canActivate: [authGuard, accountTypeGuard],
  data: { accountTypes: ['HOME'] }
}
```

---

## Error Handling Strategy

### Global Error Handler

```typescript
provideBrowserGlobalErrorListeners()
```

### HTTP Error Handling

**In Services**:
```typescript
return this.api.post<AuthResponse>('iam/login', payload)
  .pipe(
    tap((response) => {
      this.notifications.success(response.message);
    }),
    catchError((error: HttpErrorResponse) => {
      const message = error.error?.message || 'An error occurred';
      this.notifications.error(message);
      return throwError(() => error);
    })
  );
```

**In Facades**:
```typescript
loadData() {
  this.loadingState.set(true);
  this.repository.getData()
    .pipe(finalize(() => this.loadingState.set(false)))
    .subscribe({
      next: (data) => this.dataState.set(data),
      error: () => this.notifications.error('Failed to load data')
    });
}
```

---

## Testing Strategy

### Unit Tests

**Service Testing**:
```typescript
describe('AuthService', () => {
  let service: AuthService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [AuthService]
    });
    service = TestBed.inject(AuthService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  it('should login successfully', () => {
    const mockResponse = { token: 'test-token', accountType: 'HOME' };
    
    service.login({ email: 'test@test.com', password: 'pass' })
      .subscribe(response => {
        expect(response.token).toBe('test-token');
      });

    const req = httpMock.expectOne(`${API_URL}/iam/login`);
    expect(req.request.method).toBe('POST');
    req.flush(mockResponse);
  });
});
```

**Facade Testing**:
```typescript
describe('PlantFacade', () => {
  let facade: PlantFacade;
  let repository: jasmine.SpyObj<PlantRepository>;

  beforeEach(() => {
    const spy = jasmine.createSpyObj('PlantRepository', ['search']);
    
    TestBed.configureTestingModule({
      providers: [
        PlantFacade,
        { provide: PlantRepository, useValue: spy }
      ]
    });
    
    facade = TestBed.inject(PlantFacade);
    repository = TestBed.inject(PlantRepository) as jasmine.SpyObj<PlantRepository>;
  });

  it('should load plants', () => {
    const mockPlants = { content: [], totalElements: 0 };
    repository.search.and.returnValue(of(mockPlants));

    facade.loadPlants();

    expect(facade.loading()).toBe(false);
    expect(facade.plants()).toEqual([]);
  });
});
```

---

## Build Configuration

### Development

```bash
npm run start
# Runs: ng serve
# Default port: 4200
# Hot module replacement enabled
```

### Production Build

```bash
npm run build
# Runs: ng build
# Output: dist/
# Optimizations: minification, tree-shaking, AOT compilation
```

### Watch Mode

```bash
npm run watch
# Runs: ng build --watch --configuration development
```

---

## Module Dependencies Graph

```
App Module
├── Shared Module
│   ├── ApiClientService
│   ├── AuthService
│   ├── I18nService
│   ├── NotificationService
│   ├── LayoutComponent
│   ├── ToastContainerComponent
│   └── TranslatePipe
│
└── Plant Care Module
    ├── Domain Layer (Pure TypeScript)
    │   ├── Models
    │   └── Repository Interfaces
    │
    ├── Infrastructure Layer
    │   └── HTTP Repositories (depends on Shared.ApiClientService)
    │
    ├── Application Layer
    │   └── Facades (depends on Repositories, NotificationService)
    │
    └── Presentation Layer
        ├── Guards (depends on AuthService)
        └── Views (depends on Facades, Shared Components)
```

---

## Related Documentation
- [Component Documentation](./COMPONENT_DOCUMENTATION.md)
- [Design Patterns](./DESIGN_PATTERNS.md)
- [Routing Guide](./ROUTING_GUIDE.md)


