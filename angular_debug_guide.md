# Angular Debugging Techniques in Chrome DevTools

A comprehensive guide to debugging Angular applications using Chrome DevTools and Angular-specific tools.

---

## Table of Contents

1. [Angular DevTools Extension](#1-angular-devtools-extension)
2. [Component Debugging](#2-component-debugging)
3. [Service & Dependency Injection Debugging](#3-service--dependency-injection-debugging)
4. [RxJS & Observables Debugging](#4-rxjs--observables-debugging)
5. [Change Detection Debugging](#5-change-detection-debugging)
6. [Router Debugging](#6-router-debugging)
7. [Forms Debugging](#7-forms-debugging)
8. [HTTP & API Debugging](#8-http--api-debugging)
9. [Performance Optimization](#9-performance-optimization)
10. [Common Angular Issues](#10-common-angular-issues)

---

## 1. Angular DevTools Extension

### 1.1 Installation

**Chrome Web Store:**
- Search for "Angular DevTools"
- Install the official extension by Angular team

**Access:**
- Open Chrome DevTools (F12)
- Look for "Angular" tab

### 1.2 Component Explorer

**Features:**
- Component tree visualization
- Property inspection
- Directive information
- Change detection status

```typescript
// Example component to inspect
@Component({
  selector: 'app-user-profile',
  template: `
    <div class="profile">
      <h2>{{ user.name }}</h2>
      <p>{{ user.email }}</p>
      <button (click)="updateUser()">Update</button>
    </div>
  `
})
export class UserProfileComponent {
  user = {
    name: 'John Doe',
    email: 'john@example.com',
    age: 30
  };

  updateUser() {
    this.user.name = 'Jane Doe';
  }
}
```

**In Angular DevTools:**
1. Select component in tree
2. View properties in right panel
3. Modify values in real-time
4. See change detection cycles

### 1.3 Profiler

**Purpose:** Measure component render performance

**How to use:**
1. Angular DevTools → Profiler tab
2. Click Record (●)
3. Interact with app
4. Stop recording
5. Analyze component render times

```typescript
// Component to profile
@Component({
  selector: 'app-product-list',
  template: `
    <div *ngFor="let product of products">
      {{ product.name }} - {{ product.price | currency }}
    </div>
  `
})
export class ProductListComponent implements OnInit {
  products: Product[] = [];

  ngOnInit() {
    console.time('Load Products');
    this.loadProducts();
    console.timeEnd('Load Products');
  }

  loadProducts() {
    // Simulate data loading
    this.products = Array.from({ length: 1000 }, (_, i) => ({
      id: i,
      name: `Product ${i}`,
      price: Math.random() * 100
    }));
  }
}
```

---

## 2. Component Debugging

### 2.1 Component Lifecycle Debugging

```typescript
import { Component, OnInit, OnDestroy, OnChanges, 
         DoCheck, AfterContentInit, AfterContentChecked,
         AfterViewInit, AfterViewChecked } from '@angular/core';

@Component({
  selector: 'app-lifecycle-demo',
  template: `<h1>{{ title }}</h1>`
})
export class LifecycleDemoComponent implements OnInit, OnDestroy, 
  OnChanges, DoCheck, AfterContentInit, AfterContentChecked,
  AfterViewInit, AfterViewChecked {
  
  title = 'Lifecycle Demo';

  constructor() {
    console.log('1. Constructor called');
  }

  ngOnChanges(changes: SimpleChanges) {
    console.log('2. ngOnChanges called', changes);
  }

  ngOnInit() {
    console.log('3. ngOnInit called');
    debugger; // Pause here to inspect component state
  }

  ngDoCheck() {
    console.log('4. ngDoCheck called');
  }

  ngAfterContentInit() {
    console.log('5. ngAfterContentInit called');
  }

  ngAfterContentChecked() {
    console.log('6. ngAfterContentChecked called');
  }

  ngAfterViewInit() {
    console.log('7. ngAfterViewInit called');
  }

  ngAfterViewChecked() {
    console.log('8. ngAfterViewChecked called');
  }

  ngOnDestroy() {
    console.log('9. ngOnDestroy called');
  }
}
```

### 2.2 Access Component from Console

```typescript
// Component
@Component({
  selector: 'app-counter',
  template: `
    <div>
      <h2>Count: {{ count }}</h2>
      <button (click)="increment()">+</button>
    </div>
  `
})
export class CounterComponent {
  count = 0;

  increment() {
    this.count++;
  }
}
```

**In Chrome Console:**

```javascript
// Method 1: Using ng object (Angular 9+)
const component = ng.getComponent($0); // $0 = selected element
component.count = 100;
ng.applyChanges($0); // Trigger change detection

// Method 2: Using Angular DevTools
// Select component in Elements tab, then:
$0.__ngContext__; // Access component context

// Method 3: Get all components
ng.getComponents($0); // Get all components on element

// Get component directives
ng.getDirectives($0);

// Get component host element
ng.getHostElement(component);

// Get component owners
ng.getOwningComponent($0);
```

### 2.3 Template Debugging

```typescript
@Component({
  selector: 'app-debug-template',
  template: `
    <div>
      <!-- Debug pipe - shows object in JSON -->
      <pre>{{ user | json }}</pre>
      
      <!-- Conditional debugging -->
      <div *ngIf="isDebugMode">
        Debug Info: {{ debugInfo | json }}
      </div>
      
      <!-- Template reference variable debugging -->
      <input #myInput (keyup)="logInput(myInput.value)">
      
      <!-- ngTemplateOutlet for debugging -->
      <ng-template #debugTemplate let-data>
        <pre>{{ data | json }}</pre>
      </ng-template>
      
      <ng-container *ngTemplateOutlet="debugTemplate; context: { $implicit: user }">
      </ng-container>
    </div>
  `
})
export class DebugTemplateComponent {
  user = { name: 'John', age: 30 };
  isDebugMode = true;
  debugInfo = { timestamp: Date.now() };

  logInput(value: string) {
    console.log('Input value:', value);
  }
}
```

### 2.4 ViewChild & ViewChildren Debugging

```typescript
@Component({
  selector: 'app-view-debug',
  template: `
    <child-component #child1></child-component>
    <child-component #child2></child-component>
    <div #myDiv>Content</div>
  `
})
export class ViewDebugComponent implements AfterViewInit {
  @ViewChild('child1') child1!: ChildComponent;
  @ViewChild('myDiv') myDiv!: ElementRef;
  @ViewChildren(ChildComponent) children!: QueryList<ChildComponent>;

  ngAfterViewInit() {
    // Debug ViewChild
    console.log('Child1 component:', this.child1);
    console.log('Div element:', this.myDiv.nativeElement);
    
    // Debug ViewChildren
    console.log('All children:', this.children.toArray());
    
    debugger; // Inspect ViewChild/ViewChildren here
    
    // Watch for changes
    this.children.changes.subscribe(list => {
      console.log('Children changed:', list.toArray());
    });
  }
}
```

### 2.5 Input/Output Debugging

```typescript
// Parent Component
@Component({
  selector: 'app-parent',
  template: `
    <app-child 
      [inputData]="parentData"
      (outputEvent)="handleOutput($event)">
    </app-child>
  `
})
export class ParentComponent {
  parentData = { message: 'Hello from parent' };

  handleOutput(event: any) {
    console.log('Output received:', event);
    debugger; // Debug event handling
  }
}

// Child Component
@Component({
  selector: 'app-child',
  template: `
    <button (click)="sendOutput()">Send to Parent</button>
  `
})
export class ChildComponent implements OnChanges {
  @Input() inputData: any;
  @Output() outputEvent = new EventEmitter<any>();

  ngOnChanges(changes: SimpleChanges) {
    console.log('Input changed:', changes);
    
    if (changes['inputData']) {
      console.log('Previous:', changes['inputData'].previousValue);
      console.log('Current:', changes['inputData'].currentValue);
      debugger; // Debug input changes
    }
  }

  sendOutput() {
    const data = { timestamp: Date.now() };
    console.log('Emitting:', data);
    this.outputEvent.emit(data);
  }
}
```

---

## 3. Service & Dependency Injection Debugging

### 3.1 Service Debugging

```typescript
// Service with debugging
@Injectable({
  providedIn: 'root'
})
export class UserService {
  private users: User[] = [];
  private debugMode = true;

  constructor(private http: HttpClient) {
    this.debug('UserService instantiated');
  }

  getUsers(): Observable<User[]> {
    this.debug('getUsers called');
    debugger; // Pause on service method call
    
    return this.http.get<User[]>('/api/users').pipe(
      tap(users => this.debug('Users received', users)),
      catchError(error => {
        this.debug('Error fetching users', error);
        return throwError(() => error);
      })
    );
  }

  private debug(message: string, data?: any) {
    if (this.debugMode) {
      console.group(`[UserService] ${message}`);
      if (data) console.log(data);
      console.trace(); // Show call stack
      console.groupEnd();
    }
  }
}
```

### 3.2 Dependency Injection Debugging

```typescript
// Check what's injected
@Component({
  selector: 'app-di-debug'
})
export class DIDebugComponent implements OnInit {
  constructor(
    private userService: UserService,
    private router: Router,
    private injector: Injector
  ) {
    console.log('Injected services:', {
      userService: this.userService,
      router: this.router
    });
  }

  ngOnInit() {
    // Get service dynamically
    const service = this.injector.get(UserService);
    console.log('Dynamically injected:', service);
    
    // Check if service exists
    try {
      const optional = this.injector.get(OptionalService);
      console.log('Optional service exists:', optional);
    } catch (error) {
      console.log('Optional service not found');
    }
  }
}
```

### 3.3 Provider Debugging

```typescript
// Service with provider debugging
@Injectable()
export class LoggerService {
  constructor() {
    console.log('LoggerService instance created', this);
  }

  log(message: string) {
    console.log(`[Logger]: ${message}`);
  }
}

// Module with provider
@NgModule({
  providers: [
    LoggerService,
    // Debug factory provider
    {
      provide: 'CONFIG',
      useFactory: () => {
        const config = { apiUrl: '/api' };
        console.log('Factory provider created:', config);
        debugger; // Debug factory creation
        return config;
      }
    },
    // Debug class provider
    {
      provide: UserService,
      useClass: MockUserService,
      // Add this in component to verify:
      // constructor(@Inject(UserService) service: any) {
      //   console.log('Injected instance:', service);
      // }
    }
  ]
})
export class AppModule { }
```

---

## 4. RxJS & Observables Debugging

### 4.1 Basic Observable Debugging

```typescript
import { tap, catchError, finalize } from 'rxjs/operators';

@Component({
  selector: 'app-observable-debug'
})
export class ObservableDebugComponent implements OnInit {
  
  constructor(private http: HttpClient) {}

  ngOnInit() {
    this.http.get<User[]>('/api/users').pipe(
      tap(data => {
        console.log('Data received:', data);
        debugger; // Pause when data arrives
      }),
      tap({
        next: data => console.log('Next:', data),
        error: err => console.error('Error:', err),
        complete: () => console.log('Complete')
      }),
      catchError(error => {
        console.error('Caught error:', error);
        debugger; // Pause on error
        return throwError(() => error);
      }),
      finalize(() => {
        console.log('Observable finalized');
      })
    ).subscribe({
      next: data => console.log('Subscribe next:', data),
      error: err => console.error('Subscribe error:', err),
      complete: () => console.log('Subscribe complete')
    });
  }
}
```

### 4.2 Custom Debug Operator

```typescript
// Custom debug operator
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

export function debug(tag: string) {
  return <T>(source: Observable<T>): Observable<T> => {
    return source.pipe(
      tap({
        next: value => {
          console.group(`[${tag}] Next`);
          console.log(value);
          console.trace();
          console.groupEnd();
        },
        error: error => {
          console.group(`[${tag}] Error`);
          console.error(error);
          console.trace();
          console.groupEnd();
        },
        complete: () => {
          console.log(`[${tag}] Complete`);
        }
      })
    );
  };
}

// Usage
this.http.get<User[]>('/api/users').pipe(
  debug('User API Call'),
  map(users => users.filter(u => u.active)),
  debug('After Filter')
).subscribe();
```

### 4.3 Subject Debugging

```typescript
@Injectable({
  providedIn: 'root'
})
export class StateService {
  private stateSubject = new BehaviorSubject<AppState>(initialState);
  state$ = this.stateSubject.asObservable();

  constructor() {
    // Debug all state changes
    this.state$.subscribe(state => {
      console.log('State changed:', state);
      console.trace('Changed from:');
    });
  }

  updateState(newState: Partial<AppState>) {
    const currentState = this.stateSubject.value;
    const updatedState = { ...currentState, ...newState };
    
    console.group('State Update');
    console.log('Previous:', currentState);
    console.log('New:', updatedState);
    console.groupEnd();
    
    debugger; // Pause on state update
    
    this.stateSubject.next(updatedState);
  }
}
```

### 4.4 Subscription Leak Detection

```typescript
@Component({
  selector: 'app-subscription-debug'
})
export class SubscriptionDebugComponent implements OnInit, OnDestroy {
  private subscriptions = new Subscription();
  private subscriptionCount = 0;

  ngOnInit() {
    // Track subscriptions
    const sub1 = interval(1000).subscribe(() => {
      this.subscriptionCount++;
      console.log('Subscription 1 tick');
    });

    const sub2 = this.userService.getUsers().subscribe(users => {
      console.log('Subscription 2:', users);
    });

    // Add to subscription container
    this.subscriptions.add(sub1);
    this.subscriptions.add(sub2);

    console.log('Active subscriptions:', this.subscriptionCount);
  }

  ngOnDestroy() {
    console.log('Unsubscribing from all subscriptions');
    this.subscriptions.unsubscribe();
    
    // Check if properly unsubscribed
    console.log('Subscriptions closed:', this.subscriptions.closed);
    
    if (!this.subscriptions.closed) {
      console.error('⚠️ Memory leak: Subscriptions not closed!');
      debugger;
    }
  }
}

// Alternative: Use takeUntil pattern
export class BetterSubscriptionComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();

  ngOnInit() {
    interval(1000).pipe(
      takeUntil(this.destroy$),
      tap(() => console.log('This will auto-unsubscribe'))
    ).subscribe();
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
    console.log('Destroyed successfully');
  }
}
```

---

## 5. Change Detection Debugging

### 5.1 Manual Change Detection Debugging

```typescript
import { ChangeDetectorRef, ApplicationRef } from '@angular/core';

@Component({
  selector: 'app-change-detection-debug',
  template: `
    <div>
      <h2>{{ title }}</h2>
      <button (click)="update()">Update</button>
    </div>
  `
})
export class ChangeDetectionDebugComponent {
  title = 'Initial';

  constructor(
    private cdr: ChangeDetectorRef,
    private appRef: ApplicationRef
  ) {
    // Monitor change detection
    this.appRef.isStable.subscribe(stable => {
      console.log('App stable:', stable);
    });
  }

  update() {
    this.title = 'Updated';
    
    console.log('Before detectChanges');
    debugger;
    
    // Manually trigger change detection
    this.cdr.detectChanges();
    
    console.log('After detectChanges');
  }

  // Detach from change detection
  detach() {
    console.log('Detaching from change detection');
    this.cdr.detach();
  }

  // Reattach to change detection
  reattach() {
    console.log('Reattaching to change detection');
    this.cdr.reattach();
  }

  // Mark for check (with OnPush)
  markForCheck() {
    console.log('Marking for check');
    this.cdr.markForCheck();
  }
}
```

### 5.2 OnPush Strategy Debugging

```typescript
@Component({
  selector: 'app-onpush-debug',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div>
      <h2>{{ data.title }}</h2>
      <button (click)="mutate()">Mutate (Won't work)</button>
      <button (click)="immutableUpdate()">Immutable Update (Works)</button>
    </div>
  `
})
export class OnPushDebugComponent {
  data = { title: 'Initial', count: 0 };

  constructor(private cdr: ChangeDetectorRef) {}

  // BAD: Mutation won't trigger change detection with OnPush
  mutate() {
    console.log('Mutating object');
    this.data.title = 'Mutated';
    this.data.count++;
    
    console.log('Changed data:', this.data);
    console.log('⚠️ View not updated - OnPush requires new reference');
    debugger;
  }

  // GOOD: Create new object reference
  immutableUpdate() {
    console.log('Creating new object reference');
    this.data = {
      ...this.data,
      title: 'Updated',
      count: this.data.count + 1
    };
    
    console.log('✓ View will update - new reference created');
    debugger;
  }

  // Alternative: Manually mark for check
  manualUpdate() {
    this.data.title = 'Manual';
    this.cdr.markForCheck();
    console.log('Marked for check - will update on next CD cycle');
  }
}
```

### 5.3 Zone.js Debugging

```typescript
import { NgZone } from '@angular/core';

@Component({
  selector: 'app-zone-debug'
})
export class ZoneDebugComponent implements OnInit {
  count = 0;

  constructor(private zone: NgZone) {}

  ngOnInit() {
    // Check if running inside Angular zone
    console.log('In Angular Zone:', NgZone.isInAngularZone());
    
    // Run outside Angular zone (no change detection)
    this.zone.runOutsideAngular(() => {
      console.log('Outside Zone:', NgZone.isInAngularZone());
      
      setInterval(() => {
        this.count++; // Won't trigger change detection
        console.log('Count updated (no CD):', this.count);
      }, 1000);
    });

    // Run inside Angular zone (triggers change detection)
    this.zone.run(() => {
      console.log('Inside Zone:', NgZone.isInAngularZone());
      this.count = 10; // Triggers change detection
    });

    // Monitor zone events
    this.zone.onStable.subscribe(() => {
      console.log('Zone stable');
    });

    this.zone.onUnstable.subscribe(() => {
      console.log('Zone unstable');
    });
  }
}
```

---

## 6. Router Debugging

### 6.1 Router Events Debugging

```typescript
import { Router, NavigationStart, NavigationEnd, 
         NavigationCancel, NavigationError } from '@angular/router';

@Component({
  selector: 'app-root'
})
export class AppComponent implements OnInit {
  
  constructor(private router: Router) {}

  ngOnInit() {
    // Debug all router events
    this.router.events.subscribe(event => {
      console.log('Router Event:', event);
    });

    // Debug specific events
    this.router.events.pipe(
      filter(event => event instanceof NavigationStart)
    ).subscribe((event: NavigationStart) => {
      console.log('Navigation started:', event.url);
      debugger;
    });

    this.router.events.pipe(
      filter(event => event instanceof NavigationEnd)
    ).subscribe((event: NavigationEnd) => {
      console.log('Navigation ended:', event.url);
    });

    this.router.events.pipe(
      filter(event => event instanceof NavigationError)
    ).subscribe((event: NavigationError) => {
      console.error('Navigation error:', event.error);
      debugger;
    });
  }
}
```

### 6.2 Route Guards Debugging

```typescript
@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
  
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<boolean> | Promise<boolean> | boolean {
    
    console.group('AuthGuard.canActivate');
    console.log('Route:', route);
    console.log('State:', state);
    console.log('URL:', state.url);
    
    const isAuthenticated = this.authService.isAuthenticated();
    console.log('Is authenticated:', isAuthenticated);
    
    if (!isAuthenticated) {
      console.log('Redirecting to login');
      debugger; // Pause before redirect
      this.router.navigate(['/login']);
    }
    
    console.groupEnd();
    return isAuthenticated;
  }
}

// CanDeactivate guard debugging
@Injectable()
export class UnsavedChangesGuard implements CanDeactivate<ComponentWithChanges> {
  
  canDeactivate(
    component: ComponentWithChanges,
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): boolean {
    
    console.group('UnsavedChangesGuard.canDeactivate');
    console.log('Component:', component);
    console.log('Has unsaved changes:', component.hasUnsavedChanges());
    
    if (component.hasUnsavedChanges()) {
      const result = confirm('You have unsaved changes. Leave anyway?');
      console.log('User decision:', result);
      debugger;
      console.groupEnd();
      return result;
    }
    
    console.groupEnd();
    return true;
  }
}
```

### 6.3 Route Parameters Debugging

```typescript
@Component({
  selector: 'app-user-detail'
})
export class UserDetailComponent implements OnInit {
  
  constructor(
    private route: ActivatedRoute,
    private router: Router
  ) {}

  ngOnInit() {
    // Debug route params
    this.route.params.subscribe(params => {
      console.log('Route params:', params);
      console.log('User ID:', params['id']);
      debugger;
    });

    // Debug query params
    this.route.queryParams.subscribe(params => {
      console.log('Query params:', params);
    });

    // Debug route data
    this.route.data.subscribe(data => {
      console.log('Route data:', data);
    });

    // Get all route info
    console.log('Route snapshot:', this.route.snapshot);
    console.log('Params:', this.route.snapshot.params);
    console.log('Query params:', this.route.snapshot.queryParams);
    console.log('Fragment:', this.route.snapshot.fragment);
    console.log('URL:', this.route.snapshot.url);
  }

  navigate() {
    console.log('Navigating programmatically');
    this.router.navigate(['/users', 123], {
      queryParams: { tab: 'profile' },
      fragment: 'top'
    });
  }
}
```

---

## 7. Forms Debugging

### 7.1 Template-Driven Forms Debugging

```typescript
@Component({
  selector: 'app-template-form-debug',
  template: `
    <form #myForm="ngForm" (ngSubmit)="onSubmit(myForm)">
      <input 
        name="username" 
        [(ngModel)]="username" 
        #usernameInput="ngModel"
        required
        minlength="3">
      
      <!-- Debug form state -->
      <pre>Form Valid: {{ myForm.valid }}</pre>
      <pre>Form Value: {{ myForm.value | json }}</pre>
      <pre>Username Errors: {{ usernameInput.errors | json }}</pre>
      
      <button type="submit">Submit</button>
    </form>
  `
})
export class TemplateFormDebugComponent {
  username = '';

  onSubmit(form: NgForm) {
    console.group('Form Submit');
    console.log('Form valid:', form.valid);
    console.log('Form value:', form.value);
    console.log('Form controls:', form.controls);
    console.log('Form errors:', form.errors);
    
    // Debug specific control
    const usernameControl = form.controls['username'];
    console.log('Username control:', usernameControl);
    console.log('Username errors:', usernameControl.errors);
    console.log('Username touched:', usernameControl.touched);
    console.log('Username dirty:', usernameControl.dirty);
    
    debugger;
    console.groupEnd();
  }
}
```

### 7.2 Reactive Forms Debugging

```typescript
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-reactive-form-debug'
})
export class ReactiveFormDebugComponent implements OnInit {
  userForm!: FormGroup;

  constructor(private fb: FormBuilder) {}

  ngOnInit() {
    this.userForm = this.fb.group({
      username: ['', [Validators.required, Validators.minLength(3)]],
      email: ['', [Validators.required, Validators.email]],
      age: ['', [Validators.required, Validators.min(18)]]
    });

    // Debug value changes
    this.userForm.valueChanges.subscribe(value => {
      console.log('Form value changed:', value);
    });

    // Debug status changes
    this.userForm.statusChanges.subscribe(status => {
      console.log('Form status changed:', status);
    });

    // Debug specific control
    this.userForm.get('username')?.valueChanges.subscribe(value => {
      console.log('Username changed:', value);
    });
  }

  debugForm() {
    console.group('Form Debug');
    console.log('Form value:', this.userForm.value);
    console.log('Form valid:', this.userForm.valid);
    console.log('Form errors:', this.userForm.errors);
    console.log('Form controls:', this.userForm.controls);
    
    // Debug each control
    Object.keys(this.userForm.controls).forEach(key => {
      const control = this.userForm.get(key);
      console.group(`Control: ${key}`);
      console.log('Value:', control?.value);
      console.log('Valid:', control?.valid);
      console.log('Errors:', control?.errors);
      console.log('Touched:', control?.touched);
      console.log('Dirty:', control?.dirty);
      console.groupEnd();
    });
    
    debugger;
    console.groupEnd();
  }

  onSubmit() {
    if (this.userForm.invalid) {
      console.error('Form is invalid');
      this.markFormGroupTouched(this.userForm);
      debugger;
      return;
    }

    console.log('Submitting form:', this.userForm.value);
  }

  // Helper to mark all controls as touched (for validation display)
  private markFormGroupTouched(formGroup: FormGroup) {
    Object.keys(formGroup.controls).forEach(key => {
      const control = formGroup.get(key);
      control?.markAsTouched();

      if (control instanceof FormGroup) {
        this.markFormGroupTouched(control);
      }
    });
  }
}
```

### 7.3 Custom Validators Debugging

```typescript
export class CustomValidators {
  
  // Synchronous validator
  static forbiddenName(forbiddenName: string): ValidatorFn {
    return (control: AbstractControl): ValidationErrors | null => {
      console.group('ForbiddenName Validator');
      console.log('Control value:', control.value);
      console.log('Forbidden name:', forbiddenName);
      
      const forbidden = control.value === forbiddenName;
      console.log('Is forbidden:', forbidden);
      console.groupEnd();
      
      if (forbidden) {
        debugger; // Pause when validation fails
      }
      
      return forbidden ? { forbiddenName: { value: control.value } } : null;
    };
  }

  // Asynchronous validator
  static usernameExists(userService: UserService): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> {
      console.log('Checking if username exists:', control.value);
      
      return userService.checkUsername(control.value).pipe(
        map(exists => {
          console.log('Username exists:', exists);
          debugger; // Pause when async validation completes
          return exists ? { usernameExists: true } : null;
        }),
        catchError(error => {
          console.error('Validation error:', error);
          return of(null);
        })
      );
    };
  }
}
```

---

## 8. HTTP & API Debugging

### 8.1 HTTP Interceptor for Debugging

```typescript
import { HttpInterceptor, HttpRequest, HttpHandler, 
         HttpEvent, HttpResponse, HttpErrorResponse } from '@angular/common/http';

@Injectable()
export class DebugInterceptor implements HttpInterceptor {
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    
    console.group(`HTTP ${req.method} ${req.url}`);
    console.log('Request:', req);
    console.log('Headers:', req.headers);
    console.log('Body:', req.body);
    
    const started = Date.now();
    
    return next.handle(req).pipe(
      tap({
        next: (event) => {
          if (event instanceof HttpResponse) {
            const elapsed = Date.now() - started;
            console.log('Response:', event);
            console.log('Status:', event.status);
            console.log('Body:', event.body);
            console.log(`Time: ${elapsed}ms`);
            console.groupEnd();
          }
        },
        error: (error: HttpErrorResponse) => {
          const elapsed = Date.now() - started;
          console.error('Error Response:', error);
          console.error('Status:', error.status);
          console.error('Message:', error.message);
          console.error(`Time: ${elapsed}ms`);
          debugger; // Pause on HTTP errors
          console.groupEnd();
        }
      })
    );
  }
}

// Register in module
@NgModule({
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: DebugInterceptor,
      multi: true
    }
  ]
})
export class AppModule { }
```

### 8.2 HTTP Service Debugging

```typescript
@Injectable({
  providedIn: 'root'
})
export class ApiService {
  
  constructor(private http: HttpClient) {}

  getUsers(): Observable<User[]> {
    console.log('Fetching users...');
    
    return this.http.get<User[]>('/api/users').pipe(
      tap(users => console.log('Users received:', users.length)),
      catchError(error => {
        console.error('Error fetching users:', error);
        debugger;
        return throwError(() => error);
      })
    );
  }

  // Debug with retry logic
  getUserWithRetry(id: number): Observable<User> {
    let attempt = 0;
    
    return this.http.get<User>(`/api/users/${id}`).pipe(
      tap(() => {
        attempt++;
        console.log(`Attempt ${attempt}`);
      }),
      retry({
        count: 3,
        delay: (error, retryCount) => {
          console.log(`Retry ${retryCount} after error:`, error);
          debugger;
          return timer(1000 * retryCount);
        }
      }),
      catchError(error => {
        console.error('All retries failed:', error);
        return throwError(() => error);
      })
    );
  }
}
```

---

## 9. Performance Optimization

### 9.1 Performance Monitoring

```typescript
@Component({
  selector: 'app-performance-debug'
})
export class PerformanceDebugComponent implements OnInit, AfterViewInit {
  
  ngOnInit() {
    performance.mark('component-init-start');
    // Initialization code
    performance.mark('component-init-end');
    
    performance.measure(
      'component-init',
      'component-init-start',
      'component-init-end'
    );
    
    const measure = performance.getEntriesByName('component-init')[0];
    console.log(`Component init took: ${measure.duration}ms`);
  }

  ngAfterViewInit() {
    // Measure rendering time
    const navEntry = performance.getEntriesByType('navigation')[0] as any;
    console.table({
      'DNS Lookup': navEntry.domainLookupEnd - navEntry.domainLookupStart,
      'TCP Connection': navEntry.connectEnd - navEntry.connectStart,
      'Request Time': navEntry.responseStart - navEntry.requestStart,
      'Response Time': navEntry.responseEnd - navEntry.responseStart,
      'DOM Processing': navEntry.domComplete - navEntry.domLoading,
      'Total Load Time': navEntry.loadEventEnd - navEntry.fetchStart
    });
  }
}
```

### 9.2 TrackBy Function Debugging

```typescript
@Component({
  selector: 'app-list',
  template: `
    <!-- Without trackBy - re-renders all items -->
    <div *ngFor="let item of items">{{ item.name }}</div>
    
    <!-- With trackBy - only re-renders changed items -->
    <div *ngFor="let item of items; trackBy: trackById">
      {{ item.name }}
    </div>
  `
})
export class ListComponent {
  items: Item[] = [];

  trackById(index: number, item: Item): number {
    console.log('trackBy called for:', item.id);
    return item.id; // Angular will only re-render if ID changes
  }

  updateItems() {
    console.time('Update Items');
    // Simulate data update
    this.items = [...this.items, { id: this.items.length + 1, name: 'New' }];
    console.timeEnd('Update Items');
  }
}
```

### 9.3 Lazy Loading Debugging

```typescript
// App routing module
const routes: Routes = [
  {
    path: 'users',
    loadChildren: () => {
      console.log('Loading Users module...');
      return import('./users/users.module').then(m => {
        console.log('Users module loaded');
        return m.UsersModule;
      });
    }
  },
  {
    path: 'admin',
    loadChildren: () => {
      console.log('Loading Admin module...');
      console.time('Admin Module Load');
      return import('./admin/admin.module').then(m => {
        console.timeEnd('Admin Module Load');
        return m.AdminModule;
      });
    },
    canLoad: [AuthGuard] // Debug guard execution
  }
];
```

---

## 10. Common Angular Issues

### 10.1 ExpressionChangedAfterItHasBeenCheckedError

```typescript
// PROBLEM: Causes error
@Component({
  selector: 'app-parent',
  template: `<app-child [value]="parentValue"></app-child>`
})
export class ParentComponent implements AfterViewInit {
  parentValue = 'initial';

  ngAfterViewInit() {
    // BAD: Changing value after view is initialized
    this.parentValue = 'changed';
    // Error: ExpressionChangedAfterItHasBeenCheckedError
  }
}

// SOLUTION 1: Use setTimeout
ngAfterViewInit() {
  setTimeout(() => {
    this.parentValue = 'changed';
    console.log('Value changed after timeout');
  }, 0);
}

// SOLUTION 2: Use ChangeDetectorRef
constructor(private cdr: ChangeDetectorRef) {}

ngAfterViewInit() {
  this.parentValue = 'changed';
  this.cdr.detectChanges();
  console.log('Manual change detection triggered');
}

// SOLUTION 3: Move logic to ngOnInit
ngOnInit() {
  this.parentValue = 'changed';
}
```

### 10.2 Memory Leaks Detection

```typescript
@Component({
  selector: 'app-leak-debug'
})
export class LeakDebugComponent implements OnInit, OnDestroy {
  private subscriptions: Subscription[] = [];
  private intervals: any[] = [];

  ngOnInit() {
    // Track all subscriptions
    const sub1 = interval(1000).subscribe(() => {
      console.log('Tick');
    });
    this.subscriptions.push(sub1);

    // Track all timers
    const timer = setInterval(() => {
      console.log('Timer');
    }, 1000);
    this.intervals.push(timer);

    // Debug component lifecycle
    console.log('Component initialized');
    console.log('Active subscriptions:', this.subscriptions.length);
  }

  ngOnDestroy() {
    console.log('Cleaning up...');
    
    // Unsubscribe all
    this.subscriptions.forEach(sub => {
      if (!sub.closed) {
        console.warn('Unsubscribing:', sub);
        sub.unsubscribe();
      }
    });

    // Clear all timers
    this.intervals.forEach(timer => {
      console.log('Clearing timer:', timer);
      clearInterval(timer);
    });

    console.log('Cleanup complete');
    
    // Verify cleanup
    if (this.subscriptions.some(s => !s.closed)) {
      console.error('⚠️ Memory leak: Some subscriptions not closed!');
      debugger;
    }
  }
}
```

### 10.3 Undefined Property Access

```typescript
@Component({
  selector: 'app-safe-access',
  template: `
    <!-- PROBLEM: Can cause errors if user is undefined -->
    <div>{{ user.name }}</div>
    
    <!-- SOLUTION 1: Safe navigation operator -->
    <div>{{ user?.name }}</div>
    
    <!-- SOLUTION 2: ngIf -->
    <div *ngIf="user">{{ user.name }}</div>
    
    <!-- SOLUTION 3: Default value -->
    <div>{{ user?.name || 'Unknown' }}</div>
  `
})
export class SafeAccessComponent implements OnInit {
  user: User | undefined;

  ngOnInit() {
    this.loadUser();
  }

  loadUser() {
    console.log('Loading user...');
    
    this.userService.getUser().subscribe({
      next: (user) => {
        console.log('User loaded:', user);
        this.user = user;
      },
      error: (error) => {
        console.error('Failed to load user:', error);
        debugger; // Debug failed user load
      }
    });
  }
}
```

### 10.4 Circular Dependency

```typescript
// PROBLEM: Circular dependency
// service-a.ts
@Injectable()
export class ServiceA {
  constructor(private serviceB: ServiceB) {} // Depends on B
}

// service-b.ts
@Injectable()
export class ServiceB {
  constructor(private serviceA: ServiceA) {} // Depends on A
}

// SOLUTION 1: Use Injector
@Injectable()
export class ServiceA {
  private serviceB!: ServiceB;
  
  constructor(private injector: Injector) {
    // Delay injection
    setTimeout(() => {
      this.serviceB = this.injector.get(ServiceB);
      console.log('ServiceB injected');
    });
  }
}

// SOLUTION 2: Refactor to remove dependency
@Injectable()
export class SharedService {
  // Common logic here
}

@Injectable()
export class ServiceA {
  constructor(private shared: SharedService) {}
}

@Injectable()
export class ServiceB {
  constructor(private shared: SharedService) {}
}
```

---

## 11. Debugging Tools & Extensions

### 11.1 Angular DevTools Features

**Component Explorer:**
- Inspect component tree
- View component properties
- Modify values in real-time
- See injected dependencies

**Profiler:**
- Record component render times
- Identify performance bottlenecks
- View change detection cycles
- Export profiling data

**Router Tree:**
- Visualize route configuration
- See active routes
- Debug route guards

### 11.2 Augury (Legacy)

*Note: Augury is deprecated. Use Angular DevTools instead.*

### 11.3 Redux DevTools (for NgRx)

```typescript
// Store module with DevTools
@NgModule({
  imports: [
    StoreModule.forRoot(reducers),
    StoreDevtoolsModule.instrument({
      maxAge: 25, // Retains last 25 states
      logOnly: environment.production, // Restrict in production
    })
  ]
})
export class AppModule { }

// Debug actions
@Injectable()
export class MyEffects {
  loadData$ = createEffect(() =>
    this.actions$.pipe(
      ofType(loadData),
      tap(action => {
        console.log('Action dispatched:', action);
        debugger;
      }),
      switchMap(() =>
        this.apiService.getData().pipe(
          map(data => {
            console.log('Data loaded:', data);
            return loadDataSuccess({ data });
          }),
          catchError(error => {
            console.error('Load failed:', error);
            return of(loadDataFailure({ error }));
          })
        )
      )
    )
  );

  constructor(
    private actions$: Actions,
    private apiService: ApiService
  ) {}
}
```

---

## 12. Best Practices Summary

### ✅ DO's

1. **Use Angular DevTools** - Essential for component debugging
2. **Enable source maps** - Debug TypeScript, not JavaScript
3. **Use RxJS tap operator** - Log observable values
4. **Implement error handlers** - Global and local error handling
5. **Use console groups** - Organize related logs
6. **Debug in development mode** - More detailed error messages
7. **Use strict mode** - Catch errors early (`"strict": true` in tsconfig)
8. **Track subscriptions** - Prevent memory leaks
9. **Use lazy loading** - Improve initial load time
10. **Profile before optimizing** - Measure first

### ❌ DON'Ts

1. **Don't leave debugger statements** - Remove before production
2. **Don't mutate state** - Especially with OnPush
3. **Don't forget to unsubscribe** - Causes memory leaks
4. **Don't use `any` type** - Loses type safety
5. **Don't ignore console errors** - They indicate real problems
6. **Don't disable strict mode** - It helps find bugs
7. **Don't skip error handling** - Always handle errors
8. **Don't use `console.log` in production** - Use proper logging service

---

## 13. Quick Debug Commands

```typescript
// In Chrome Console with Angular app running

// Get component instance
ng.getComponent($0)

// Get all components on element
ng.getComponents($0)

// Trigger change detection
ng.applyChanges($0)

// Get directives
ng.getDirectives($0)

// Get host element
ng.getHostElement(component)

// Get owning component
ng.getOwningComponent($0)

// Get root components
ng.getRootComponents()

// Get injector
ng.getInjector($0)

// Probe element (legacy)
ng.probe($0)
```

---

## Conclusion

Debugging Angular applications requires understanding:

1. **Component lifecycle** - Know when things happen
2. **Change detection** - How and when updates occur
3. **RxJS streams** - Async data flow
4. **Dependency injection** - Service instantiation
5. **Router mechanics** - Navigation lifecycle
6. **Form validation** - State management

Use the right tool for each scenario:
- **Angular DevTools** - Component inspection
- **Chrome DevTools** - JavaScript debugging  
- **Console API** - Quick logging
- **Breakpoints** - Step-through debugging
- **Performance tab** - Optimization

Happy debugging! 🐛