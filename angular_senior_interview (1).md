# Angular Interview Questions & Answers - Senior Level (15+ Years Experience)

## Core Angular Concepts

### 1. Angular Architecture & Core Concepts

**Q: Explain Angular's architecture and core building blocks.**

**A:** Angular follows a component-based architecture with the following core building blocks:

**Components:**
- Building blocks of UI
- Consists of TypeScript class, HTML template, and CSS
- Decorated with `@Component()`

```typescript
@Component({
  selector: 'app-user',
  templateUrl: './user.component.html',
  styleUrls: ['./user.component.css'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserComponent implements OnInit, OnDestroy {
  @Input() userId: string;
  @Output() userDeleted = new EventEmitter<string>();
  @ViewChild('userForm') form: ElementRef;
  @ContentChild(TemplateRef) template: TemplateRef<any>;
  
  user$: Observable<User>;
  
  constructor(
    private userService: UserService,
    private cd: ChangeDetectorRef
  ) {}
  
  ngOnInit() {
    this.user$ = this.userService.getUser(this.userId);
  }
  
  ngOnDestroy() {
    // Cleanup
  }
}
```

**Modules:**
- Containers for cohesive blocks of code
- Every Angular app has at least one module (AppModule)

```typescript
@NgModule({
  declarations: [AppComponent, UserComponent],
  imports: [BrowserModule, HttpClientModule, RouterModule],
  providers: [UserService],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

**Services & Dependency Injection:**
- Singleton objects for business logic and data
- Injected via Angular's DI system

```typescript
@Injectable({
  providedIn: 'root' // Tree-shakeable
})
export class UserService {
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users');
  }
}
```

**Q: Explain Dependency Injection in Angular in depth.**

**A:** Angular's DI system provides dependencies to classes upon instantiation.

**Hierarchical Injector:**
```typescript
// Root level - App-wide singleton
@Injectable({ providedIn: 'root' })
export class GlobalService { }

// Module level
@NgModule({
  providers: [ModuleService] // One instance per module
})
export class FeatureModule { }

// Component level
@Component({
  providers: [ComponentService] // New instance per component
})
export class MyComponent { }
```

**Injection Tokens:**
```typescript
// Using InjectionToken for non-class dependencies
export const API_URL = new InjectionToken<string>('api.url');

@NgModule({
  providers: [
    { provide: API_URL, useValue: 'https://api.example.com' }
  ]
})
export class AppModule { }

// Inject in service
@Injectable()
export class ApiService {
  constructor(@Inject(API_URL) private apiUrl: string) {}
}
```

**Provider Types:**
```typescript
// useClass - provide a different class
{ provide: LoggerService, useClass: AdvancedLogger }

// useValue - provide a static value
{ provide: 'API_KEY', useValue: '12345' }

// useFactory - create dynamically
{
  provide: DataService,
  useFactory: (http: HttpClient, config: Config) => {
    return config.useMock ? new MockDataService() : new DataService(http);
  },
  deps: [HttpClient, Config]
}

// useExisting - alias to another provider
{ provide: NewLogger, useExisting: LoggerService }
```

**Optional and Multiple Injection:**
```typescript
export class MyComponent {
  constructor(
    @Optional() private logger?: LoggerService, // May be null
    @Self() private localService: LocalService, // Only from this injector
    @SkipSelf() private parentService: ParentService, // Skip this injector
    @Host() private hostService: HostService, // Up to host component
    @Inject(PLUGINS) private plugins: Plugin[] // Multi providers
  ) {}
}
```

### 2. Change Detection

**Q: Explain Angular's change detection mechanism.**

**A:** Angular checks the component tree to detect changes and update the DOM.

**Default Change Detection:**
- Runs on every browser event, timer, XHR, promise
- Checks entire component tree top to bottom
- Can be inefficient for large applications

```typescript
@Component({
  selector: 'app-user',
  template: `
    <div>{{ user.name }}</div>
    <button (click)="updateUser()">Update</button>
  `
})
export class UserComponent {
  user = { name: 'John' };
  
  updateUser() {
    this.user.name = 'Jane'; // Triggers change detection
  }
}
```

**OnPush Change Detection:**
- Only checks when:
  - Input reference changes
  - Event originates from component
  - Observable emits (async pipe)
  - Manual trigger via ChangeDetectorRef

```typescript
@Component({
  selector: 'app-user',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div>{{ user.name }}</div>
    <div>{{ (userData$ | async)?.name }}</div>
  `
})
export class UserComponent {
  @Input() user: User; // Must receive new reference
  userData$: Observable<User>;
  
  constructor(private cd: ChangeDetectorRef) {}
  
  updateUser() {
    // This won't trigger change detection with OnPush
    this.user.name = 'Jane';
    
    // Must create new reference
    this.user = { ...this.user, name: 'Jane' };
    
    // Or manually trigger
    this.cd.markForCheck();
  }
  
  detachChangeDetection() {
    this.cd.detach(); // Stop automatic detection
  }
  
  reattachChangeDetection() {
    this.cd.reattach(); // Resume automatic detection
  }
  
  detectChangesManually() {
    this.cd.detectChanges(); // Run detection immediately
  }
}
```

**NgZone:**
```typescript
export class AppComponent {
  constructor(private ngZone: NgZone) {}
  
  // Run outside Angular's zone (no change detection)
  runOutsideAngular() {
    this.ngZone.runOutsideAngular(() => {
      setInterval(() => {
        // Heavy computation that doesn't need change detection
        this.processData();
      }, 100);
    });
  }
  
  // Run inside zone (triggers change detection)
  runInsideAngular() {
    this.ngZone.run(() => {
      this.updateUI();
    });
  }
}
```

**Q: What are Signals and how do they compare to traditional change detection?**

**A:** Signals (Angular 16+) provide fine-grained reactivity.

```typescript
import { Component, signal, computed, effect } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <div>Count: {{ count() }}</div>
    <div>Double: {{ double() }}</div>
    <button (click)="increment()">Increment</button>
  `
})
export class CounterComponent {
  // Signal - writable reactive value
  count = signal(0);
  
  // Computed - derived reactive value
  double = computed(() => this.count() * 2);
  
  constructor() {
    // Effect - side effects when signals change
    effect(() => {
      console.log('Count changed:', this.count());
    });
  }
  
  increment() {
    this.count.update(value => value + 1);
    // or this.count.set(10);
  }
}

// Signals with services
@Injectable({ providedIn: 'root' })
export class UserStore {
  private usersSignal = signal<User[]>([]);
  
  users = this.usersSignal.asReadonly();
  userCount = computed(() => this.users().length);
  
  addUser(user: User) {
    this.usersSignal.update(users => [...users, user]);
  }
  
  removeUser(id: string) {
    this.usersSignal.update(users => users.filter(u => u.id !== id));
  }
}
```

**Comparison:**
- **Zone.js (Traditional)**: Global, coarse-grained, checks entire tree
- **Signals**: Local, fine-grained, only updates what changed
- **Benefits**: Better performance, simpler mental model, no Zone.js dependency

### 3. Component Communication

**Q: Explain different ways components can communicate.**

**A:**

**1. Parent to Child - @Input:**
```typescript
// Parent
@Component({
  template: `<app-child [user]="currentUser"></app-child>`
})
export class ParentComponent {
  currentUser = { name: 'John' };
}

// Child
@Component({
  selector: 'app-child'
})
export class ChildComponent {
  @Input() user: User;
  
  // Intercept input changes
  @Input()
  set userName(name: string) {
    this._userName = name?.trim();
  }
  get userName(): string {
    return this._userName;
  }
  private _userName: string;
  
  // React to input changes
  ngOnChanges(changes: SimpleChanges) {
    if (changes['user']) {
      console.log('User changed:', changes['user'].currentValue);
    }
  }
}
```

**2. Child to Parent - @Output:**
```typescript
// Child
@Component({
  selector: 'app-child'
})
export class ChildComponent {
  @Output() userDeleted = new EventEmitter<string>();
  @Output() userUpdated = new EventEmitter<User>();
  
  deleteUser(id: string) {
    this.userDeleted.emit(id);
  }
}

// Parent
@Component({
  template: `
    <app-child 
      (userDeleted)="onUserDeleted($event)"
      (userUpdated)="onUserUpdated($event)">
    </app-child>
  `
})
export class ParentComponent {
  onUserDeleted(id: string) {
    console.log('User deleted:', id);
  }
}
```

**3. Template Reference Variable:**
```typescript
@Component({
  template: `
    <app-child #childComponent></app-child>
    <button (click)="childComponent.doSomething()">Call Child</button>
  `
})
export class ParentComponent { }
```

**4. ViewChild/ViewChildren:**
```typescript
@Component({
  template: `
    <app-child></app-child>
    <app-child></app-child>
  `
})
export class ParentComponent implements AfterViewInit {
  @ViewChild(ChildComponent) child: ChildComponent;
  @ViewChildren(ChildComponent) children: QueryList<ChildComponent>;
  
  ngAfterViewInit() {
    this.child.doSomething();
    this.children.forEach(child => child.doSomething());
  }
}
```

**5. Service with Subject/BehaviorSubject:**
```typescript
@Injectable({ providedIn: 'root' })
export class DataService {
  private dataSubject = new BehaviorSubject<Data>(null);
  data$ = this.dataSubject.asObservable();
  
  updateData(data: Data) {
    this.dataSubject.next(data);
  }
}

// Component A
export class ComponentA {
  constructor(private dataService: DataService) {}
  
  sendData() {
    this.dataService.updateData({ message: 'Hello' });
  }
}

// Component B
export class ComponentB {
  data$ = this.dataService.data$;
  
  constructor(private dataService: DataService) {}
}
```

**6. State Management (NgRx):**
```typescript
// Actions
export const loadUsers = createAction('[User] Load Users');
export const loadUsersSuccess = createAction(
  '[User] Load Users Success',
  props<{ users: User[] }>()
);

// Component A dispatches
this.store.dispatch(loadUsers());

// Component B selects
users$ = this.store.select(selectAllUsers);
```

## RxJS & Reactive Programming

### 4. RxJS Operators & Patterns

**Q: Explain commonly used RxJS operators with real-world examples.**

**A:**

**Transformation Operators:**
```typescript
// map - Transform each value
this.userService.getUser(id).pipe(
  map(user => user.name.toUpperCase())
).subscribe(name => console.log(name));

// switchMap - Switch to new observable (cancels previous)
this.searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => this.searchService.search(term))
).subscribe(results => this.results = results);

// mergeMap - Merge multiple observables (doesn't cancel)
this.userIds$.pipe(
  mergeMap(id => this.userService.getUser(id))
).subscribe(user => this.users.push(user));

// concatMap - Sequential execution (waits for previous)
this.requests$.pipe(
  concatMap(request => this.http.post('/api', request))
).subscribe(response => console.log(response));

// exhaustMap - Ignore new emissions while processing
this.saveButton.clicks$.pipe(
  exhaustMap(() => this.userService.save(this.user))
).subscribe();
```

**Filtering Operators:**
```typescript
// filter - Emit values that pass condition
this.numbers$.pipe(
  filter(n => n % 2 === 0)
).subscribe(even => console.log(even));

// take - Take first n values
this.data$.pipe(take(5)).subscribe();

// takeUntil - Take until another observable emits
destroy$ = new Subject<void>();

ngOnInit() {
  this.data$.pipe(
    takeUntil(this.destroy$)
  ).subscribe(data => this.data = data);
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}

// distinctUntilChanged - Only emit when value changes
this.searchControl.valueChanges.pipe(
  distinctUntilChanged()
).subscribe();

// debounceTime - Emit after silence period
this.searchControl.valueChanges.pipe(
  debounceTime(300)
).subscribe();

// throttleTime - Emit at most once per period
this.scrollEvents$.pipe(
  throttleTime(100)
).subscribe();
```

**Combination Operators:**
```typescript
// combineLatest - Combine latest values from all
combineLatest([
  this.userService.getUser(id),
  this.postsService.getPosts(id),
  this.settingsService.getSettings(id)
]).pipe(
  map(([user, posts, settings]) => ({ user, posts, settings }))
).subscribe(data => this.data = data);

// forkJoin - Wait for all to complete (like Promise.all)
forkJoin({
  user: this.userService.getUser(id),
  posts: this.postsService.getPosts(id)
}).subscribe(({ user, posts }) => {
  this.user = user;
  this.posts = posts;
});

// merge - Merge multiple observables into one
merge(
  this.clicks$,
  this.hovers$,
  this.touches$
).subscribe(event => this.handleEvent(event));

// zip - Combine values at same index
zip(
  interval(1000),
  of('a', 'b', 'c')
).subscribe(([num, letter]) => console.log(num, letter));
```

**Error Handling:**
```typescript
// catchError - Handle errors
this.userService.getUser(id).pipe(
  catchError(error => {
    console.error('Error:', error);
    return of(null); // Return fallback value
    // or return throwError(() => new Error('Failed'));
  })
).subscribe();

// retry - Retry on error
this.http.get('/api/data').pipe(
  retry(3),
  catchError(error => of([]))
).subscribe();

// retryWhen - Retry with custom logic
this.http.get('/api/data').pipe(
  retryWhen(errors => errors.pipe(
    delay(1000),
    take(3),
    concat(throwError(() => new Error('Max retries')))
  ))
).subscribe();
```

**Utility Operators:**
```typescript
// tap - Side effects without modifying stream
this.userService.getUser(id).pipe(
  tap(user => console.log('User loaded:', user)),
  tap({
    next: user => this.logService.log('Success', user),
    error: err => this.logService.error('Failed', err),
    complete: () => this.logService.log('Complete')
  })
).subscribe();

// shareReplay - Share and replay emissions
this.config$ = this.http.get('/api/config').pipe(
  shareReplay({ bufferSize: 1, refCount: true })
);

// Multiple subscribers share same request
this.config$.subscribe();
this.config$.subscribe(); // Doesn't make new HTTP request
```

**Q: Explain hot vs cold observables.**

**A:**

**Cold Observable:**
- Starts emitting when subscribed
- Each subscriber gets independent stream
- Data producer created inside observable

```typescript
// Cold - Each subscriber gets separate HTTP request
const cold$ = this.http.get('/api/data');

cold$.subscribe(data => console.log('Sub 1:', data));
cold$.subscribe(data => console.log('Sub 2:', data)); // New request

// Cold - Each subscriber gets separate interval
const interval$ = interval(1000);

interval$.subscribe(n => console.log('Sub 1:', n));
setTimeout(() => {
  interval$.subscribe(n => console.log('Sub 2:', n)); // Starts from 0
}, 3000);
```

**Hot Observable:**
- Emits regardless of subscribers
- Subscribers share same stream
- Data producer outside observable

```typescript
// Hot - Shared Subject
const hot$ = new Subject<number>();

hot$.subscribe(n => console.log('Sub 1:', n));
hot$.next(1);
hot$.next(2);

hot$.subscribe(n => console.log('Sub 2:', n)); // Misses 1 and 2
hot$.next(3); // Both receive 3

// Convert cold to hot with share()
const shared$ = this.http.get('/api/data').pipe(
  share() // or shareReplay(1)
);

shared$.subscribe(data => console.log('Sub 1:', data));
shared$.subscribe(data => console.log('Sub 2:', data)); // Shares request
```

**Q: Memory leaks with RxJS and how to prevent them.**

**A:**

**Common Causes:**
```typescript
// BAD - Subscription leak
export class BadComponent implements OnInit {
  ngOnInit() {
    this.dataService.getData().subscribe(data => {
      this.data = data;
    }); // Never unsubscribed
  }
}

// GOOD - Manual unsubscribe
export class GoodComponent implements OnInit, OnDestroy {
  private subscription: Subscription;
  
  ngOnInit() {
    this.subscription = this.dataService.getData().subscribe(data => {
      this.data = data;
    });
  }
  
  ngOnDestroy() {
    this.subscription.unsubscribe();
  }
}

// BETTER - Multiple subscriptions
export class BetterComponent implements OnInit, OnDestroy {
  private subscriptions = new Subscription();
  
  ngOnInit() {
    this.subscriptions.add(
      this.dataService.getData().subscribe(data => this.data = data)
    );
    
    this.subscriptions.add(
      this.userService.getUser().subscribe(user => this.user = user)
    );
  }
  
  ngOnDestroy() {
    this.subscriptions.unsubscribe(); // Unsubscribes all
  }
}

// BEST - takeUntil pattern
export class BestComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  
  ngOnInit() {
    this.dataService.getData().pipe(
      takeUntil(this.destroy$)
    ).subscribe(data => this.data = data);
    
    this.userService.getUser().pipe(
      takeUntil(this.destroy$)
    ).subscribe(user => this.user = user);
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// EXCELLENT - Async pipe (auto-unsubscribes)
@Component({
  template: `
    <div *ngIf="data$ | async as data">
      {{ data.name }}
    </div>
  `
})
export class AsyncComponent {
  data$ = this.dataService.getData(); // No manual subscription needed
}
```

## Forms

### 5. Reactive Forms vs Template-Driven Forms

**Q: Compare Reactive Forms and Template-Driven Forms. When to use each?**

**A:**

**Template-Driven Forms:**
- Two-way data binding with ngModel
- Logic in template
- Simpler for basic forms
- Less control, harder to test

```typescript
// Component
@Component({
  template: `
    <form #userForm="ngForm" (ngSubmit)="onSubmit(userForm)">
      <input 
        name="username" 
        [(ngModel)]="user.username"
        required
        minlength="3"
        #username="ngModel">
      
      <div *ngIf="username.invalid && username.touched">
        <span *ngIf="username.errors?.['required']">Required</span>
        <span *ngIf="username.errors?.['minlength']">Min 3 chars</span>
      </div>
      
      <button [disabled]="userForm.invalid">Submit</button>
    </form>
  `
})
export class TemplateFormComponent {
  user = { username: '', email: '' };
  
  onSubmit(form: NgForm) {
    if (form.valid) {
      console.log(this.user);
    }
  }
}
```

**Reactive Forms:**
- Explicit form model in component
- More control and flexibility
- Easier to test
- Better for complex forms

```typescript
@Component({
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <input formControlName="username">
      
      <div *ngIf="username.invalid && username.touched">
        <span *ngIf="username.hasError('required')">Required</span>
        <span *ngIf="username.hasError('minlength')">
          Min {{ username.errors?.['minlength'].requiredLength }} chars
        </span>
      </div>
      
      <div formGroupName="address">
        <input formControlName="street">
        <input formControlName="city">
      </div>
      
      <div formArrayName="phones">
        <div *ngFor="let phone of phones.controls; let i = index">
          <input [formControlName]="i">
        </div>
      </div>
      
      <button [disabled]="userForm.invalid">Submit</button>
    </form>
  `
})
export class ReactiveFormComponent implements OnInit {
  userForm: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit() {
    this.userForm = this.fb.group({
      username: ['', [Validators.required, Validators.minLength(3)]],
      email: ['', [Validators.required, Validators.email]],
      password: ['', [
        Validators.required,
        Validators.pattern(/^(?=.*[A-Z])(?=.*[0-9]).{8,}$/)
      ]],
      confirmPassword: [''],
      address: this.fb.group({
        street: [''],
        city: ['']
      }),
      phones: this.fb.array([
        this.fb.control('')
      ])
    }, {
      validators: this.passwordMatchValidator
    });
    
    // React to value changes
    this.userForm.get('username').valueChanges.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(username => this.userService.checkUsername(username))
    ).subscribe(available => {
      if (!available) {
        this.username.setErrors({ taken: true });
      }
    });
  }
  
  // Custom validator
  passwordMatchValidator(group: FormGroup) {
    const password = group.get('password').value;
    const confirmPassword = group.get('confirmPassword').value;
    return password === confirmPassword ? null : { passwordMismatch: true };
  }
  
  // Getters for convenience
  get username() {
    return this.userForm.get('username');
  }
  
  get phones() {
    return this.userForm.get('phones') as FormArray;
  }
  
  addPhone() {
    this.phones.push(this.fb.control(''));
  }
  
  removePhone(index: number) {
    this.phones.removeAt(index);
  }
  
  onSubmit() {
    if (this.userForm.valid) {
      console.log(this.userForm.value);
    } else {
      this.userForm.markAllAsTouched();
    }
  }
}
```

**Custom Validators:**
```typescript
// Synchronous validator
export function forbiddenNameValidator(nameRe: RegExp): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const forbidden = nameRe.test(control.value);
    return forbidden ? { forbiddenName: { value: control.value } } : null;
  };
}

// Async validator
export function uniqueUsernameValidator(
  userService: UserService
): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    return userService.checkUsername(control.value).pipe(
      map(exists => exists ? { usernameTaken: true } : null),
      catchError(() => of(null))
    );
  };
}

// Usage
this.userForm = this.fb.group({
  username: ['', 
    [Validators.required, forbiddenNameValidator(/admin/i)],
    [uniqueUsernameValidator(this.userService)]
  ]
});
```

**Dynamic Forms:**
```typescript
@Injectable()
export class DynamicFormService {
  createForm(questions: Question[]): FormGroup {
    const group: any = {};
    
    questions.forEach(question => {
      group[question.key] = question.required
        ? [question.value || '', Validators.required]
        : [question.value || ''];
    });
    
    return new FormGroup(group);
  }
}

// Usage
export class DynamicFormComponent {
  questions: Question[] = [
    { key: 'name', label: 'Name', type: 'text', required: true },
    { key: 'email', label: 'Email', type: 'email', required: true },
    { key: 'age', label: 'Age', type: 'number', required: false }
  ];
  
  form = this.formService.createForm(this.questions);
}
```

## State Management

### 6. NgRx & State Management

**Q: Explain NgRx architecture and when to use it.**

**A:** NgRx is a Redux-inspired state management library for Angular.

**Core Concepts:**

**Actions:**
```typescript
// user.actions.ts
import { createAction, props } from '@ngrx/store';

export const loadUsers = createAction('[User List] Load Users');

export const loadUsersSuccess = createAction(
  '[User API] Load Users Success',
  props<{ users: User[] }>()
);

export const loadUsersFailure = createAction(
  '[User API] Load Users Failure',
  props<{ error: string }>()
);

export const addUser = createAction(
  '[User Form] Add User',
  props<{ user: User }>()
);

export const updateUser = createAction(
  '[User Edit] Update User',
  props<{ user: Update<User> }>()
);

export const deleteUser = createAction(
  '[User List] Delete User',
  props<{ id: string }>()
);
```

**Reducer:**
```typescript
// user.reducer.ts
import { createReducer, on } from '@ngrx/store';
import { EntityState, EntityAdapter, createEntityAdapter } from '@ngrx/entity';

export interface UserState extends EntityState<User> {
  loading: boolean;
  error: string | null;
  selectedUserId: string | null;
}

export const userAdapter: EntityAdapter<User> = createEntityAdapter<User>();

const initialState: UserState = userAdapter.getInitialState({
  loading: false,
  error: null,
  selectedUserId: null
});

export const userReducer = createReducer(
  initialState,
  
  on(UserActions.loadUsers, state => ({
    ...state,
    loading: true,
    error: null
  })),
  
  on(UserActions.loadUsersSuccess, (state, { users }) =>
    userAdapter.setAll(users, {
      ...state,
      loading: false
    })
  ),
  
  on(UserActions.loadUsersFailure, (state, { error }) => ({
    ...state,
    loading: false,
    error
  })),
  
  on(UserActions.addUser, (state, { user }) =>
    userAdapter.addOne(user, state)
  ),
  
  on(UserActions.updateUser, (state, { user }) =>
    userAdapter.updateOne(user, state)
  ),
  
  on(UserActions.deleteUser, (state, { id }) =>
    userAdapter.removeOne(id, state)
  )
);

// Selectors
const { selectAll, selectEntities, selectIds, selectTotal } = 
  userAdapter.getSelectors();

export const selectAllUsers = selectAll;
export const selectUserEntities = selectEntities;
export const selectUserIds = selectIds;
export const selectUserTotal = selectTotal;
```

**Selectors:**
```typescript
// user.selectors.ts
import { createFeatureSelector, createSelector } from '@ngrx/store';

export const selectUserState = createFeatureSelector<UserState>('users');

export const selectAllUsers = createSelector(
  selectUserState,
  userAdapter.getSelectors().selectAll
);

export const selectUsersLoading = createSelector(
  selectUserState,
  state => state.loading
);

export const selectUsersError = createSelector(
  selectUserState,
  state => state.error
);

export const selectSelectedUserId = createSelector(
  selectUserState,
  state => state.selectedUserId
);

export const selectSelectedUser = createSelector(
  selectUserState,
  selectSelectedUserId,
  (state, userId) => state.entities[userId]
);

// Composed selectors
export const selectActiveUsers = createSelector(
  selectAllUsers,
  users => users.filter(user => user.active)
);

export const selectUsersByRole = (role: string) = createSelector(
  selectAllUsers,
  users => users.filter(user => user.role === role)
);

// Selector with props
export const selectUserById = createSelector(
  selectUserState,
  (state: UserState, props: { id: string }) => state.entities[props.id]
);
```

**Effects:**
```typescript
// user.effects.ts
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { of } from 'rxjs';
import { map, catchError, switchMap, mergeMap, tap } from 'rxjs/operators';

@Injectable()
export class UserEffects {
  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UserActions.loadUsers),
      switchMap(() =>
        this.userService.getUsers().pipe(
          map(users => UserActions.loadUsersSuccess({ users })),
          catchError(error =>
            of(UserActions.loadUsersFailure({ error: error.message }))
          )
        )
      )
    )
  );
  
  addUser$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UserActions.addUser),
      mergeMap(({ user }) =>
        this.userService.createUser(user).pipe(
          map(newUser => UserActions.addUserSuccess({ user: newUser })),
          catchError(error =>
            of(UserActions.addUserFailure({ error: error.message }))
          )
        )
      )
    )
  );
  
  deleteUser$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UserActions.deleteUser),
      mergeMap(({ id }) =>
        this.userService.deleteUser(id).pipe(
          map(() => UserActions.deleteUserSuccess({ id })),
          catchError(error =>
            of(UserActions.deleteUserFailure({ error: error.message }))
          )
        )
      )
    )
  );
  
  // Non-dispatching effect (side effects only)
  logActions$ = createEffect(() =>
    this.actions$.pipe(
      tap(action => console.log('Action:', action))
    ),
    { dispatch: false }
  );
  
  // Navigate after success
  navigateAfterCreate$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UserActions.addUserSuccess),
      tap(({ user }) => this.router.navigate(['/users', user.id]))
    ),
    { dispatch: false }
  );
  
  constructor(
    private actions$: Actions,
    private userService: UserService,
    private router: Router
  ) {}
}
```

**Component Usage:**
```typescript
@Component({
  selector: 'app-user-list',
  template: `
    <div *ngIf="loading$ | async">Loading...</div>
    <div *ngIf="error$ | async as error">{{ error }}</div>
    
    <div *ngFor="let user of users$ | async">
      {{ user.name }}
      <button (click)="deleteUser(user.id)">Delete</button>
    </div>
  `
})
export class UserListComponent implements OnInit {
  users$ = this.store.select(selectAllUsers);
  loading$ = this.store.select(selectUsersLoading);
  error$ = this.store.select(selectUsersError);
  
  constructor(private store: Store) {}
  
  ngOnInit() {
    this.store.dispatch(UserActions.loadUsers());
  }
  
  deleteUser(id: string) {
    this.store.dispatch(UserActions.deleteUser({ id }));
  }
}
```

**When to use NgRx:**
- Large applications with complex state
- Multiple data sources
- Many components sharing state
- Need for time-travel debugging
- Team familiar with Redux patterns

**Alternatives:**
- Services with BehaviorSubject (simpler apps)
- Component Store (@ngrx/component-store)
- Akita, Elf, or other state management libraries
- Signals (Angular 16+)

**Q: Explain NgRx Component Store.**

**A:** Component Store is lightweight, local state management.

```typescript
@Injectable()
export class UserComponentStore extends ComponentStore<UserState> {
  constructor(private userService: UserService) {
    super({ users: [], loading: false, error: null });
  }
  
  // Selectors
  readonly users$ = this.select(state => state.users);
  readonly loading$ = this.select(state => state.loading);
  readonly error$ = this.select(state => state.error);
  
  // Composed selector
  readonly activeUsers$ = this.select(
    this.users$,
    users => users.filter(u => u.active)
  );
  
  // Updaters (synchronous state updates)
  readonly setUsers = this.updater((state, users: User[]) => ({
    ...state,
    users,
    loading: false
  }));
  
  readonly setLoading = this.updater((state, loading: boolean) => ({
    ...state,
    loading
  }));
  
  readonly addUser = this.updater((state, user: User) => ({
    ...state,
    users: [...state.users, user]
  }));
  
  readonly updateUser = this.updater((state, updated: User) => ({
    ...state,
    users: state.users.map(u => u.id === updated.id ? updated : u)
  }));
  
  readonly removeUser = this.updater((state, id: string) => ({
    ...state,
    users: state.users.filter(u => u.id !== id)
  }));
  
  // Effects (asynchronous operations)
  readonly loadUsers = this.effect((trigger$: Observable<void>) =>
    trigger$.pipe(
      tap(() => this.setLoading(true)),
      switchMap(() =>
        this.userService.getUsers().pipe(
          tapResponse(
            users => this.setUsers(users),
            error => this.patchState({ error, loading: false })
          )
        )
      )
    )
  );
  
  readonly saveUser = this.effect((user$: Observable<User>) =>
    user$.pipe(
      switchMap(user =>
        this.userService.saveUser(user).pipe(
          tapResponse(
            savedUser => this.addUser(savedUser),
            error => this.patchState({ error })
          )
        )
      )
    )
  );
}

// Usage in component
@Component({
  template: `
    <div *ngIf="store.loading$ | async">Loading...</div>
    <div *ngFor="let user of store.users$ | async">
      {{ user.name }}
    </div>
    <button (click)="loadUsers()">Load</button>
  `,
  providers: [UserComponentStore]
})
export class UserComponent implements OnInit {
  constructor(readonly store: UserComponentStore) {}
  
  ngOnInit() {
    this.store.loadUsers();
  }
  
  loadUsers() {
    this.store.loadUsers();
  }
}
```

## Routing & Navigation

### 7. Advanced Routing

**Q: Explain Angular Router features and advanced patterns.**

**A:**

**Basic Routing:**
```typescript
const routes: Routes = [
  { path: '', redirectTo: '/home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent },
  { path: 'users/:id', component: UserDetailComponent },
  { path: 'admin', component: AdminComponent, canActivate: [AuthGuard] },
  { path: '**', component: NotFoundComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

**Lazy Loading:**
```typescript
const routes: Routes = [
  {
    path: 'users',
    loadChildren: () => import('./users/users.module')
      .then(m => m.UsersModule)
  },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module')
      .then(m => m.AdminModule),
    canLoad: [AuthGuard]
  }
];

// Preloading strategy
@Injectable()
export class CustomPreloadStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    return route.data?.['preload'] ? load() : of(null);
  }
}

RouterModule.forRoot(routes, {
  preloadingStrategy: CustomPreloadStrategy
})
```

**Route Guards:**
```typescript
// CanActivate - Can navigate to route
@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}
  
  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<boolean | UrlTree> {
    return this.authService.isLoggedIn$.pipe(
      map(isLoggedIn => {
        if (isLoggedIn) {
          return true;
        }
        return this.router.createUrlTree(['/login'], {
          queryParams: { returnUrl: state.url }
        });
      })
    );
  }
}

// CanActivateChild - Can navigate to child routes
@Injectable({ providedIn: 'root' })
export class AdminGuard implements CanActivateChild {
  canActivateChild(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): boolean {
    return this.authService.hasRole('admin');
  }
}

// CanDeactivate - Can leave route
export interface CanComponentDeactivate {
  canDeactivate: () => Observable<boolean> | Promise<boolean> | boolean;
}

@Injectable({ providedIn: 'root' })
export class CanDeactivateGuard 
  implements CanDeactivate<CanComponentDeactivate> {
  
  canDeactivate(
    component: CanComponentDeactivate
  ): Observable<boolean> | Promise<boolean> | boolean {
    return component.canDeactivate ? component.canDeactivate() : true;
  }
}

// Component with unsaved changes
@Component({})
export class EditUserComponent implements CanComponentDeactivate {
  hasUnsavedChanges = false;
  
  canDeactivate(): Observable<boolean> {
    if (this.hasUnsavedChanges) {
      return this.dialogService.confirm(
        'You have unsaved changes. Do you want to leave?'
      );
    }
    return of(true);
  }
}

// CanLoad - Can load lazy module
@Injectable({ providedIn: 'root' })
export class CanLoadGuard implements CanLoad {
  canLoad(route: Route): boolean {
    return this.authService.isAuthorized(route.path);
  }
}

// Resolve - Pre-fetch data
@Injectable({ providedIn: 'root' })
export class UserResolve implements Resolve<User> {
  constructor(private userService: UserService) {}
  
  resolve(route: ActivatedRouteSnapshot): Observable<User> {
    return this.userService.getUser(route.params['id']).pipe(
      catchError(() => {
        this.router.navigate(['/not-found']);
        return EMPTY;
      })
    );
  }
}

// Route configuration
{
  path: 'user/:id',
  component: UserDetailComponent,
  resolve: { user: UserResolve },
  canActivate: [AuthGuard],
  canDeactivate: [CanDeactivateGuard]
}

// Component usage
export class UserDetailComponent implements OnInit {
  user: User;
  
  constructor(private route: ActivatedRoute) {}
  
  ngOnInit() {
    // Data already resolved
    this.user = this.route.snapshot.data['user'];
    
    // Or subscribe to changes
    this.route.data.subscribe(data => {
      this.user = data['user'];
    });
  }
}
```

**Accessing Route Parameters:**
```typescript
export class UserDetailComponent implements OnInit {
  userId: string;
  queryParams: any;
  
  constructor(
    private route: ActivatedRouteSnapshot,
    private router: Router
  ) {}
  
  ngOnInit() {
    // Snapshot (one-time read)
    this.userId = this.route.snapshot.params['id'];
    this.queryParams = this.route.snapshot.queryParams;
    const fragment = this.route.snapshot.fragment;
    
    // Observable (reactive)
    this.route.params.subscribe(params => {
      this.userId = params['id'];
    });
    
    this.route.queryParams.subscribe(queryParams => {
      this.queryParams = queryParams;
    });
    
    // paramMap (recommended)
    this.route.paramMap.subscribe(params => {
      this.userId = params.get('id');
    });
  }
  
  // Navigate programmatically
  navigateToUser(id: string) {
    this.router.navigate(['/users', id]);
    
    // With query params and fragment
    this.router.navigate(['/users', id], {
      queryParams: { tab: 'profile' },
      fragment: 'section1',
      queryParamsHandling: 'merge', // preserve existing
      relativeTo: this.route
    });
    
    // Navigate by URL
    this.router.navigateByUrl(`/users/${id}`);
  }
}
```

## Performance Optimization

### 8. Angular Performance

**Q: Describe performance optimization techniques in Angular.**

**A:**

**1. OnPush Change Detection:**
```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div>{{ user.name }}</div>
    <div *ngFor="let item of items; trackBy: trackByFn">
      {{ item.name }}
    </div>
  `
})
export class OptimizedComponent {
  @Input() user: User; // Must pass new reference
  @Input() items: Item[];
  
  trackByFn(index: number, item: Item) {
    return item.id; // Track by unique ID
  }
}
```

**2. Lazy Loading & Code Splitting:**
```typescript
// Route-level code splitting
const routes: Routes = [
  {
    path: 'feature',
    loadChildren: () => import('./feature/feature.module')
      .then(m => m.FeatureModule)
  }
];

// Component-level lazy loading
@Component({
  template: `
    <ng-container *ngComponentOutlet="lazyComponent$ | async">
    </ng-container>
  `
})
export class DynamicComponent {
  lazyComponent$ = from(
    import('./heavy-component').then(m => m.HeavyComponent)
  );
}
```

**3. Virtual Scrolling:**
```typescript
import { ScrollingModule } from '@angular/cdk/scrolling';

@Component({
  template: `
    <cdk-virtual-scroll-viewport itemSize="50" class="viewport">
      <div *cdkVirtualFor="let item of items" class="item">
        {{ item.name }}
      </div>
    </cdk-virtual-scroll-viewport>
  `,
  styles: [`
    .viewport {
      height: 400px;
      width: 100%;
    }
    .item {
      height: 50px;
    }
  `]
})
export class VirtualScrollComponent {
  items = Array.from({ length: 100000 }, (_, i) => ({
    id: i,
    name: `Item ${i}`
  }));
}
```

**4. Pure Pipes:**
```typescript
@Pipe({
  name: 'expensiveCalculation',
  pure: true // Memoizes result
})
export class ExpensivePipe implements PipeTransform {
  transform(value: number): number {
    console.log('Pipe executed'); // Only when input changes
    return value * 2;
  }
}

// Impure pipe (executes every change detection)
@Pipe({
  name: 'impureFilter',
  pure: false
})
export class ImpureFilterPipe implements PipeTransform {
  transform(items: any[], filter: string): any[] {
    return items.filter(item => item.name.includes(filter));
  }
}
```

**5. Detach Change Detection:**
```typescript
@Component({})
export class ManualCDComponent {
  constructor(private cd: ChangeDetectorRef) {
    // Detach from change detection tree
    this.cd.detach();
  }
  
  updateData() {
    this.data = newData;
    // Manually trigger
    this.cd.detectChanges();
  }
}
```

**6. Web Workers:**
```typescript
// Create worker
const worker = new Worker(new URL('./app.worker', import.meta.url));

worker.onmessage = ({ data }) => {
  console.log('Result from worker:', data);
};

worker.postMessage({ numbers: [1, 2, 3, 4, 5] });

// app.worker.ts
addEventListener('message', ({ data }) => {
  const result = data.numbers.reduce((sum, num) => sum + num, 0);
  postMessage(result);
});
```

**7. Preloading Data:**
```typescript
@Injectable({ providedIn: 'root' })
export class DataPreloadService {
  private cache = new Map<string, any>();
  
  preload(key: string, factory: () => Observable<any>) {
    if (!this.cache.has(key)) {
      factory().subscribe(data => this.cache.set(key, data));
    }
  }
  
  get(key: string): any {
    return this.cache.get(key);
  }
}

// App component
export class AppComponent {
  constructor(private preloadService: DataPreloadService) {
    // Preload on app start
    this.preloadService.preload('users', () =>
      this.http.get('/api/users')
    );
  }
}
```

**8. Memoization:**
```typescript
function memoize<T extends (...args: any[]) => any>(fn: T): T {
  const cache = new Map();
  return ((...args: any[]) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn(...args);
    cache.set(key, result);
    return result;
  }) as T;
}

// Usage
const expensiveCalculation = memoize((n: number) => {
  return n * n;
});
```

## Testing

### 9. Unit Testing

**Q: Explain Angular testing strategies.**

**A:**

**Component Testing:**
```typescript
describe('UserComponent', () => {
  let component: UserComponent;
  let fixture: ComponentFixture<UserComponent>;
  let userService: jasmine.SpyObj<UserService>;
  let compiled: HTMLElement;
  
  beforeEach(async () => {
    const userServiceSpy = jasmine.createSpyObj('UserService', 
      ['getUser', 'updateUser']
    );
    
    await TestBed.configureTestingModule({
      declarations: [UserComponent],
      imports: [HttpClientTestingModule, ReactiveFormsModule],
      providers: [
        { provide: UserService, useValue: userServiceSpy }
      ]
    }).compileComponents();
    
    userService = TestBed.inject(UserService) as jasmine.SpyObj<UserService>;
    fixture = TestBed.createComponent(UserComponent);
    component = fixture.componentInstance;
    compiled = fixture.nativeElement;
  });
  
  it('should create', () => {
    expect(component).toBeTruthy();
  });
  
  it('should display user name', () => {
    component.user = { id: '1', name: 'John' };
    fixture.detectChanges();
    
    const nameElement = compiled.querySelector('.user-name');
    expect(nameElement.textContent).toContain('John');
  });
  
  it('should call service on save', () => {
    const user = { id: '1', name: 'John' };
    userService.updateUser.and.returnValue(of(user));
    
    component.saveUser(user);
    
    expect(userService.updateUser).toHaveBeenCalledWith(user);
  });
  
  it('should emit event on delete', () => {
    spyOn(component.userDeleted, 'emit');
    
    component.deleteUser('1');
    
    expect(component.userDeleted.emit).toHaveBeenCalledWith('1');
  });
  
  it('should handle async data', fakeAsync(() => {
    const user = { id: '1', name: 'John' };
    userService.getUser.and.returnValue(of(user).pipe(delay(1000)));
    
    component.loadUser('1');
    tick(1000);
    
    expect(component.user).toEqual(user);
  }));
});
```

**Service Testing:**
```typescript
describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;
  
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService]
    });
    
    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });
  
  afterEach(() => {
    httpMock.verify(); // Ensure no outstanding requests
  });
  
  it('should fetch users', () => {
    const mockUsers = [
      { id: '1', name: 'John' },
      { id: '2', name: 'Jane' }
    ];
    
    service.getUsers().subscribe(users => {
      expect(users).toEqual(mockUsers);
    });
    
    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });
  
  it('should handle error', () => {
    service.getUsers().subscribe(
      () => fail('should have failed'),
      error => expect(error.status).toBe(500)
    );
    
    const req = httpMock.expectOne('/api/users');
    req.flush('Error', { status: 500, statusText: 'Server Error' });
  });
  
  it('should post user with headers', () => {
    const newUser = { name: 'John' };
    
    service.createUser(newUser).subscribe();
    
    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('POST');
    expect(req.request.body).toEqual(newUser);
    expect(req.request.headers.get('Content-Type')).toBe('application/json');
    req.flush({ id: '1', ...newUser });
  });
});
```

**Directive Testing:**
```typescript
@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  @Input() appHighlight: string;
  
  constructor(private el: ElementRef) {}
  
  @HostListener('mouseenter') onMouseEnter() {
    this.highlight(this.appHighlight || 'yellow');
  }
  
  @HostListener('mouseleave') onMouseLeave() {
    this.highlight(null);
  }
  
  private highlight(color: string) {
    this.el.nativeElement.style.backgroundColor = color;
  }
}

describe('HighlightDirective', () => {
  let fixture: ComponentFixture<TestComponent>;
  let des: DebugElement[];
  
  @Component({
    template: `
      <div appHighlight>Default</div>
      <div appHighlight="red">Red</div>
    `
  })
  class TestComponent {}
  
  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [HighlightDirective, TestComponent]
    });
    
    fixture = TestBed.createComponent(TestComponent);
    fixture.detectChanges();
    des = fixture.debugElement.queryAll(By.directive(HighlightDirective));
  });
  
  it('should have 2 highlighted elements', () => {
    expect(des.length).toBe(2);
  });
  
  it('should highlight on mouseenter', () => {
    const el = des[0].nativeElement;
    
    el.dispatchEvent(new Event('mouseenter'));
    expect(el.style.backgroundColor).toBe('yellow');
    
    el.dispatchEvent(new Event('mouseleave'));
    expect(el.style.backgroundColor).toBe('');
  });
  
  it('should use custom color', () => {
    const el = des[1].nativeElement;
    
    el.dispatchEvent(new Event('mouseenter'));
    expect(el.style.backgroundColor).toBe('red');
  });
});
```

**Pipe Testing:**
```typescript
describe('TruncatePipe', () => {
  let pipe: TruncatePipe;
  
  beforeEach(() => {
    pipe = new TruncatePipe();
  });
  
  it('should truncate string', () => {
    expect(pipe.transform('Hello World', 5)).toBe('Hello...');
  });
  
  it('should not truncate if shorter', () => {
    expect(pipe.transform('Hi', 5)).toBe('Hi');
  });
  
  it('should use custom suffix', () => {
    expect(pipe.transform('Hello World', 5, '---')).toBe('Hello---');
  });
});
```

**NgRx Testing:**
```typescript
describe('UserReducer', () => {
  it('should return initial state', () => {
    const action = { type: 'Unknown' };
    const state = userReducer(undefined, action);
    
    expect(state).toEqual(initialState);
  });
  
  it('should set loading on loadUsers', () => {
    const action = UserActions.loadUsers();
    const state = userReducer(initialState, action);
    
    expect(state.loading).toBe(true);
  });
  
  it('should add users on success', () => {
    const users = [{ id: '1', name: 'John' }];
    const action = UserActions.loadUsersSuccess({ users });
    const state = userReducer(initialState, action);
    
    expect(state.entities['1']).toEqual(users[0]);
    expect(state.loading).toBe(false);
  });
});

describe('UserEffects', () => {
  let actions$: Observable<any>;
  let effects: UserEffects;
  let userService: jasmine.SpyObj<UserService>;
  
  beforeEach(() => {
    const serviceSpy = jasmine.createSpyObj('UserService', ['getUsers']);
    
    TestBed.configureTestingModule({
      providers: [
        UserEffects,
        provideMockActions(() => actions$),
        { provide: UserService, useValue: serviceSpy }
      ]
    });
    
    effects = TestBed.inject(UserEffects);
    userService = TestBed.inject(UserService) as jasmine.SpyObj<UserService>;
  });
  
  it('should load users successfully', (done) => {
    const users = [{ id: '1', name: 'John' }];
    const action = UserActions.loadUsers();
    const completion = UserActions.loadUsersSuccess({ users });
    
    actions$ = hot('-a', { a: action });
    const expected = cold('-b', { b: completion });
    userService.getUsers.and.returnValue(of(users));
    
    expect(effects.loadUsers$).toBeObservable(expected);
    done();
  });
});

describe('UserSelectors', () => {
  it('should select all users', () => {
    const state = {
      users: {
        ids: ['1', '2'],
        entities: {
          '1': { id: '1', name: 'John' },
          '2': { id: '2', name: 'Jane' }
        }
      }
    };
    
    const result = selectAllUsers(state);
    expect(result.length).toBe(2);
    expect(result[0].name).toBe('John');
  });
});
```

## Advanced Topics

### 10. Custom Directives & Pipes

**Q: Create advanced custom directives.**

**A:**

**Structural Directive:**
```typescript
@Directive({
  selector: '[appUnless]'
})
export class UnlessDirective {
  private hasView = false;
  
  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef
  ) {}
  
  @Input() set appUnless(condition: boolean) {
    if (!condition && !this.hasView) {
      this.viewContainer.createEmbeddedView(this.templateRef);
      this.hasView = true;
    } else if (condition && this.hasView) {
      this.viewContainer.clear();
      this.hasView = false;
    }
  }
}

// Usage: <div *appUnless="condition">Content</div>
```

**Advanced Structural Directive with Context:**
```typescript
export interface RangeContext {
  $implicit: number;
  index: number;
  first: boolean;
  last: boolean;
  even: boolean;
  odd: boolean;
}

@Directive({
  selector: '[appRange]'
})
export class RangeDirective {
  constructor(
    private templateRef: TemplateRef<RangeContext>,
    private viewContainer: ViewContainerRef
  ) {}
  
  @Input() set appRange(count: number) {
    this.viewContainer.clear();
    
    for (let i = 0; i < count; i++) {
      this.viewContainer.createEmbeddedView(this.templateRef, {
        $implicit: i,
        index: i,
        first: i === 0,
        last: i === count - 1,
        even: i % 2 === 0,
        odd: i % 2 !== 0
      });
    }
  }
}

// Usage:
// <div *appRange="5; let num; let i = index; let isFirst = first">
//   {{ num }} - {{ i }} - {{ isFirst }}
// </div>
```

**Attribute Directive with Renderer2:**
```typescript
@Directive({
  selector: '[appDropdown]',
  exportAs: 'appDropdown'
})
export class DropdownDirective {
  @Input() dropdownClass = 'open';
  @Output() dropdownToggled = new EventEmitter<boolean>();
  
  private isOpen = false;
  
  constructor(
    private el: ElementRef,
    private renderer: Renderer2
  ) {}
  
  @HostListener('click', ['$event'])
  onClick(event: Event) {
    event.stopPropagation();
    this.toggle();
  }
  
  @HostListener('document:click')
  onDocumentClick() {
    if (this.isOpen) {
      this.close();
    }
  }
  
  toggle() {
    this.isOpen ? this.close() : this.open();
  }
  
  open() {
    this.renderer.addClass(this.el.nativeElement, this.dropdownClass);
    this.isOpen = true;
    this.dropdownToggled.emit(true);
  }
  
  close() {
    this.renderer.removeClass(this.el.nativeElement, this.dropdownClass);
    this.isOpen = false;
    this.dropdownToggled.emit(false);
  }
}
```

**Async Validator Directive:**
```typescript
@Directive({
  selector: '[appUniqueUsername]',
  providers: [{
    provide: NG_ASYNC_VALIDATORS,
    useExisting: UniqueUsernameValidatorDirective,
    multi: true
  }]
})
export class UniqueUsernameValidatorDirective implements AsyncValidator {
  constructor(private userService: UserService) {}
  
  validate(control: AbstractControl): Observable<ValidationErrors | null> {
    if (!control.value) {
      return of(null);
    }
    
    return this.userService.checkUsername(control.value).pipe(
      map(exists => exists ? { usernameTaken: true } : null),
      catchError(() => of(null))
    );
  }
}
```

**Advanced Pipes:**
```typescript
// Memoized Pipe
@Pipe({
  name: 'memoize',
  pure: true
})
export class MemoizePipe implements PipeTransform {
  private cache = new Map<string, any>();
  
  transform(fn: Function, ...args: any[]): any {
    const key = JSON.stringify(args);
    
    if (this.cache.has(key)) {
      return this.cache.get(key);
    }
    
    const result = fn(...args);
    this.cache.set(key, result);
    return result;
  }
}

// Async Pipe with Loading State
@Pipe({
  name: 'withLoading'
})
export class WithLoadingPipe implements PipeTransform {
  transform(
    observable: Observable<any>
  ): Observable<{ loading: boolean; value: any; error: any }> {
    return observable.pipe(
      map(value => ({ loading: false, value, error: null })),
      startWith({ loading: true, value: null, error: null }),
      catchError(error => of({ loading: false, value: null, error }))
    );
  }
}

// Usage: <div *ngIf="data$ | withLoading | async as state">
//   <div *ngIf="state.loading">Loading...</div>
//   <div *ngIf="state.error">Error: {{ state.error }}</div>
//   <div *ngIf="state.value">{{ state.value }}</div>
// </div>

// Filter Pipe with Highlight
@Pipe({
  name: 'filterHighlight'
})
export class FilterHighlightPipe implements PipeTransform {
  constructor(private sanitizer: DomSanitizer) {}
  
  transform(items: any[], searchText: string, field: string): any[] {
    if (!items || !searchText) {
      return items;
    }
    
    const search = searchText.toLowerCase();
    
    return items.filter(item => {
      const value = item[field].toLowerCase();
      if (value.includes(search)) {
        const highlighted = value.replace(
          new RegExp(search, 'gi'),
          match => `<mark>${match}</mark>`
        );
        item[`${field}Highlighted`] = this.sanitizer.sanitize(
          SecurityContext.HTML,
          highlighted
        );
        return true;
      }
      return false;
    });
  }
}
```

## HTTP & Interceptors

### 11. HTTP Client & Interceptors

**Q: Explain HTTP interceptors and their use cases.**

**A:**

**Authentication Interceptor:**
```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private authService: AuthService) {}
  
  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();
    
    if (token) {
      const cloned = req.clone({
        headers: req.headers.set('Authorization', `Bearer ${token}`)
      });
      return next.handle(cloned);
    }
    
    return next.handle(req);
  }
}
```

**Error Handling Interceptor:**
```typescript
@Injectable()
export class ErrorInterceptor implements HttpInterceptor {
  constructor(
    private router: Router,
    private notificationService: NotificationService
  ) {}
  
  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      catchError((error: HttpErrorResponse) => {
        let errorMessage = 'An error occurred';
        
        if (error.error instanceof ErrorEvent) {
          // Client-side error
          errorMessage = error.error.message;
        } else {
          // Server-side error
          switch (error.status) {
            case 401:
              this.router.navigate(['/login']);
              errorMessage = 'Unauthorized';
              break;
            case 403:
              errorMessage = 'Forbidden';
              break;
            case 404:
              errorMessage = 'Not found';
              break;
            case 500:
              errorMessage = 'Server error';
              break;
            default:
              errorMessage = error.message;
          }
        }
        
        this.notificationService.showError(errorMessage);
        return throwError(() => error);
      })
    );
  }
}
```

**Caching Interceptor:**
```typescript
@Injectable()
export class CachingInterceptor implements HttpInterceptor {
  private cache = new Map<string, HttpResponse<any>>();
  
  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    // Only cache GET requests
    if (req.method !== 'GET') {
      return next.handle(req);
    }
    
    // Check if cached
    const cachedResponse = this.cache.get(req.url);
    if (cachedResponse) {
      return of(cachedResponse.clone());
    }
    
    // Make request and cache
    return next.handle(req).pipe(
      tap(event => {
        if (event instanceof HttpResponse) {
          this.cache.set(req.url, event.clone());
        }
      })
    );
  }
}
```

**Loading Interceptor:**
```typescript
@Injectable()
export class LoadingInterceptor implements HttpInterceptor {
  private requests: HttpRequest<any>[] = [];
  
  constructor(private loadingService: LoadingService) {}
  
  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    this.requests.push(req);
    this.loadingService.setLoading(true);
    
    return next.handle(req).pipe(
      finalize(() => {
        this.requests = this.requests.filter(r => r !== req);
        if (this.requests.length === 0) {
          this.loadingService.setLoading(false);
        }
      })
    );
  }
}
```

**Retry Interceptor:**
```typescript
@Injectable()
export class RetryInterceptor implements HttpInterceptor {
  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    const maxRetries = 3;
    const delayMs = 1000;
    
    return next.handle(req).pipe(
      retryWhen(errors =>
        errors.pipe(
          concatMap((error, index) => {
            if (index >= maxRetries || error.status < 500) {
              return throwError(() => error);
            }
            
            return timer(delayMs * Math.pow(2, index));
          })
        )
      )
    );
  }
}
```

**Logging Interceptor:**
```typescript
@Injectable()
export class LoggingInterceptor implements HttpInterceptor {
  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    const started = Date.now();
    
    return next.handle(req).pipe(
      tap({
        next: event => {
          if (event instanceof HttpResponse) {
            const elapsed = Date.now() - started;
            console.log(`${req.method} ${req.url} completed in ${elapsed}ms`);
          }
        },
        error: error => {
          const elapsed = Date.now() - started;
          console.error(`${req.method} ${req.url} failed after ${elapsed}ms`);
        }
      })
    );
  }
}
```

**Registering Interceptors:**
```typescript
@NgModule({
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptor,
      multi: true
    },
    {
      provide: HTTP_INTERCEPTORS,
      useClass: ErrorInterceptor,
      multi: true
    },
    {
      provide: HTTP_INTERCEPTORS,
      useClass: LoadingInterceptor,
      multi: true
    }
  ]
})
export class AppModule { }
```

## Coding Challenges

### Challenge 1: Autocomplete Component

```typescript
@Component({
  selector: 'app-autocomplete',
  template: `
    <div class="autocomplete">
      <input
        type="text"
        [formControl]="searchControl"
        placeholder="Search..."
        (focus)="onFocus()"
        (blur)="onBlur()">
      
      <ul class="suggestions" *ngIf="showSuggestions && (results$ | async) as results">
        <li *ngFor="let result of results; let i = index"
            [class.active]="i === selectedIndex"
            (mousedown)="selectResult(result)"
            (mouseenter)="selectedIndex = i">
          {{ result[displayField] }}
        </li>
      </ul>
    </div>
  `,
  styles: [`
    .autocomplete {
      position: relative;
    }
    .suggestions {
      position: absolute;
      top: 100%;
      left: 0;
      right: 0;
      max-height: 200px;
      overflow-y: auto;
      background: white;
      border: 1px solid #ccc;
      list-style: none;
      margin: 0;
      padding: 0;
      z-index: 1000;
    }
    .suggestions li {
      padding: 8px 12px;
      cursor: pointer;
    }
    .suggestions li:hover,
    .suggestions li.active {
      background: #f0f0f0;
    }
  `]
})
export class AutocompleteComponent implements OnInit {
  @Input() searchFn: (term: string) => Observable<any[]>;
  @Input() displayField = 'name';
  @Input() minLength = 2;
  @Input() debounceTime = 300;
  @Output() selected = new EventEmitter<any>();
  
  searchControl = new FormControl('');
  results$: Observable<any[]>;
  showSuggestions = false;
  selectedIndex = -1;
  
  ngOnInit() {
    this.results$ = this.searchControl.valueChanges.pipe(
      debounceTime(this.debounceTime),
      distinctUntilChanged(),
      filter(term => term?.length >= this.minLength),
      switchMap(term => this.searchFn(term)),
      tap(() => this.selectedIndex = -1)
    );
  }
  
  @HostListener('document:keydown', ['$event'])
  handleKeydown(event: KeyboardEvent) {
    if (!this.showSuggestions) return;
    
    if (event.key === 'ArrowDown') {
      event.preventDefault();
      this.selectedIndex++;
    } else if (event.key === 'ArrowUp') {
      event.preventDefault();
      this.selectedIndex = Math.max(0, this.selectedIndex - 1);
    } else if (event.key === 'Enter' && this.selectedIndex >= 0) {
      event.preventDefault();
      this.results$.pipe(take(1)).subscribe(results => {
        if (results[this.selectedIndex]) {
          this.selectResult(results[this.selectedIndex]);
        }
      });
    } else if (event.key === 'Escape') {
      this.showSuggestions = false;
    }
  }
  
  onFocus() {
    this.showSuggestions = true;
  }
  
  onBlur() {
    // Delay to allow click event to fire
    setTimeout(() => this.showSuggestions = false, 200);
  }
  
  selectResult(result: any) {
    this.searchControl.setValue(result[this.displayField], { emitEvent: false });
    this.selected.emit(result);
    this.showSuggestions = false;
  }
}
```

### Challenge 2: Infinite Scroll Directive

```typescript
@Directive({
  selector: '[appInfiniteScroll]'
})
export class InfiniteScrollDirective implements OnInit, OnDestroy {
  @Input() scrollThreshold = 200;
  @Input() scrollDebounce = 200;
  @Output() scrolled = new EventEmitter<void>();
  
  private destroy$ = new Subject<void>();
  
  constructor(private el: ElementRef) {}
  
  ngOnInit() {
    fromEvent(this.el.nativeElement, 'scroll')
      .pipe(
        debounceTime(this.scrollDebounce),
        map(() => this.isScrolledToBottom()),
        filter(isBottom => isBottom),
        takeUntil(this.destroy$)
      )
      .subscribe(() => this.scrolled.emit());
  }
  
  private isScrolledToBottom(): boolean {
    const element = this.el.nativeElement;
    const scrollPosition = element.scrollTop + element.clientHeight;
    const scrollHeight = element.scrollHeight;
    
    return scrollPosition >= scrollHeight - this.scrollThreshold;
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// Usage
@Component({
  template: `
    <div class="scroll-container"
         appInfiniteScroll
         (scrolled)="onScrolled()">
      <div *ngFor="let item of items">{{ item }}</div>
      <div *ngIf="loading">Loading...</div>
    </div>
  `
})
export class ListComponent {
  items = [];
  loading = false;
  page = 1;
  
  onScrolled() {
    if (!this.loading) {
      this.loadMore();
    }
  }
  
  loadMore() {
    this.loading = true;
    this.dataService.getItems(this.page).subscribe(newItems => {
      this.items.push(...newItems);
      this.page++;
      this.loading = false;
    });
  }
}
```

### Challenge 3: Dynamic Form Builder

```typescript
export interface FormField {
  type: 'text' | 'email' | 'number' | 'select' | 'checkbox' | 'date';
  key: string;
  label: string;
  value?: any;
  required?: boolean;
  options?: { label: string; value: any }[];
  validators?: ValidatorFn[];
  asyncValidators?: AsyncValidatorFn[];
}

@Injectable({ providedIn: 'root' })
export class DynamicFormService {
  constructor(private fb: FormBuilder) {}
  
  createForm(fields: FormField[]): FormGroup {
    const group: any = {};
    
    fields.forEach(field => {
      const validators = [
        ...(field.required ? [Validators.required] : []),
        ...(field.validators || [])
      ];
      
      group[field.key] = [
        field.value || this.getDefaultValue(field.type),
        validators,
        field.asyncValidators || []
      ];
    });
    
    return this.fb.group(group);
  }
  
  private getDefaultValue(type: string): any {
    switch (type) {
      case 'checkbox': return false;
      case 'number': return 0;
      default: return '';
    }
  }
}

@Component({
  selector: 'app-dynamic-form',
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <div *ngFor="let field of fields" class="form-field">
        <label>{{ field.label }}</label>
        
        <input *ngIf="field.type === 'text' || field.type === 'email'"
               [type]="field.type"
               [formControlName]="field.key">
        
        <input *ngIf="field.type === 'number'"
               type="number"
               [formControlName]="field.key">
        
        <input *ngIf="field.type === 'date'"
               type="date"
               [formControlName]="field.key">
        
        <select *ngIf="field.type === 'select'"
                [formControlName]="field.key">
          <option *ngFor="let opt of field.options"
                  [value]="opt.value">
            {{ opt.label }}
          </option>
        </select>
        
        <input *ngIf="field.type === 'checkbox'"
               type="checkbox"
               [formControlName]="field.key">
        
        <div class="errors" *ngIf="form.get(field.key).invalid && form.get(field.key).touched">
          <span *ngIf="form.get(field.key).hasError('required')">
            {{ field.label }} is required
          </span>
          <span *ngIf="form.get(field.key).hasError('email')">
            Invalid email format
          </span>
        </div>
      </div>
      
      <button type="submit" [disabled]="form.invalid">Submit</button>
    </form>
  `
})
export class DynamicFormComponent implements OnInit {
  @Input() fields: FormField[];
  @Output() formSubmit = new EventEmitter<any>();
  
  form: FormGroup;
  
  constructor(private formService: DynamicFormService) {}
  
  ngOnInit() {
    this.form = this.formService.createForm(this.fields);
  }
  
  onSubmit() {
    if (this.form.valid) {
      this.formSubmit.emit(this.form.value);
    }
  }
}
```

### Challenge 4: Reusable Data Table

```typescript
@Component({
  selector: 'app-data-table',
  template: `
    <div class="table-container">
      <table>
        <thead>
          <tr>
            <th *ngFor="let column of columns"
                (click)="sort(column.key)"
                [class.sortable]="column.sortable">
              {{ column.label }}
              <span *ngIf="sortKey === column.key">
                {{ sortDirection === 'asc' ? '▲' : '▼' }}
              </span>
            </th>
            <th *ngIf="actions.length">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr *ngFor="let row of sortedData">
            <td *ngFor="let column of columns">
              <ng-container *ngIf="column.template; else defaultCell">
                <ng-container
                  *ngTemplateOutlet="column.template; context: { $implicit: row }">
                </ng-container>
              </ng-container>
              <ng-template #defaultCell>
                {{ row[column.key] }}
              </ng-template>
            </td>
            <td *ngIf="actions.length">
              <button *ngFor="let action of actions"
                      (click)="action.handler(row)"
                      [class]="action.class">
                {{ action.label }}
              </button>
            </td>
          </tr>
        </tbody>
      </table>
      
      <div class="pagination" *ngIf="pagination">
        <button (click)="goToPage(currentPage - 1)"
                [disabled]="currentPage === 1">
          Previous
        </button>
        <span>Page {{ currentPage }} of {{ totalPages }}</span>
        <button (click)="goToPage(currentPage + 1)"
                [disabled]="currentPage === totalPages">
          Next
        </button>
      </div>
    </div>
  `,
  styles: [`
    table {
      width: 100%;
      border-collapse: collapse;
    }
    th, td {
      padding: 12px;
      text-align: left;
      border-bottom: 1px solid #ddd;
    }
    th.sortable {
      cursor: pointer;
      user-select: none;
    }
    .pagination {
      margin-top: 20px;
      display: flex;
      justify-content: center;
      gap: 10px;
      align-items: center;
    }
  `]
})
export class DataTableComponent<T> {
  @Input() data: T[];
  @Input() columns: TableColumn<T>[];
  @Input() actions: TableAction<T>[] = [];
  @Input() pagination: boolean = false;
  @Input() pageSize: number = 10;
  
  sortKey: string;
  sortDirection: 'asc' | 'desc' = 'asc';
  currentPage = 1;
  
  get sortedData(): T[] {
    let result = [...this.data];
    
    if (this.sortKey) {
      result.sort((a, b) => {
        const aVal = a[this.sortKey];
        const bVal = b[this.sortKey];
        const comparison = aVal > bVal ? 1 : aVal < bVal ? -1 : 0;
        return this.sortDirection === 'asc' ? comparison : -comparison;
      });
    }
    
    if (this.pagination) {
      const start = (this.currentPage - 1) * this.pageSize;
      result = result.slice(start, start + this.pageSize);
    }
    
    return result;
  }
  
  get totalPages(): number {
    return Math.ceil(this.data.length / this.pageSize);
  }
  
  sort(key: string) {
    if (this.sortKey === key) {
      this.sortDirection = this.sortDirection === 'asc' ? 'desc' : 'asc';
    } else {
      this.sortKey = key;
      this.sortDirection = 'asc';
    }
  }
  
  goToPage(page: number) {
    if (page >= 1 && page <= this.totalPages) {
      this.currentPage = page;
    }
  }
}

export interface TableColumn<T> {
  key: keyof T;
  label: string;
  sortable?: boolean;
  template?: TemplateRef<any>;
}

export interface TableAction<T> {
  label: string;
  handler: (row: T) => void;
  class?: string;
}

// Usage
@Component({
  template: `
    <app-data-table
      [data]="users"
      [columns]="columns"
      [actions]="actions"
      [pagination]="true"
      [pageSize]="5">
    </app-data-table>
    
    <ng-template #nameTemplate let-user>
      <strong>{{ user.name }}</strong>
    </ng-template>
  `
})
export class UserListComponent {
  @ViewChild('nameTemplate') nameTemplate: TemplateRef<any>;
  
  users = [
    { id: 1, name: 'John', email: 'john@example.com', active: true },
    { id: 2, name: 'Jane', email: 'jane@example.com', active: false }
  ];
  
  columns: TableColumn<User>[];
  actions: TableAction<User>[];
  
  ngAfterViewInit() {
    this.columns = [
      { key: 'id', label: 'ID', sortable: true },
      { key: 'name', label: 'Name', sortable: true, template: this.nameTemplate },
      { key: 'email', label: 'Email', sortable: true },
      { key: 'active', label: 'Active' }
    ];
    
    this.actions = [
      { label: 'Edit', handler: user => this.editUser(user) },
      { label: 'Delete', handler: user => this.deleteUser(user), class: 'danger' }
    ];
  }
  
  editUser(user: User) {
    console.log('Edit', user);
  }
  
  deleteUser(user: User) {
    console.log('Delete', user);
  }
}
```

---

*These comprehensive answers demonstrate deep Angular knowledge, practical problem-solving abilities, and the kind of nuanced understanding expected from a senior engineer with 15+ years of experience.*