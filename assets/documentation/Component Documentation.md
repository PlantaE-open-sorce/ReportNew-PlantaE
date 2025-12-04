# Component Documentation

## Overview

This document provides detailed specifications for all components in the PlantaE Frontend Application, including their props, events, and usage examples.

---

## Shared Components

### 1. LayoutComponent

**Location**: `src/app/shared/presentation/components/layout/layout.component.ts`

**Purpose**: Main layout wrapper that provides the application structure with sidebar navigation, header, and footer.

#### Features
- Responsive sidebar navigation with mobile support
- Dynamic menu items based on user account type
- Language switcher integration
- Toast notifications container
- Authentication-aware menu items

#### Template Structure
```html
<div class="app-layout">
  <aside class="sidebar">
    <!-- Navigation menu -->
  </aside>
  <main>
    <router-outlet></router-outlet>
  </main>
  <footer>
    <app-footer-content></app-footer-content>
  </footer>
</div>
<app-toast-container></app-toast-container>
```

#### Dependencies
- `RouterOutlet`, `RouterLink`, `RouterLinkActive` for navigation
- `FooterContentComponent` for footer
- `LanguageSwitcherComponent` for language selection
- `ToastContainerComponent` for notifications
- `I18nService` for internationalization
- `AuthService` for authentication state
- `AlertFacade` for alert badge count

#### Sidebar Configuration

**SidebarLink Interface**:
```typescript
interface SidebarLink {
  path: string;              // Route path
  labelKey: string;          // i18n translation key
  fallback: string;          // Fallback text
  icon: string;              // Icon identifier
  exact?: boolean;           // Exact route matching
  requiresPro?: boolean;     // Pro feature flag
  hasBadge?: boolean;        // Show badge (e.g., alerts count)
  allowedAccountTypes?: string[];  // Restrict by account type
}
```

#### Account Types
- `HOME`: Standard user account
- `VIVERO_FORESTAL`: Nursery grower account

#### Icons
The component includes built-in SVG icon definitions for:
- `home`: Home dashboard
- `dashboard`: Dashboard view
- `plant`: Plant management
- `sensor`: Sensor monitoring
- `management`: Plant management
- `devices`: Device configuration
- `report`: Reports view
- `alert`: Alerts notification
- `settings`: Settings page
- `user`: User profile

#### Usage Example
```typescript
// Automatically used in app.routes.ts as the parent layout
{
  path: '',
  component: LayoutComponent,
  children: [
    { path: 'dashboard', component: DashboardViewComponent }
  ]
}
```

---

### 2. ToastContainerComponent

**Location**: `src/app/shared/presentation/components/toast-container/toast-container.component.ts`

**Purpose**: Displays toast notifications (success, error, info) in a fixed position on screen.

#### Props
None (uses service injection)

#### Consumed Services
- `NotificationService`: Provides reactive toast messages

#### Template Structure
```html
<section class="toast-container" *ngIf="notifications.messages().length">
  <article *ngFor="let toast of notifications.messages()" 
           [class]="'toast toast--' + toast.type">
    <span>{{ toast.text }}</span>
    <button type="button" (click)="dismiss(toast.id)">×</button>
  </article>
</section>
```

#### Toast Types
- `success`: Green background (#1e8a4b)
- `error`: Red background (#b02a37)
- `info`: Blue background (#0d6efd)

#### Methods

**dismiss(id: number)**
- Removes a toast message by ID
- Called when user clicks the close button

#### Styling Features
- Fixed position at top-right (desktop)
- Full width on mobile (<640px)
- Auto-dismiss after 5 seconds
- Smooth transitions
- Z-index: 1000 for overlay

#### Usage Example
```typescript
// Inject NotificationService in any component
constructor(private notifications: NotificationService) {}

// Show notifications
this.notifications.success('Plant saved successfully!');
this.notifications.error('Failed to load data');
this.notifications.info('Processing request...');
```

---

### 3. LanguageSwitcherComponent

**Location**: `src/app/shared/presentation/components/language-switcher/language-switcher.component.ts`

**Purpose**: Toggle between supported languages (English/Spanish) with animated UI.

#### Props
None (uses service injection)

#### Events
- Language change triggers authentication service to update user preferences

#### Dependencies
- `I18nService`: Manages translations and language state
- `AuthService`: Persists language preference

#### Template Structure
```html
<div class="language-switcher" role="group" 
     [attr.aria-label]="'common:aria.languageSwitcher' | t:'Selector de idioma'">
  <div class="language-switcher__track">
    <span class="language-switcher__thumb" 
          [style.left]="thumbLeft()" 
          [style.width]="thumbWidth()"></span>
    <button *ngFor="let lang of languages" 
            (click)="onLanguageChange(lang)"
            [class.active]="isActive(lang)"
            [attr.aria-pressed]="isActive(lang)">
      {{ lang | uppercase }}
    </button>
  </div>
</div>
```

#### Computed Properties

**languages**: Array of supported language codes
**currentLang**: Signal containing active language

#### Methods

**onLanguageChange(language: string)**
- Updates user language preference
- Persists to backend via AuthService
- Triggers re-translation of UI

**isActive(language: string)**
- Returns true if language is currently selected

**thumbWidth()**
- Calculates animated thumb width based on number of languages

**thumbLeft()**
- Calculates animated thumb position

#### Styling Features
- Pill-shaped toggle with animated thumb
- Active state with color change
- Smooth transitions (0.2s ease)
- Responsive font sizing

#### Usage Example
```html
<!-- Typically used in LayoutComponent header -->
<header>
  <app-language-switcher></app-language-switcher>
</header>
```

---

### 4. FooterContentComponent

**Location**: `src/app/shared/presentation/components/footer-content/footer-content.component.ts`

**Purpose**: Application footer with copyright and links.

#### Props
None

#### Template Structure
Displays copyright information and relevant links.

#### Styling
Minimal footer design aligned with application theme.

---

## Pipes

### TranslatePipe

**Location**: `src/app/shared/presentation/pipes/translate.pipe.ts`

**Name**: `t`

**Purpose**: Translates keys using the I18nService with optional fallback.

#### Transform Signature
```typescript
transform(key: string, fallback?: string): string
```

#### Parameters
- `key`: Translation key in format `namespace:key` (e.g., `iam:login.title`)
- `fallback`: Optional fallback text if translation not found

#### Usage Examples
```html
<!-- Basic usage -->
<h1>{{ 'iam:login.title' | t }}</h1>

<!-- With fallback -->
<p>{{ 'iam:login.subtitle' | t:'Enter your credentials' }}</p>

<!-- In attributes -->
<button [attr.aria-label]="'common:aria.languageSwitcher' | t:'Language selector'">
```

#### Behavior
- **Pure**: `false` (re-evaluates when language changes)
- **Standalone**: `true`
- Automatically loads namespace if not already cached
- Returns fallback or key if translation missing

---

## View Components (Page-Level)

### 1. LoginViewComponent

**Location**: `src/app/plant-care/presentation/views/login/login.view.ts`

**Purpose**: User authentication page with email/password login.

#### Form Structure
```typescript
interface LoginForm {
  email: string;      // Email validator applied
  password: string;   // Required
}
```

#### Template Sections
1. **Auth Panel**: Left side with form
2. **Auth Hero**: Right side with quote and statistics

#### Hero Statistics
```typescript
heroStats = [
  { value: '500+', label: 'iam:login.stats.devices', fallback: 'Active devices' },
  { value: '98%', label: 'iam:login.stats.success', fallback: 'Alerts resolved' },
  { value: '24/7', label: 'iam:login.stats.monitoring', fallback: 'Continuous monitoring' }
];
```

#### Methods

**submit()**
- Validates form
- Calls AuthService.login()
- Redirects to appropriate dashboard on success

#### Routing
- Success redirects to user's default route based on account type
- Links to `/register` and `/forgot-password`

#### Validation
- Email: Required, valid email format
- Password: Required
- Submit button disabled until form valid

---

### 2. RegisterViewComponent

**Location**: `src/app/plant-care/presentation/views/register/register.view.ts`

**Purpose**: New user registration with account type selection.

#### Form Structure
```typescript
interface RegisterForm {
  displayName: string;
  email: string;
  password: string;
  confirmPassword: string;
  language: string;
  accountType: string;  // 'HOME' | 'VIVERO_FORESTAL'
}
```

#### Account Types
- **HOME**: Standard user for personal plant management
- **VIVERO_FORESTAL**: Nursery grower with additional features

#### Validation Rules
- Display name: Required
- Email: Required, valid email format
- Password: Required, minimum length
- Confirm Password: Must match password
- Language: Required selection
- Account Type: Required selection

#### Methods

**submit()**
- Validates passwords match
- Calls AuthService.register()
- Redirects to dashboard on success

---

### 3. DashboardViewComponent

**Location**: `src/app/plant-care/presentation/views/dashboard/dashboard.view.ts`

**Purpose**: Main dashboard for HOME account type users.

#### Guards
- `authGuard`: Must be authenticated
- `accountTypeGuard`: Only HOME account type

#### Features
- Recent plants overview
- Alert summary
- Sensor readings visualization
- Quick actions

---

### 4. PlantListViewComponent

**Location**: `src/app/plant-care/presentation/views/plant-list/plant-list.view.ts`

**Purpose**: Displays paginated list of user's plants.

#### Features
- Search and filter capabilities
- Pagination controls
- Plant status indicators
- Quick actions (edit, delete, view details)

#### Dependencies
- `PlantFacade`: Manages plant state and operations

---

### 5. PlantDetailViewComponent

**Location**: `src/app/plant-care/presentation/views/plant-detail/plant-detail.view.ts`

**Purpose**: Detailed view of a single plant with sensor data and alerts.

#### Route Parameters
- `id`: Plant identifier

#### Features
- Plant information display
- Associated sensor readings
- Alert history
- Device connection status

---

## Form Validation Patterns

### Common Validators

```typescript
// Email validation
Validators.required
Validators.email

// Password validation
Validators.required
Validators.minLength(6)

// Custom validators
passwordMatchValidator // Ensures password confirmation matches
```

### Error Handling Pattern

```typescript
// In component
this.form.invalid  // Disables submit button

// On submit
if (this.form.invalid) {
  return;
}

// Service call with error handling
this.authService.login(payload)
  .subscribe({
    next: (response) => {
      // Success notification shown by service
      this.router.navigate([this.getDefaultRoute()]);
    },
    error: (error) => {
      // Error notification shown by interceptor/service
    }
  });
```

---

## Accessibility Features

### ARIA Labels
All interactive components include proper ARIA labels:

```html
<!-- Language Switcher -->
<div role="group" [attr.aria-label]="'common:aria.languageSwitcher' | t">

<!-- Buttons with pressed state -->
<button [attr.aria-pressed]="isActive(lang)">

<!-- Navigation -->
<nav aria-label="Main navigation">
```

### Keyboard Navigation
- All interactive elements are keyboard accessible
- Focus states clearly visible
- Logical tab order maintained

### Screen Reader Support
- Semantic HTML elements used
- Proper heading hierarchy
- Form labels properly associated

---

## Testing Examples

### Component Unit Test

```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { ToastContainerComponent } from './toast-container.component';
import { NotificationService } from '../../../infrastructure/services/notification.service';

describe('ToastContainerComponent', () => {
  let component: ToastContainerComponent;
  let fixture: ComponentFixture<ToastContainerComponent>;
  let notificationService: NotificationService;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [ToastContainerComponent]
    }).compileComponents();

    fixture = TestBed.createComponent(ToastContainerComponent);
    component = fixture.componentInstance;
    notificationService = TestBed.inject(NotificationService);
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should display toast messages', () => {
    notificationService.success('Test message');
    fixture.detectChanges();
    
    const toastElement = fixture.nativeElement.querySelector('.toast--success');
    expect(toastElement).toBeTruthy();
    expect(toastElement.textContent).toContain('Test message');
  });

  it('should dismiss toast on button click', () => {
    notificationService.success('Test message');
    fixture.detectChanges();
    
    const dismissButton = fixture.nativeElement.querySelector('button');
    dismissButton.click();
    fixture.detectChanges();
    
    expect(fixture.nativeElement.querySelector('.toast')).toBeFalsy();
  });
});
```

### Pipe Unit Test

```typescript
import { TestBed } from '@angular/core/testing';
import { TranslatePipe } from './translate.pipe';
import { I18nService } from '../../infrastructure/services/i18n.service';

describe('TranslatePipe', () => {
  let pipe: TranslatePipe;
  let i18nService: jasmine.SpyObj<I18nService>;

  beforeEach(() => {
    const spy = jasmine.createSpyObj('I18nService', ['translate']);
    
    TestBed.configureTestingModule({
      providers: [
        TranslatePipe,
        { provide: I18nService, useValue: spy }
      ]
    });
    
    pipe = TestBed.inject(TranslatePipe);
    i18nService = TestBed.inject(I18nService) as jasmine.SpyObj<I18nService>;
  });

  it('should translate key', () => {
    i18nService.translate.and.returnValue('Translated text');
    
    const result = pipe.transform('iam:login.title', 'Fallback');
    
    expect(result).toBe('Translated text');
    expect(i18nService.translate).toHaveBeenCalledWith('iam:login.title', 'Fallback');
  });
});
```

### Integration Test

```typescript
import { TestBed } from '@angular/core/testing';
import { RouterTestingModule } from '@angular/router/testing';
import { LoginViewComponent } from './login.view';
import { AuthService } from '../../../../shared/infrastructure/services/auth.service';
import { of, throwError } from 'rxjs';

describe('LoginViewComponent Integration', () => {
  let component: LoginViewComponent;
  let authService: jasmine.SpyObj<AuthService>;

  beforeEach(async () => {
    const authSpy = jasmine.createSpyObj('AuthService', ['login']);

    await TestBed.configureTestingModule({
      imports: [LoginViewComponent, RouterTestingModule],
      providers: [{ provide: AuthService, useValue: authSpy }]
    }).compileComponents();

    authService = TestBed.inject(AuthService) as jasmine.SpyObj<AuthService>;
  });

  it('should login successfully', () => {
    const mockResponse = { token: 'test-token', accountType: 'HOME', message: 'Success' };
    authService.login.and.returnValue(of(mockResponse));

    component.form.patchValue({
      email: 'test@example.com',
      password: 'password123'
    });

    component.submit();

    expect(authService.login).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123'
    });
  });
});
```

---

## Component Lifecycle Patterns

### OnInit Pattern
```typescript
export class DashboardViewComponent implements OnInit {
  ngOnInit() {
    // Load initial data
    this.homeFacade.loadSummary();
    this.plantFacade.loadPlants({ page: 0, size: 5 });
    
    // Subscribe to route params if needed
    this.route.params.subscribe(params => {
      // React to route changes
    });
  }
}
```

### OnDestroy Pattern
```typescript
export class SomeComponent implements OnDestroy {
  private destroy$ = new Subject<void>();

  ngOnInit() {
    this.someObservable$
      .pipe(takeUntil(this.destroy$))
      .subscribe(data => {
        // Handle data
      });
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

---

## Performance Optimization

### Signal-Based State
Components use Angular Signals for reactive state management:

```typescript
// In facade
private readonly loadingState = signal(false);
readonly loading: Signal<boolean> = this.loadingState.asReadonly();

// In component
constructor(public facade: PlantFacade) {}

// In template
<div *ngIf="facade.loading()">Loading...</div>
```

### Standalone Components
All components are standalone, reducing bundle size and improving tree-shaking:

```typescript
@Component({
  selector: 'app-component',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule, TranslatePipe],
  // ...
})
```

### OnPush Change Detection
For optimal performance, components should use OnPush strategy where appropriate:

```typescript
@Component({
  // ...
  changeDetection: ChangeDetectionStrategy.OnPush
})
```

---

## Related Documentation
- [Module Documentation](./MODULE_DOCUMENTATION.md)
- [Design Patterns](./DESIGN_PATTERNS.md)
- [Routing Guide](./ROUTING_GUIDE.md)


