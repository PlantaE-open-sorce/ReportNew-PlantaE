# Design Patterns

## Overview

This document outlines the design patterns, architectural decisions, and best practices implemented in the PlantaE Frontend Application.

---

## Architectural Patterns

### 1. Clean Architecture

**Purpose**: Maintain separation of concerns and ensure business logic independence from framework details.

#### Layer Structure

```
┌─────────────────────────────────────┐
│     Presentation Layer              │
│   (Components, Views, Guards)       │
└──────────────┬──────────────────────┘
               │
┌──────────────┴──────────────────────┐
│     Application Layer               │
│   (Facades, Use Cases, Services)    │
└──────────────┬──────────────────────┘
               │
┌──────────────┴──────────────────────┐
│     Domain Layer                    │
│   (Models, Repository Interfaces)   │
└──────────────┬──────────────────────┘
               │
┌──────────────┴──────────────────────┐
│     Infrastructure Layer            │
│   (HTTP Repos, External Services)   │
└─────────────────────────────────────┘
```

#### Dependency Rule
- **Inner layers** know nothing about outer layers
- **Outer layers** depend on inner layers
- **Domain layer** is pure TypeScript (no Angular dependencies)
- **Infrastructure** implements domain interfaces

#### Example Implementation

**Domain Layer** (Business Rules):
```typescript
// domain/models/plant.model.ts
export interface Plant {
  id: string;
  name: string;
  species: string;
  status: PlantStatus;
}

// domain/repositories/plant.repository.ts
export abstract class PlantRepository {
  abstract getById(id: string): Observable<Plant>;
  abstract save(plant: Plant): Observable<Plant>;
}
```

**Infrastructure Layer** (External Concerns):
```typescript
// infrastructure/repositories/http-plant.repository.ts
@Injectable({ providedIn: 'root' })
export class HttpPlantRepository extends PlantRepository {
  constructor(private api: ApiClientService) {
    super();
  }
  
  getById(id: string): Observable<Plant> {
    return this.api.get<Plant>(`plants/${id}`);
  }
  
  save(plant: Plant): Observable<Plant> {
    return this.api.post<Plant>('plants', plant);
  }
}
```

**Application Layer** (Use Cases):
```typescript
// application/facades/plant.facade.ts
@Injectable({ providedIn: 'root' })
export class PlantFacade {
  constructor(private repository: PlantRepository) {}
  
  loadPlant(id: string) {
    return this.repository.getById(id);
  }
}
```

**Presentation Layer** (UI):
```typescript
// presentation/views/plant-detail/plant-detail.view.ts
@Component({...})
export class PlantDetailViewComponent {
  constructor(private facade: PlantFacade) {}
  
  ngOnInit() {
    const id = this.route.snapshot.params['id'];
    this.facade.loadPlant(id).subscribe();
  }
}
```

---

### 2. Repository Pattern

**Purpose**: Abstract data access logic and provide a collection-like interface for domain objects.

#### Benefits
- Centralized data access logic
- Easy to test (mock repositories)
- Separation of data source from business logic
- Flexibility to change data sources

#### Implementation

**Abstract Repository**:
```typescript
export abstract class PlantRepository {
  abstract search(params: PlantSearchParams): Observable<PagedResult<Plant>>;
  abstract get(id: string): Observable<Plant>;
  abstract create(payload: CreatePlantPayload): Observable<Plant>;
  abstract update(id: string, payload: UpdatePlantPayload): Observable<Plant>;
  abstract delete(id: string): Observable<void>;
  abstract getAlerts(plantId: string): Observable<PlantAlert[]>;
}
```

**Concrete Implementation**:
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

  getAlerts(plantId: string): Observable<PlantAlert[]> {
    return this.api.get<PlantAlert[]>(`plants/${plantId}/alerts`);
  }
}
```

**Dependency Injection**:
```typescript
// app.config.ts
{ provide: PlantRepository, useExisting: HttpPlantRepository }
```

#### Alternative Implementations

You can easily swap implementations without changing business logic:

```typescript
// For offline mode
export class LocalStoragePlantRepository extends PlantRepository {
  get(id: string): Observable<Plant> {
    const plants = JSON.parse(localStorage.getItem('plants') || '[]');
    const plant = plants.find(p => p.id === id);
    return of(plant);
  }
}

// For testing
export class MockPlantRepository extends PlantRepository {
  get(id: string): Observable<Plant> {
    return of({ id, name: 'Test Plant', species: 'Test' });
  }
}
```

---

### 3. Facade Pattern

**Purpose**: Provide a simplified interface to complex subsystems and manage application state.

#### Benefits
- Simplified API for components
- Centralized state management
- Encapsulated business logic
- Easier testing and maintenance

#### Implementation

```typescript
@Injectable({ providedIn: 'root' })
export class PlantFacade {
  // Private state (writable signals)
  private readonly plantsState = signal<PagedResult<Plant> | null>(null);
  private readonly loadingState = signal(false);
  private readonly selectedPlantState = signal<Plant | null>(null);
  private readonly filtersState = signal<PlantSearchParams>({
    page: 0,
    size: 10,
    sort: 'createdAt,desc'
  });

  // Public state (readonly signals)
  readonly plants = computed(() => this.plantsState()?.content ?? []);
  readonly loading = this.loadingState.asReadonly();
  readonly selectedPlant = this.selectedPlantState.asReadonly();
  readonly pageInfo = this.plantsState.asReadonly();
  readonly isEmpty = computed(() => this.plants().length === 0);

  constructor(
    private readonly plantRepository: PlantRepository,
    private readonly notifications: NotificationService
  ) {}

  // Use case: Load plants with filters
  loadPlants(overrides?: PlantSearchParams) {
    const params = { ...this.filtersState(), ...overrides };
    this.filtersState.set(params);
    this.loadingState.set(true);

    this.plantRepository.search(params)
      .pipe(finalize(() => this.loadingState.set(false)))
      .subscribe({
        next: (result) => this.plantsState.set(result),
        error: () => this.notifications.error('Failed to load plants')
      });
  }

  // Use case: Load single plant
  loadPlant(id: string) {
    this.loadingState.set(true);
    
    this.plantRepository.get(id)
      .pipe(finalize(() => this.loadingState.set(false)))
      .subscribe({
        next: (plant) => this.selectedPlantState.set(plant),
        error: () => this.notifications.error('Failed to load plant')
      });
  }

  // Use case: Create plant
  createPlant(payload: CreatePlantPayload) {
    return this.plantRepository.create(payload).pipe(
      tap(() => {
        this.notifications.success('Plant created successfully');
        this.loadPlants(); // Refresh list
      }),
      catchError((error) => {
        this.notifications.error('Failed to create plant');
        return throwError(() => error);
      })
    );
  }

  // Use case: Update plant
  updatePlant(id: string, payload: UpdatePlantPayload) {
    return this.plantRepository.update(id, payload).pipe(
      tap((plant) => {
        this.notifications.success('Plant updated successfully');
        this.selectedPlantState.set(plant);
        this.loadPlants(); // Refresh list
      }),
      catchError((error) => {
        this.notifications.error('Failed to update plant');
        return throwError(() => error);
      })
    );
  }

  // Use case: Delete plant
  deletePlant(id: string) {
    return this.plantRepository.delete(id).pipe(
      tap(() => {
        this.notifications.success('Plant deleted successfully');
        this.loadPlants(); // Refresh list
      }),
      catchError((error) => {
        this.notifications.error('Failed to delete plant');
        return throwError(() => error);
      })
    );
  }

  // Helper: Clear selection
  clearSelection() {
    this.selectedPlantState.set(null);
  }

  // Helper: Update filters
  updateFilters(filters: Partial<PlantSearchParams>) {
    this.filtersState.update(current => ({ ...current, ...filters }));
  }
}
```

**Usage in Components**:
```typescript
@Component({
  template: `
    <div *ngIf="facade.loading()">Loading...</div>
    <ul *ngIf="!facade.isEmpty()">
      <li *ngFor="let plant of facade.plants()">
        {{ plant.name }}
      </li>
    </ul>
  `
})
export class PlantListComponent {
  constructor(public facade: PlantFacade) {
    facade.loadPlants();
  }
}
```

---

### 4. Dependency Injection (DI)

**Purpose**: Inversion of Control for better testability and flexibility.

#### Providers Strategy

**Root-Level Services** (Singletons):
```typescript
@Injectable({ providedIn: 'root' })
export class AuthService { }
```

**Benefits**:
- Single instance across application
- Lazy loaded if not used
- Tree-shakable

**Component-Level Services**:
```typescript
@Component({
  providers: [LocalService]
})
export class SomeComponent {
  constructor(private local: LocalService) {}
}
```

**Benefits**:
- New instance per component
- Isolated state

#### Injection Tokens

**Configuration Token**:
```typescript
export interface PlantaeFrontConfig {
  apiBaseUrl: string;
  defaultLanguage: string;
  supportedLanguages: string[];
}

export const PLANTAE_CONFIG = new InjectionToken<PlantaeFrontConfig>(
  'plantae.config'
);

// Provider
{
  provide: PLANTAE_CONFIG,
  useValue: {
    apiBaseUrl: environment.apiBaseUrl,
    defaultLanguage: 'en',
    supportedLanguages: ['en', 'es']
  }
}

// Usage
constructor(@Inject(PLANTAE_CONFIG) private config: PlantaeFrontConfig) {}
```

#### Interface-Based DI

```typescript
// Abstract class as token
export abstract class PlantRepository {
  abstract get(id: string): Observable<Plant>;
}

// Provider mapping
{ provide: PlantRepository, useExisting: HttpPlantRepository }

// Injection
constructor(private repo: PlantRepository) {}
// Receives HttpPlantRepository instance
```

---

### 5. Interceptor Pattern

**Purpose**: Intercept and modify HTTP requests/responses globally.

#### Functional Interceptor (Angular 15+)

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

**Benefits**:
- Functional approach (no classes)
- Can use `inject()` function
- Composable
- Type-safe

**Registration**:
```typescript
provideHttpClient(withInterceptors([authInterceptor]))
```

#### Use Cases

**Error Handling Interceptor**:
```typescript
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const notifications = inject(NotificationService);
  
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        notifications.error('Session expired. Please login again.');
      } else if (error.status === 500) {
        notifications.error('Server error. Please try again later.');
      }
      return throwError(() => error);
    })
  );
};
```

**Logging Interceptor**:
```typescript
export const loggingInterceptor: HttpInterceptorFn = (req, next) => {
  console.log('HTTP Request:', req.method, req.url);
  
  return next(req).pipe(
    tap({
      next: (event) => {
        if (event.type === HttpEventType.Response) {
          console.log('HTTP Response:', event.status, event.url);
        }
      }
    })
  );
};
```

---

## State Management Patterns

### 1. Signal-Based State

**Purpose**: Reactive state management with fine-grained updates.

#### Signal Types

**Writable Signal** (Private):
```typescript
private readonly stateSignal = signal<State>(initialState);
```

**Readonly Signal** (Public):
```typescript
readonly state = this.stateSignal.asReadonly();
```

**Computed Signal** (Derived):
```typescript
readonly derivedState = computed(() => {
  const state = this.stateSignal();
  return transformState(state);
});
```

#### State Update Patterns

**Set** (Replace):
```typescript
this.stateSignal.set(newState);
```

**Update** (Transform):
```typescript
this.stateSignal.update(current => ({
  ...current,
  property: newValue
}));
```

#### Example: Auth State Management

```typescript
@Injectable({ providedIn: 'root' })
export class AuthService {
  private readonly state = signal<AuthState>({
    token: null,
    accountType: null,
    language: 'en',
    userId: null
  });

  readonly isAuthenticated = computed(() => !!this.state().token);
  readonly accountType = computed(() => this.state().accountType);
  readonly currentUser = computed(() => ({
    id: this.state().userId,
    accountType: this.state().accountType
  }));

  login(credentials: LoginRequest) {
    return this.api.post<AuthResponse>('iam/login', credentials)
      .pipe(
        tap((response) => {
          this.state.update(current => ({
            ...current,
            token: response.token,
            accountType: response.accountType,
            userId: response.userId
          }));
        })
      );
  }

  logout() {
    this.state.set({
      token: null,
      accountType: null,
      language: this.state().language,
      userId: null
    });
  }
}
```

---

### 2. Local Component State

**Purpose**: Manage state within a single component.

#### Pattern

```typescript
@Component({...})
export class FormComponent {
  // Form state
  protected readonly form = this.fb.group({
    name: ['', Validators.required],
    email: ['', [Validators.required, Validators.email]]
  });

  // Loading state
  protected readonly isSubmitting = signal(false);
  
  // Computed validation
  protected readonly isValid = computed(() => this.form.valid);

  submit() {
    if (this.form.invalid) return;
    
    this.isSubmitting.set(true);
    
    this.service.save(this.form.value)
      .pipe(finalize(() => this.isSubmitting.set(false)))
      .subscribe({
        next: () => this.router.navigate(['/success']),
        error: () => this.notifications.error('Save failed')
      });
  }
}
```

---

## Component Design Patterns

### 1. Smart vs. Presentational Components

#### Smart Component (Container)
- Connects to services/facades
- Manages state
- Handles business logic
- Passes data to presentational components

```typescript
@Component({
  selector: 'app-plant-list',
  template: `
    <app-plant-table 
      [plants]="facade.plants()" 
      [loading]="facade.loading()"
      (plantSelected)="onPlantSelected($event)"
      (plantDeleted)="onPlantDeleted($event)">
    </app-plant-table>
  `
})
export class PlantListComponent {
  constructor(public facade: PlantFacade) {
    facade.loadPlants();
  }

  onPlantSelected(plant: Plant) {
    this.router.navigate(['/plants', plant.id]);
  }

  onPlantDeleted(plant: Plant) {
    this.facade.deletePlant(plant.id).subscribe();
  }
}
```

#### Presentational Component (Dumb)
- Receives data via `@Input()`
- Emits events via `@Output()`
- No service dependencies
- Reusable and testable

```typescript
@Component({
  selector: 'app-plant-table',
  template: `
    <table *ngIf="!loading; else loader">
      <tr *ngFor="let plant of plants">
        <td>{{ plant.name }}</td>
        <td>
          <button (click)="plantSelected.emit(plant)">View</button>
          <button (click)="plantDeleted.emit(plant)">Delete</button>
        </td>
      </tr>
    </table>
    <ng-template #loader>Loading...</ng-template>
  `
})
export class PlantTableComponent {
  @Input() plants: Plant[] = [];
  @Input() loading = false;
  @Output() plantSelected = new EventEmitter<Plant>();
  @Output() plantDeleted = new EventEmitter<Plant>();
}
```

---

### 2. Standalone Components

**Purpose**: Self-contained components with explicit dependencies.

#### Benefits
- Better tree-shaking
- Clearer dependencies
- Easier to reuse
- Simplified testing

#### Pattern

```typescript
@Component({
  selector: 'app-component',
  standalone: true,
  imports: [
    CommonModule,           // *ngIf, *ngFor, etc.
    ReactiveFormsModule,    // Forms
    RouterLink,             // Routing
    TranslatePipe,          // Custom pipes
    OtherComponent          // Other components
  ],
  template: `...`,
  styles: [`...`]
})
export class MyComponent {}
```

---

### 3. View Encapsulation

**Purpose**: Component style isolation.

#### Default (Emulated)
```typescript
@Component({
  encapsulation: ViewEncapsulation.Emulated,  // Default
  styles: [`
    .container { padding: 1rem; }
  `]
})
```
- Styles scoped to component via attribute selectors
- No style leaking

#### ShadowDom
```typescript
@Component({
  encapsulation: ViewEncapsulation.ShadowDom,
  styles: [`...`]
})
```
- True Shadow DOM encapsulation
- Complete isolation
- May have browser compatibility issues

#### None
```typescript
@Component({
  encapsulation: ViewEncapsulation.None,
  styles: [`...`]
})
```
- Global styles
- Use for theme/layout components

---

## Form Patterns

### 1. Reactive Forms

**Purpose**: Type-safe, reactive form handling.

#### Basic Pattern

```typescript
@Component({...})
export class LoginComponent {
  protected readonly form = this.fb.group({
    email: ['', [Validators.required, Validators.email]],
    password: ['', [Validators.required, Validators.minLength(6)]]
  });

  constructor(private fb: FormBuilder) {}

  submit() {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }

    const { email, password } = this.form.value;
    this.authService.login({ email, password }).subscribe();
  }

  get email() {
    return this.form.get('email');
  }

  get password() {
    return this.form.get('password');
  }
}
```

#### Template

```html
<form [formGroup]="form" (ngSubmit)="submit()">
  <label>
    <span>Email</span>
    <input formControlName="email" type="email" />
    <span *ngIf="email?.invalid && email?.touched" class="error">
      Invalid email
    </span>
  </label>

  <label>
    <span>Password</span>
    <input formControlName="password" type="password" />
    <span *ngIf="password?.invalid && password?.touched" class="error">
      Minimum 6 characters
    </span>
  </label>

  <button type="submit" [disabled]="form.invalid">Submit</button>
</form>
```

---

### 2. Custom Validators

#### Synchronous Validator

```typescript
export function passwordMatchValidator(
  passwordKey: string,
  confirmPasswordKey: string
): ValidatorFn {
  return (group: AbstractControl): ValidationErrors | null => {
    const password = group.get(passwordKey);
    const confirmPassword = group.get(confirmPasswordKey);

    if (!password || !confirmPassword) {
      return null;
    }

    return password.value === confirmPassword.value 
      ? null 
      : { passwordMismatch: true };
  };
}

// Usage
this.form = this.fb.group({
  password: ['', Validators.required],
  confirmPassword: ['', Validators.required]
}, {
  validators: passwordMatchValidator('password', 'confirmPassword')
});
```

#### Asynchronous Validator

```typescript
export function emailExistsValidator(
  authService: AuthService
): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value) {
      return of(null);
    }

    return authService.checkEmailExists(control.value).pipe(
      map(exists => exists ? { emailExists: true } : null),
      catchError(() => of(null))
    );
  };
}

// Usage
this.form = this.fb.group({
  email: ['', 
    [Validators.required, Validators.email],
    [emailExistsValidator(this.authService)]
  ]
});
```

---

## Routing Patterns

### 1. Guard Composition

**Purpose**: Combine multiple guards for complex authorization.

```typescript
// Route configuration
{
  path: 'nursery',
  component: NurseryDashboardComponent,
  canActivate: [authGuard, accountTypeGuard],
  data: { accountTypes: ['VIVERO_FORESTAL'] }
}
```

**Execution**: Guards execute in order. First failure stops execution.

---

### 2. Route Data

**Purpose**: Pass static data to routes.

```typescript
{
  path: 'plants',
  component: PlantListComponent,
  data: {
    title: 'Plant Management',
    breadcrumb: 'Plants',
    requiresPro: false
  }
}

// In component
constructor(private route: ActivatedRoute) {
  const title = this.route.snapshot.data['title'];
}
```

---

### 3. Lazy Loading

**Purpose**: Load features on demand to reduce initial bundle size.

```typescript
{
  path: 'admin',
  loadComponent: () => import('./admin/admin.component')
    .then(m => m.AdminComponent),
  canActivate: [authGuard, adminGuard]
}
```

**Benefits**:
- Smaller initial bundle
- Faster app startup
- Load features when needed

---

## Error Handling Patterns

### 1. Global Error Handler

```typescript
@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  constructor(private notifications: NotificationService) {}

  handleError(error: Error) {
    console.error('Global error:', error);
    this.notifications.error('An unexpected error occurred');
  }
}

// Provider
{ provide: ErrorHandler, useClass: GlobalErrorHandler }
```

---

### 2. HTTP Error Handling

**Centralized in Services**:
```typescript
login(credentials: LoginRequest) {
  return this.api.post<AuthResponse>('iam/login', credentials)
    .pipe(
      catchError((error: HttpErrorResponse) => {
        const message = error.error?.message || 'Login failed';
        this.notifications.error(message);
        return throwError(() => error);
      })
    );
}
```

---

## Best Practices

### 1. Component Communication

**Parent → Child**: Use `@Input()`
**Child → Parent**: Use `@Output()` and `EventEmitter`
**Unrelated Components**: Use service with signals

### 2. Memory Management

**Automatic Cleanup** (Signals):
```typescript
// No manual unsubscription needed
readonly data = computed(() => this.service.data());
```

**Manual Cleanup** (Observables):
```typescript
private destroy$ = new Subject<void>();

ngOnInit() {
  this.observable$
    .pipe(takeUntil(this.destroy$))
    .subscribe();
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}
```

### 3. Type Safety

**Strict typing**:
```typescript
// Avoid 'any'
function process(data: unknown) {
  if (typeof data === 'string') {
    return data.toUpperCase();
  }
}

// Use generics
function identity<T>(value: T): T {
  return value;
}
```

### 4. Immutability

**State updates**:
```typescript
// Bad
this.state.items.push(newItem);

// Good
this.state = {
  ...this.state,
  items: [...this.state.items, newItem]
};

// With signals
this.stateSignal.update(current => ({
  ...current,
  items: [...current.items, newItem]
}));
```

---

## Related Documentation
- [Component Documentation](./COMPONENT_DOCUMENTATION.md)
- [Module Documentation](./MODULE_DOCUMENTATION.md)
- [Routing Guide](./ROUTING_GUIDE.md)


