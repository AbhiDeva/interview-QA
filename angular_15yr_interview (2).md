# Angular UI Developer Interview Questions & Answers
## For 15 Years of Experience

### 1. How has your approach to Angular architecture evolved over the years?

Over 15 years, I've witnessed the evolution from AngularJS to modern Angular. My architecture approach has shifted from monolithic applications to micro-frontends, from two-way binding patterns to reactive programming with RxJS. I now prioritize modular architecture using standalone components (Angular 14+), lazy loading strategies, and design patterns like state management with NgRx or Akita. I focus heavily on scalability, performance optimization through OnPush change detection, and maintaining clean separation of concerns with smart/dumb component patterns.

**Example: Smart/Dumb Component Pattern**
```typescript
// Smart Component (Container)
@Component({
  selector: 'app-user-list-container',
  template: `
    <app-user-list 
      [users]="users$ | async"
      [loading]="loading$ | async"
      (userSelected)="onUserSelected($event)"
      (deleteUser)="onDeleteUser($event)">
    </app-user-list>
  `
})
export class UserListContainerComponent {
  users$ = this.store.select(selectUsers);
  loading$ = this.store.select(selectLoading);

  constructor(private store: Store) {}

  onUserSelected(user: User): void {
    this.store.dispatch(UserActions.selectUser({ user }));
  }

  onDeleteUser(userId: string): void {
    this.store.dispatch(UserActions.deleteUser({ userId }));
  }
}

// Dumb Component (Presentational)
@Component({
  selector: 'app-user-list',
  template: `
    <div *ngIf="loading">Loading...</div>
    <div *ngFor="let user of users; trackBy: trackByUserId">
      <span (click)="userSelected.emit(user)">{{ user.name }}</span>
      <button (click)="deleteUser.emit(user.id)">Delete</button>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserListComponent {
  @Input() users: User[] = [];
  @Input() loading = false;
  @Output() userSelected = new EventEmitter<User>();
  @Output() deleteUser = new EventEmitter<string>();

  trackByUserId(index: number, user: User): string {
    return user.id;
  }
}
```

### 2. Explain the difference between AngularJS and modern Angular frameworks.

AngularJS (1.x) was JavaScript-based using MVC pattern with two-way data binding via dirty checking. Modern Angular (2+) is TypeScript-based, component-oriented, uses Zone.js for change detection, has a more performant rendering engine, better mobile support, and improved dependency injection. AngularJS used controllers and $scope, while modern Angular uses components and decorators. The migration from AngularJS required complete rewrites, though the Angular team provided ngUpgrade for hybrid applications.

### 3. What are standalone components and why are they significant?

Introduced in Angular 14 and stable in Angular 15, standalone components eliminate the need for NgModules. They're self-contained with imports, declarations, and providers defined within the component decorator. This simplifies the application structure, reduces boilerplate, improves tree-shaking, and makes components more portable and testable. For large-scale applications, this represents a fundamental shift toward a more streamlined and modern development experience.

**Example: Standalone Component**
```typescript
// Standalone component with imports
@Component({
  selector: 'app-product-card',
  standalone: true,
  imports: [CommonModule, MatCardModule, MatButtonModule, RouterLink],
  template: `
    <mat-card>
      <mat-card-header>
        <mat-card-title>{{ product.name }}</mat-card-title>
      </mat-card-header>
      <mat-card-content>
        <p>{{ product.description }}</p>
        <p class="price">{{ product.price | currency }}</p>
      </mat-card-content>
      <mat-card-actions>
        <button mat-button [routerLink]="['/products', product.id]">
          View Details
        </button>
      </mat-card-actions>
    </mat-card>
  `,
  styles: [`
    .price { font-weight: bold; color: #1976d2; }
  `]
})
export class ProductCardComponent {
  @Input() product!: Product;
}

// Bootstrapping standalone app
bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
    provideAnimations(),
    importProvidersFrom(StoreModule.forRoot(reducers))
  ]
});
```

### 4. How do you implement micro-frontend architecture with Angular?

I've implemented micro-frontends using Module Federation (Webpack 5), creating independent Angular applications that can be integrated at runtime. Each micro-frontend has its own repository, build pipeline, and deployment cycle. Key considerations include shared dependencies management, cross-application communication through custom events or shared services, routing strategies, and maintaining consistent UI/UX. I also use single-spa framework for framework-agnostic orchestration when integrating Angular with other frameworks.

### 5. Describe advanced RxJS patterns you use regularly.

I extensively use operators like switchMap for canceling previous requests, combineLatest for synchronizing multiple streams, shareReplay for multicast caching, debounceTime for search optimization, and catchError for graceful error handling. I implement custom operators for business logic reusability, use Subject variants (BehaviorSubject, ReplaySubject) appropriately, and apply backpressure strategies. Memory leak prevention through proper unsubscription using takeUntil pattern or async pipe is critical in production applications.

### 6. What's your strategy for state management in enterprise Angular applications?

For complex applications, I use NgRx following Redux principles with actions, reducers, effects, and selectors. I implement entity adapters for normalized state, use facade pattern to hide complexity from components, and leverage NgRx Component Store for local component state. For medium complexity, I might use Akita or services with BehaviorSubjects. The key is choosing the right tool based on application complexity, team expertise, and maintainability requirements.

**Example: NgRx Implementation**
```typescript
// State interface
export interface UserState extends EntityState<User> {
  selectedUserId: string | null;
  loading: boolean;
  error: string | null;
}

// Actions
export const UserActions = createActionGroup({
  source: 'User',
  events: {
    'Load Users': emptyProps(),
    'Load Users Success': props<{ users: User[] }>(),
    'Load Users Failure': props<{ error: string }>(),
    'Select User': props<{ userId: string }>(),
    'Update User': props<{ user: User }>(),
    'Delete User': props<{ userId: string }>()
  }
});

// Reducer with Entity Adapter
const adapter = createEntityAdapter<User>();
const initialState: UserState = adapter.getInitialState({
  selectedUserId: null,
  loading: false,
  error: null
});

export const userReducer = createReducer(
  initialState,
  on(UserActions.loadUsers, (state) => ({
    ...state,
    loading: true,
    error: null
  })),
  on(UserActions.loadUsersSuccess, (state, { users }) =>
    adapter.setAll(users, { ...state, loading: false })
  ),
  on(UserActions.loadUsersFailure, (state, { error }) => ({
    ...state,
    loading: false,
    error
  })),
  on(UserActions.selectUser, (state, { userId }) => ({
    ...state,
    selectedUserId: userId
  })),
  on(UserActions.updateUser, (state, { user }) =>
    adapter.updateOne({ id: user.id, changes: user }, state)
  ),
  on(UserActions.deleteUser, (state, { userId }) =>
    adapter.removeOne(userId, state)
  )
);

// Effects
@Injectable()
export class UserEffects {
  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UserActions.loadUsers),
      exhaustMap(() =>
        this.userService.getUsers().pipe(
          map(users => UserActions.loadUsersSuccess({ users })),
          catchError(error => 
            of(UserActions.loadUsersFailure({ 
              error: error.message 
            }))
          )
        )
      )
    )
  );

  constructor(
    private actions$: Actions,
    private userService: UserService
  ) {}
}

// Selectors
export const selectUserState = createFeatureSelector<UserState>('users');

const { selectAll, selectEntities } = adapter.getSelectors();

export const selectAllUsers = createSelector(
  selectUserState,
  selectAll
);

export const selectUserEntities = createSelector(
  selectUserState,
  selectEntities
);

export const selectSelectedUser = createSelector(
  selectUserEntities,
  selectUserState,
  (entities, state) => 
    state.selectedUserId ? entities[state.selectedUserId] : null
);

export const selectLoading = createSelector(
  selectUserState,
  state => state.loading
);

// Facade Service
@Injectable()
export class UserFacade {
  users$ = this.store.select(selectAllUsers);
  selectedUser$ = this.store.select(selectSelectedUser);
  loading$ = this.store.select(selectLoading);

  constructor(private store: Store) {}

  loadUsers(): void {
    this.store.dispatch(UserActions.loadUsers());
  }

  selectUser(userId: string): void {
    this.store.dispatch(UserActions.selectUser({ userId }));
  }

  updateUser(user: User): void {
    this.store.dispatch(UserActions.updateUser({ user }));
  }
}
```

### 7. How do you optimize Angular application performance?

Performance optimization involves multiple layers: implementing OnPush change detection strategy, lazy loading modules and routes, using trackBy with ngFor, optimizing bundle sizes with tree-shaking and code splitting, implementing virtual scrolling for large lists, using pure pipes, preloading strategies, service workers for caching, CDN for static assets, and monitoring with tools like Lighthouse and webpack-bundle-analyzer. I also use Web Workers for CPU-intensive tasks.

**Example: Performance Optimization Techniques**
```typescript
// 1. OnPush Change Detection with Immutable Updates
@Component({
  selector: 'app-product-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div *ngFor="let product of products; trackBy: trackByProductId">
      {{ product.name }} - {{ product.price | currency }}
    </div>
  `
})
export class ProductListComponent {
  @Input() products: Product[] = [];

  trackByProductId(index: number, product: Product): string {
    return product.id;
  }
}

// 2. Virtual Scrolling for Large Lists
@Component({
  selector: 'app-virtual-list',
  template: `
    <cdk-virtual-scroll-viewport itemSize="50" class="viewport">
      <div *cdkVirtualFor="let item of items; trackBy: trackById" 
           class="item">
        {{ item.name }}
      </div>
    </cdk-virtual-scroll-viewport>
  `,
  styles: [`
    .viewport { height: 400px; }
    .item { height: 50px; }
  `]
})
export class VirtualListComponent {
  items = Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    name: `Item ${i}`
  }));

  trackById(index: number, item: any): number {
    return item.id;
  }
}

// 3. Lazy Loading Routes
const routes: Routes = [
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.module')
      .then(m => m.DashboardModule)
  },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module')
      .then(m => m.AdminModule),
    canLoad: [AuthGuard]
  }
];

// 4. Preloading Strategy
export class CustomPreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    // Preload routes with data.preload = true
    return route.data && route.data['preload'] ? load() : of(null);
  }
}

@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      preloadingStrategy: CustomPreloadingStrategy
    })
  ]
})
export class AppRoutingModule {}

// 5. Pure Pipe for Expensive Calculations
@Pipe({
  name: 'expensiveFilter',
  pure: true // Recalculates only when inputs change
})
export class ExpensiveFilterPipe implements PipeTransform {
  transform(items: any[], filterBy: string): any[] {
    if (!items || !filterBy) return items;
    
    return items.filter(item => 
      item.name.toLowerCase().includes(filterBy.toLowerCase())
    );
  }
}

// 6. Web Worker for CPU-Intensive Tasks
// app.worker.ts
addEventListener('message', ({ data }) => {
  const result = performHeavyCalculation(data);
  postMessage(result);
});

function performHeavyCalculation(data: number[]): number {
  return data.reduce((sum, val) => sum + Math.sqrt(val), 0);
}

// Component using Web Worker
@Component({
  selector: 'app-calculator'
})
export class CalculatorComponent {
  processData(data: number[]): void {
    if (typeof Worker !== 'undefined') {
      const worker = new Worker(new URL('./app.worker', import.meta.url));
      
      worker.onmessage = ({ data }) => {
        console.log('Result:', data);
        worker.terminate();
      };
      
      worker.postMessage(data);
    }
  }
}

// 7. Memoization Decorator
export function Memoize() {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;
    const cache = new Map();

    descriptor.value = function (...args: any[]) {
      const key = JSON.stringify(args);
      
      if (cache.has(key)) {
        return cache.get(key);
      }
      
      const result = originalMethod.apply(this, args);
      cache.set(key, result);
      return result;
    };
    
    return descriptor;
  };
}

@Injectable()
export class DataService {
  @Memoize()
  expensiveCalculation(input: number): number {
    // Expensive operation cached per input
    return input * Math.random();
  }
}
```

### 8. Explain your approach to testing Angular applications.

I follow the testing pyramid with unit tests (Jasmine/Jest), integration tests, and E2E tests (Cypress/Playwright). For components, I test inputs/outputs, user interactions, and state changes. Services are tested with mock dependencies using jasmine spies or jest mocks. I aim for meaningful coverage over percentage targets, use TestBed for component testing, implement page objects for E2E tests, and integrate testing in CI/CD pipelines. Snapshot testing helps catch unintended UI changes.

**Example: Comprehensive Testing Strategy**
```typescript
// 1. Component Unit Test
describe('UserListComponent', () => {
  let component: UserListComponent;
  let fixture: ComponentFixture<UserListComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [UserListComponent],
      imports: [CommonModule],
      providers: []
    }).compileComponents();

    fixture = TestBed.createComponent(UserListComponent);
    component = fixture.componentInstance;
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should display users when provided', () => {
    const mockUsers: User[] = [
      { id: '1', name: 'John Doe', email: 'john@example.com' },
      { id: '2', name: 'Jane Smith', email: 'jane@example.com' }
    ];

    component.users = mockUsers;
    fixture.detectChanges();

    const compiled = fixture.nativeElement;
    const userElements = compiled.querySelectorAll('.user-item');
    
    expect(userElements.length).toBe(2);
    expect(userElements[0].textContent).toContain('John Doe');
  });

  it('should emit userSelected event when user is clicked', () => {
    const mockUser: User = { 
      id: '1', 
      name: 'John', 
      email: 'john@example.com' 
    };
    
    spyOn(component.userSelected, 'emit');
    
    component.onUserClick(mockUser);
    
    expect(component.userSelected.emit).toHaveBeenCalledWith(mockUser);
  });

  it('should track users by id', () => {
    const user: User = { id: '123', name: 'Test', email: 'test@example.com' };
    const trackByResult = component.trackByUserId(0, user);
    
    expect(trackByResult).toBe('123');
  });
});

// 2. Service Test with HTTP Mock
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
    httpMock.verify();
  });

  it('should fetch users successfully', () => {
    const mockUsers: User[] = [
      { id: '1', name: 'John', email: 'john@example.com' }
    ];

    service.getUsers().subscribe(users => {
      expect(users.length).toBe(1);
      expect(users).toEqual(mockUsers);
    });

    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });

  it('should handle error when fetching users fails', () => {
    const errorMessage = 'Server error';

    service.getUsers().subscribe({
      next: () => fail('should have failed'),
      error: (error) => {
        expect(error.status).toBe(500);
      }
    });

    const req = httpMock.expectOne('/api/users');
    req.flush(errorMessage, { status: 500, statusText: 'Server Error' });
  });
});

// 3. NgRx Effect Test
describe('UserEffects', () => {
  let actions$: Observable<Action>;
  let effects: UserEffects;
  let userService: jasmine.SpyObj<UserService>;

  beforeEach(() => {
    const userServiceSpy = jasmine.createSpyObj('UserService', ['getUsers']);

    TestBed.configureTestingModule({
      providers: [
        UserEffects,
        provideMockActions(() => actions$),
        { provide: UserService, useValue: userServiceSpy }
      ]
    });

    effects = TestBed.inject(UserEffects);
    userService = TestBed.inject(UserService) as jasmine.SpyObj<UserService>;
  });

  it('should load users successfully', (done) => {
    const mockUsers: User[] = [{ id: '1', name: 'John', email: 'john@example.com' }];
    const action = UserActions.loadUsers();
    const completion = UserActions.loadUsersSuccess({ users: mockUsers });

    actions$ = of(action);
    userService.getUsers.and.returnValue(of(mockUsers));

    effects.loadUsers$.subscribe(result => {
      expect(result).toEqual(completion);
      done();
    });
  });

  it('should handle load users failure', (done) => {
    const error = new Error('Failed to load');
    const action = UserActions.loadUsers();
    const completion = UserActions.loadUsersFailure({ 
      error: error.message 
    });

    actions$ = of(action);
    userService.getUsers.and.returnValue(throwError(() => error));

    effects.loadUsers$.subscribe(result => {
      expect(result).toEqual(completion);
      done();
    });
  });
});

// 4. Cypress E2E Test with Page Object
// user-list.po.ts
export class UserListPage {
  visit(): void {
    cy.visit('/users');
  }

  getUserItems() {
    return cy.get('[data-cy=user-item]');
  }

  clickUser(index: number): void {
    this.getUserItems().eq(index).click();
  }

  searchUser(query: string): void {
    cy.get('[data-cy=search-input]').type(query);
  }

  getDeleteButton(index: number) {
    return cy.get('[data-cy=delete-button]').eq(index);
  }
}

// user-list.spec.ts
describe('User List', () => {
  const page = new UserListPage();

  beforeEach(() => {
    cy.intercept('GET', '/api/users', { 
      fixture: 'users.json' 
    }).as('getUsers');
    
    page.visit();
    cy.wait('@getUsers');
  });

  it('should display list of users', () => {
    page.getUserItems().should('have.length', 5);
  });

  it('should filter users by search query', () => {
    page.searchUser('John');
    page.getUserItems().should('have.length', 2);
  });

  it('should navigate to user details on click', () => {
    page.clickUser(0);
    cy.url().should('include', '/users/1');
  });

  it('should delete user when delete is clicked', () => {
    cy.intercept('DELETE', '/api/users/1', {
      statusCode: 204
    }).as('deleteUser');

    page.getDeleteButton(0).click();
    cy.get('[data-cy=confirm-dialog]').find('button').contains('Yes').click();
    
    cy.wait('@deleteUser');
    page.getUserItems().should('have.length', 4);
  });
});

// 5. Jest Snapshot Test
describe('ProductCardComponent Snapshot', () => {
  it('should match snapshot', () => {
    const fixture = TestBed.createComponent(ProductCardComponent);
    const component = fixture.componentInstance;
    
    component.product = {
      id: '1',
      name: 'Test Product',
      price: 99.99,
      description: 'Test description'
    };
    
    fixture.detectChanges();
    
    expect(fixture).toMatchSnapshot();
  });
});
```

### 9. What are Angular Signals and how do they improve change detection?

Signals, introduced in Angular 16, provide a reactive primitive for fine-grained reactivity. Unlike Zone.js polling, signals notify Angular precisely when state changes, enabling more efficient change detection. They reduce unnecessary checks, improve performance, and provide better developer ergonomics. Computed signals derive values automatically, and effects run side effects when dependencies change. This represents Angular's future direction toward zoneless applications.

**Example: Angular Signals**
```typescript
// 1. Basic Signal Usage
@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <div>
      <p>Count: {{ count() }}</p>
      <p>Double: {{ doubleCount() }}</p>
      <button (click)="increment()">Increment</button>
      <button (click)="decrement()">Decrement</button>
      <button (click)="reset()">Reset</button>
    </div>
  `
})
export class CounterComponent {
  // Writable signal
  count = signal(0);
  
  // Computed signal (automatically updates when count changes)
  doubleCount = computed(() => this.count() * 2);

  increment(): void {
    this.count.update(value => value + 1);
  }

  decrement(): void {
    this.count.update(value => value - 1);
  }

  reset(): void {
    this.count.set(0);
  }
}

// 2. Signals with Complex State
interface TodoState {
  items: Todo[];
  filter: 'all' | 'active' | 'completed';
}

@Component({
  selector: 'app-todo-list',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div>
      <input #input (keyup.enter)="addTodo(input.value); input.value = ''">
      
      <div>
        <button (click)="setFilter('all')">All ({{ allCount() }})</button>
        <button (click)="setFilter('active')">Active ({{ activeCount() }})</button>
        <button (click)="setFilter('completed')">Completed ({{ completedCount() }})</button>
      </div>

      <div *ngFor="let todo of filteredTodos()">
        <input 
          type="checkbox" 
          [checked]="todo.completed"
          (change)="toggleTodo(todo.id)">
        <span [class.completed]="todo.completed">{{ todo.text }}</span>
        <button (click)="removeTodo(todo.id)">Delete</button>
      </div>
    </div>
  `
})
export class TodoListComponent implements OnInit {
  // State signals
  private todos = signal<Todo[]>([]);
  private filter = signal<'all' | 'active' | 'completed'>('all');

  // Computed signals
  filteredTodos = computed(() => {
    const filter = this.filter();
    const todos = this.todos();

    switch (filter) {
      case 'active':
        return todos.filter(t => !t.completed);
      case 'completed':
        return todos.filter(t => t.completed);
      default:
        return todos;
    }
  });

  allCount = computed(() => this.todos().length);
  activeCount = computed(() => this.todos().filter(t => !t.completed).length);
  completedCount = computed(() => this.todos().filter(t => t.completed).length);

  constructor(private todoService: TodoService) {
    // Effect runs whenever dependencies change
    effect(() => {
      const todos = this.todos();
      console.log('Todos changed:', todos.length);
      // Could save to localStorage here
      localStorage.setItem('todos', JSON.stringify(todos));
    });
  }

  ngOnInit(): void {
    // Load initial data
    this.todoService.getTodos().subscribe(todos => {
      this.todos.set(todos);
    });
  }

  addTodo(text: string): void {
    if (!text.trim()) return;
    
    this.todos.update(todos => [
      ...todos,
      { id: Date.now().toString(), text, completed: false }
    ]);
  }

  toggleTodo(id: string): void {
    this.todos.update(todos =>
      todos.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }

  removeTodo(id: string): void {
    this.todos.update(todos => todos.filter(todo => todo.id !== id));
  }

  setFilter(filter: 'all' | 'active' | 'completed'): void {
    this.filter.set(filter);
  }
}

// 3. Signal-based Service
@Injectable({ providedIn: 'root' })
export class CartService {
  private items = signal<CartItem[]>([]);
  
  // Public readonly signals
  readonly cartItems = this.items.asReadonly();
  readonly totalItems = computed(() => 
    this.items().reduce((sum, item) => sum + item.quantity, 0)
  );
  readonly totalPrice = computed(() =>
    this.items().reduce((sum, item) => sum + (item.price * item.quantity), 0)
  );

  addItem(product: Product): void {
    this.items.update(items => {
      const existing = items.find(i => i.productId === product.id);
      
      if (existing) {
        return items.map(i =>
          i.productId === product.id
            ? { ...i, quantity: i.quantity + 1 }
            : i
        );
      }
      
      return [...items, {
        productId: product.id,
        name: product.name,
        price: product.price,
        quantity: 1
      }];
    });
  }

  removeItem(productId: string): void {
    this.items.update(items => items.filter(i => i.productId !== productId));
  }

  updateQuantity(productId: string, quantity: number): void {
    if (quantity <= 0) {
      this.removeItem(productId);
      return;
    }

    this.items.update(items =>
      items.map(i =>
        i.productId === productId ? { ...i, quantity } : i
      )
    );
  }

  clear(): void {
    this.items.set([]);
  }
}

// 4. Interop with RxJS
@Component({
  selector: 'app-user-profile',
  standalone: true
})
export class UserProfileComponent {
  private userService = inject(UserService);
  
  // Convert Observable to Signal
  user = toSignal(this.userService.getCurrentUser(), { 
    initialValue: null 
  });
  
  // Convert Signal to Observable
  userId = signal('123');
  userId$ = toObservable(this.userId);

  constructor() {
    // Subscribe to signal changes as observable
    this.userId$.pipe(
      switchMap(id => this.userService.getUser(id))
    ).subscribe();
  }
}
```

### 10. How do you handle authentication and authorization in Angular?

I implement JWT-based authentication with HTTP interceptors for token attachment, route guards (CanActivate, CanLoad) for authorization, refresh token mechanisms, and secure token storage considerations (memory vs httpOnly cookies). Role-based or permission-based access control is implemented through guard services. I handle token expiration gracefully, implement logout functionality, and ensure security best practices like HTTPS, CSRF protection, and sanitization.

**Example: Complete Auth Implementation**
```typescript
// 1. Auth Service
@Injectable({ providedIn: 'root' })
export class AuthService {
  private readonly TOKEN_KEY = 'auth_token';
  private readonly REFRESH_TOKEN_KEY = 'refresh_token';
  private currentUserSubject = new BehaviorSubject<User | null>(null);
  public currentUser$ = this.currentUserSubject.asObservable();

  constructor(private http: HttpClient, private router: Router) {
    this.loadUserFromToken();
  }

  login(credentials: LoginCredentials): Observable<AuthResponse> {
    return this.http.post<AuthResponse>('/api/auth/login', credentials).pipe(
      tap(response => {
        this.setTokens(response.accessToken, response.refreshToken);
        this.currentUserSubject.next(response.user);
      }),
      catchError(error => {
        console.error('Login failed:', error);
        return throwError(() => error);
      })
    );
  }

  logout(): void {
    localStorage.removeItem(this.TOKEN_KEY);
    localStorage.removeItem(this.REFRESH_TOKEN_KEY);
    this.currentUserSubject.next(null);
    this.router.navigate(['/login']);
  }

  refreshToken(): Observable<AuthResponse> {
    const refreshToken = localStorage.getItem(this.REFRESH_TOKEN_KEY);
    
    return this.http.post<AuthResponse>('/api/auth/refresh', { refreshToken }).pipe(
      tap(response => {
        this.setTokens(response.accessToken, response.refreshToken);
      })
    );
  }

  getToken(): string | null {
    return localStorage.getItem(this.TOKEN_KEY);
  }

  isAuthenticated(): boolean {
    const token = this.getToken();
    if (!token) return false;
    
    return !this.isTokenExpired(token);
  }

  hasRole(role: string): boolean {
    const user = this.currentUserSubject.value;
    return user?.roles?.includes(role) ?? false;
  }

  hasPermission(permission: string): boolean {
    const user = this.currentUserSubject.value;
    return user?.permissions?.includes(permission) ?? false;
  }

  private setTokens(accessToken: string, refreshToken: string): void {
    localStorage.setItem(this.TOKEN_KEY, accessToken);
    localStorage.setItem(this.REFRESH_TOKEN_KEY, refreshToken);
  }

  private loadUserFromToken(): void {
    const token = this.getToken();
    if (token && !this.isTokenExpired(token)) {
      const user = this.decodeToken(token);
      this.currentUserSubject.next(user);
    }
  }

  private isTokenExpired(token: string): boolean {
    try {
      const decoded: any = jwtDecode(token);
      return decoded.exp < Date.now() / 1000;
    } catch {
      return true;
    }
  }

  private decodeToken(token: string): User {
    const decoded: any = jwtDecode(token);
    return {
      id: decoded.sub,
      email: decoded.email,
      roles: decoded.roles || [],
      permissions: decoded.permissions || []
    };
  }
}

// 2. HTTP Interceptor for Token Attachment
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  private isRefreshing = false;
  private refreshTokenSubject = new BehaviorSubject<string | null>(null);

  constructor(private authService: AuthService) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Skip token for auth endpoints
    if (req.url.includes('/api/auth/')) {
      return next.handle(req);
    }

    const token = this.authService.getToken();
    
    if (token) {
      req = this.addToken(req, token);
    }

    return next.handle(req).pipe(
      catchError(error => {
        if (error instanceof HttpErrorResponse && error.status === 401) {
          return this.handle401Error(req, next);
        }
        return throwError(() => error);
      })
    );
  }

  private addToken(request: HttpRequest<any>, token: string): HttpRequest<any> {
    return request.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });
  }

  private handle401Error(
    request: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    if (!this.isRefreshing) {
      this.isRefreshing = true;
      this.refreshTokenSubject.next(null);

      return this.authService.refreshToken().pipe(
        switchMap((response) => {
          this.isRefreshing = false;
          this.refreshTokenSubject.next(response.accessToken);
          return next.handle(this.addToken(request, response.accessToken));
        }),
        catchError((err) => {
          this.isRefreshing = false;
          this.authService.logout();
          return throwError(() => err);
        })
      );
    }

    return this.refreshTokenSubject.pipe(
      filter(token => token !== null),
      take(1),
      switchMap(token => next.handle(this.addToken(request, token!)))
    );
  }
}

// 3. Route Guards
@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): boolean {
    if (this.authService.isAuthenticated()) {
      return true;
    }

    this.router.navigate(['/login'], { 
      queryParams: { returnUrl: state.url } 
    });
    return false;
  }
}

@Injectable({ providedIn: 'root' })
export class RoleGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(route: ActivatedRouteSnapshot): boolean {
    const requiredRoles = route.data['roles'] as string[];
    
    if (!requiredRoles || requiredRoles.length === 0) {
      return true;
    }

    const hasRole = requiredRoles.some(role => 
      this.authService.hasRole(role)
    );

    if (!hasRole) {
      this.router.navigate(['/forbidden']);
      return false;
    }

    return true;
  }
}

// 4. Route Configuration
const routes: Routes = [
  { path: 'login', component: LoginComponent },
  {
    path: 'dashboard',
    component: DashboardComponent,
    canActivate: [AuthGuard]
  },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule),
    canActivate: [AuthGuard, RoleGuard],
    data: { roles: ['ADMIN'] }
  },
  {
    path: 'user-management',
    component: UserManagementComponent,
    canActivate: [AuthGuard],
    canDeactivate: [UnsavedChangesGuard],
    data: { permissions: ['USER_WRITE'] }
  }
];

// 5. Permission Directive
@Directive({
  selector: '[appHasPermission]',
  standalone: true
})
export class HasPermissionDirective implements OnInit {
  @Input('appHasPermission') permission!: string;

  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef,
    private authService: AuthService
  ) {}

  ngOnInit(): void {
    if (this.authService.hasPermission(this.permission)) {
      this.viewContainer.createEmbeddedView(this.templateRef);
    } else {
      this.viewContainer.clear();
    }
  }
}

// Usage in template
// <button *appHasPermission="'USER_DELETE'" (click)="deleteUser()">
//   Delete User
// </button>

// 6. Login Component
@Component({
  selector: 'app-login',
  template: `
    <form [formGroup]="loginForm" (ngSubmit)="onSubmit()">
      <input formControlName="email" type="email" placeholder="Email">
      <input formControlName="password" type="password" placeholder="Password">
      <button type="submit" [disabled]="loginForm.invalid || loading">
        {{ loading ? 'Logging in...' : 'Login' }}
      </button>
      <div *ngIf="error" class="error">{{ error }}</div>
    </form>
  `
})
export class LoginComponent {
  loginForm = new FormGroup({
    email: new FormControl('', [Validators.required, Validators.email]),
    password: new FormControl('', Validators.required)
  });

  loading = false;
  error: string | null = null;

  constructor(
    private authService: AuthService,
    private router: Router,
    private route: ActivatedRoute
  ) {}

  onSubmit(): void {
    if (this.loginForm.invalid) return;

    this.loading = true;
    this.error = null;

    this.authService.login(this.loginForm.value as LoginCredentials).subscribe({
      next: () => {
        const returnUrl = this.route.snapshot.queryParams['returnUrl'] || '/dashboard';
        this.router.navigateByUrl(returnUrl);
      },
      error: (error) => {
        this.error = error.error?.message || 'Login failed';
        this.loading = false;
      },
      complete: () => {
        this.loading = false;
      }
    });
  }
}
```

### 11. Describe your experience with Angular Universal and SSR.

Angular Universal enables server-side rendering for better SEO, faster initial page loads, and improved social media sharing. I've implemented SSR with considerations for browser-only APIs (using platform checks), state transfer to avoid duplicate requests, preboot for event buffering, and deployment on Node.js servers or serverless platforms. Challenges include memory management, third-party library compatibility, and balancing SSR benefits against complexity.

### 12. How do you approach internationalization (i18n) in large applications?

I use Angular's built-in i18n support or ngx-translate for runtime translation. Strategies include extracting strings to translation files, implementing language switching, handling RTL layouts, localizing dates/numbers/currencies using Angular pipes, lazy loading translation files, and managing translation workflows with tools like Lokalise or Crowdin. For large applications, I implement translation keys systematically and ensure consistency across the application.

### 13. What's your experience with Angular animations?

I've implemented complex animations using Angular's animation DSL with triggers, states, transitions, and keyframes. Use cases include route transitions, list animations with stagger, complex sequences, and gesture-based animations. I balance performance considerations using CSS transforms, GPU acceleration, and avoiding layout thrashing. For advanced interactions, I've integrated GSAP or Web Animations API when Angular's animations have limitations.

### 14. How do you handle error handling and logging in production?

I implement global error handlers with ErrorHandler, HTTP interceptors for API errors, retry strategies with RxJS, user-friendly error messages, and error boundary patterns. For logging, I integrate services like Sentry, LogRocket, or custom solutions with different log levels (debug, info, warn, error). Source maps are uploaded for production debugging. I also implement telemetry for monitoring application health and user experience metrics.

### 15. Explain custom decorators and their use cases.

I've created custom decorators for cross-cutting concerns like @Memoize for caching method results, @Debounce for rate limiting, @Log for automatic logging, and @Authorize for method-level security. Decorators use TypeScript's experimental feature and can modify classes, methods, properties, or parameters. They're powerful for AOP (Aspect-Oriented Programming) patterns, reducing boilerplate, and enforcing conventions across large codebases.

**Example: Custom Decorators**
```typescript
// 1. @Memoize Decorator - Cache method results
export function Memoize(options?: { ttl?: number }) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;
    const cache = new Map<string, { value: any; timestamp: number }>();
    const ttl = options?.ttl || Infinity;

    descriptor.value = function (...args: any[]) {
      const key = JSON.stringify(args);
      const cached = cache.get(key);

      if (cached && Date.now() - cached.timestamp < ttl) {
        console.log(`Cache hit for ${propertyKey}`);
        return cached.value;
      }

      const result = originalMethod.apply(this, args);
      cache.set(key, { value: result, timestamp: Date.now() });
      
      return result;
    };

    return descriptor;
  };
}

// Usage
@Injectable()
export class DataService {
  @Memoize({ ttl: 5000 }) // Cache for 5 seconds
  expensiveCalculation(input: number): number {
    console.log('Performing expensive calculation...');
    return input * Math.random() * 1000;
  }
}

// 2. @Debounce Decorator - Rate limit method calls
export function Debounce(delay: number = 300) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;
    let timeoutId: any;

    descriptor.value = function (...args: any[]) {
      clearTimeout(timeoutId);
      
      timeoutId = setTimeout(() => {
        originalMethod.apply(this, args);
      }, delay);
    };

    return descriptor;
  };
}

// Usage
@Component({
  selector: 'app-search'
})
export class SearchComponent {
  @Debounce(500)
  onSearchInput(query: string): void {
    console.log('Searching for:', query);
    // API call here
  }
}

// 3. @Throttle Decorator - Limit execution frequency
export function Throttle(delay: number = 300) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;
    let lastExecuted = 0;

    descriptor.value = function (...args: any[]) {
      const now = Date.now();
      
      if (now - lastExecuted >= delay) {
        lastExecuted = now;
        return originalMethod.apply(this, args);
      }
    };

    return descriptor;
  };
}

// Usage
@Component({
  selector: 'app-scroll-handler'
})
export class ScrollHandlerComponent {
  @Throttle(200)
  @HostListener('window:scroll')
  onWindowScroll(): void {
    console.log('Scroll position:', window.scrollY);
  }
}

// 4. @Log Decorator - Automatic logging
export function Log(level: 'info' | 'warn' | 'error' = 'info') {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      const className = target.constructor.name;
      console[level](`[${className}.${propertyKey}] Called with:`, args);
      
      try {
        const result = originalMethod.apply(this, args);
        console[level](`[${className}.${propertyKey}] Returned:`, result);
        return result;
      } catch (error) {
        console.error(`[${className}.${propertyKey}] Error:`, error);
        throw error;
      }
    };

    return descriptor;
  };
}

// Usage
@Injectable()
export class UserService {
  @Log('info')
  getUser(id: string): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`);
  }

  @Log('warn')
  deleteUser(id: string): Observable<void> {
    return this.http.delete<void>(`/api/users/${id}`);
  }
}

// 5. @Retry Decorator - Automatic retry on failure
export function Retry(maxAttempts: number = 3, delay: number = 1000) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = async function (...args: any[]) {
      let lastError: any;

      for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
          return await originalMethod.apply(this, args);
        } catch (error) {
          lastError = error;
          console.warn(
            `Attempt ${attempt}/${maxAttempts} failed for ${propertyKey}:`,
            error
          );

          if (attempt < maxAttempts) {
            await new Promise(resolve => setTimeout(resolve, delay * attempt));
          }
        }
      }

      throw lastError;
    };

    return descriptor;
  };
}

// Usage
@Injectable()
export class ApiService {
  @Retry(3, 1000)
  async fetchCriticalData(): Promise<any> {
    const response = await fetch('/api/critical-data');
    if (!response.ok) throw new Error('Fetch failed');
    return response.json();
  }
}

// 6. @Measure Decorator - Performance measurement
export function Measure() {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      const start = performance.now();
      const result = originalMethod.apply(this, args);
      const end = performance.now();
      
      console.log(
        `${target.constructor.name}.${propertyKey} executed in ${(end - start).toFixed(2)}ms`
      );

      return result;
    };

    return descriptor;
  };
}

// Usage
@Component({
  selector: 'app-dashboard'
})
export class DashboardComponent {
  @Measure()
  loadDashboardData(): void {
    // Complex calculation or data loading
    const data = this.processLargeDataset();
  }

  private processLargeDataset(): any[] {
    return Array.from({ length: 10000 }, (_, i) => ({ id: i }));
  }
}

// 7. @Required Decorator - Validate required inputs
export function Required() {
  return function (target: any, propertyKey: string) {
    let value: any;

    const getter = () => {
      return value;
    };

    const setter = (newValue: any) => {
      if (newValue === undefined || newValue === null) {
        throw new Error(`${propertyKey} is required but was ${newValue}`);
      }
      value = newValue;
    };

    Object.defineProperty(target, propertyKey, {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true
    });
  };
}

// Usage
@Component({
  selector: 'app-user-card'
})
export class UserCardComponent {
  @Required()
  @Input() user!: User;

  @Required()
  @Input() displayMode!: 'compact' | 'full';
}

// 8. @ConfirmAction Decorator - Require user confirmation
export function ConfirmAction(message: string = 'Are you sure?') {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      if (confirm(message)) {
        return originalMethod.apply(this, args);
      }
      console.log('Action cancelled by user');
    };

    return descriptor;
  };
}

// Usage
@Component({
  selector: 'app-user-list'
})
export class UserListComponent {
  @ConfirmAction('Are you sure you want to delete this user?')
  deleteUser(userId: string): void {
    this.userService.delete(userId).subscribe();
  }

  @ConfirmAction('This will clear all data. Continue?')
  clearAllData(): void {
    this.dataService.clearAll();
  }
}

// 9. Class Decorator - Add metadata
export function Component_Metadata(metadata: { version: string; author: string }) {
  return function <T extends { new(...args: any[]): {} }>(constructor: T) {
    return class extends constructor {
      _metadata = metadata;
      
      getMetadata() {
        return this._metadata;
      }
    };
  };
}

// Usage
@Component_Metadata({ version: '1.0.0', author: 'John Doe' })
@Component({
  selector: 'app-custom'
})
export class CustomComponent {
  // Component now has metadata attached
}
```

### 16. How do you manage dependencies and upgrades in Angular projects?

I follow semantic versioning, use npm/yarn/pnpm with lock files, regularly audit dependencies with npm audit, implement automated dependency updates with Renovate or Dependabot, test thoroughly before upgrades, and maintain compatibility matrices. For major Angular upgrades, I use ng update, review breaking changes, update third-party libraries, and implement incremental upgrades when possible. Maintaining upgrade readiness requires discipline and planning.

### 17. What's your approach to accessibility (a11y) in Angular applications?

Accessibility is fundamental, not optional. I ensure semantic HTML, ARIA attributes where needed, keyboard navigation support, focus management, sufficient color contrast, screen reader testing, and adherence to WCAG 2.1 AA standards. I use Angular CDK's a11y utilities, implement skip links, ensure form accessibility with proper labels and error announcements, and integrate automated testing with tools like axe-core and pa11y in CI/CD pipelines.

### 18. Describe your experience with monorepo management for Angular.

I've managed monorepos using Nx or Angular CLI workspaces, which provide code sharing, consistent tooling, and dependency management across multiple applications and libraries. Benefits include atomic commits across projects, easier refactoring, and shared CI/CD configurations. I implement proper library boundaries, use path mapping, manage interdependencies carefully, and leverage computation caching for faster builds. Monorepos require discipline but scale well for large organizations.

### 19. How do you implement real-time features in Angular applications?

For real-time functionality, I've used WebSockets with libraries like Socket.io or native WebSocket API, Server-Sent Events for one-way updates, and SignalR for .NET backends. I wrap WebSocket connections in RxJS observables for consistent reactive patterns, implement reconnection logic, handle connection state, and manage subscriptions properly. For collaborative features, I've implemented operational transformation or CRDT patterns for conflict resolution.

**Example: Real-time Implementation**
```typescript
// 1. WebSocket Service
@Injectable({ providedIn: 'root' })
export class WebSocketService {
  private socket$!: WebSocketSubject<any>;
  private reconnectInterval = 5000;
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 10;

  private messagesSubject$ = new Subject<any>();
  public messages$ = this.messagesSubject$.asObservable();

  private connectionStatusSubject$ = new BehaviorSubject<boolean>(false);
  public connectionStatus$ = this.connectionStatusSubject$.asObservable();

  connect(url: string): void {
    if (!this.socket$ || this.socket$.closed) {
      this.socket$ = this.getNewWebSocket(url);
      
      this.socket$.pipe(
        retryWhen(errors =>
          errors.pipe(
            tap(() => {
              this.reconnectAttempts++;
              console.log(`Reconnect attempt ${this.reconnectAttempts}`);
            }),
            delayWhen(() => {
              if (this.reconnectAttempts >= this.maxReconnectAttempts) {
                throw new Error('Max reconnection attempts reached');
              }
              return timer(this.reconnectInterval);
            })
          )
        ),
        catchError(error => {
          console.error('WebSocket connection error:', error);
          this.connectionStatusSubject$.next(false);
          return EMPTY;
        })
      ).subscribe({
        next: (message) => {
          this.messagesSubject$.next(message);
          this.reconnectAttempts = 0;
        },
        error: (error) => {
          console.error('WebSocket error:', error);
          this.connectionStatusSubject$.next(false);
        },
        complete: () => {
          console.log('WebSocket connection closed');
          this.connectionStatusSubject$.next(false);
        }
      });

      this.connectionStatusSubject$.next(true);
    }
  }

  private getNewWebSocket(url: string): WebSocketSubject<any> {
    return webSocket({
      url,
      openObserver: {
        next: () => {
          console.log('WebSocket connected');
          this.connectionStatusSubject$.next(true);
        }
      },
      closeObserver: {
        next: () => {
          console.log('WebSocket disconnected');
          this.connectionStatusSubject$.next(false);
        }
      }
    });
  }

  send(message: any): void {
    if (this.socket$) {
      this.socket$.next(message);
    }
  }

  disconnect(): void {
    if (this.socket$) {
      this.socket$.complete();
      this.connectionStatusSubject$.next(false);
    }
  }
}

// 2. Chat Service using WebSocket
@Injectable({ providedIn: 'root' })
export class ChatService {
  private ws = inject(WebSocketService);
  
  private chatMessagesSubject$ = new Subject<ChatMessage>();
  public chatMessages$ = this.chatMessagesSubject$.asObservable();

  private typingUsersSubject$ = new Subject<string[]>();
  public typingUsers$ = this.typingUsersSubject$.asObservable();

  constructor() {
    this.ws.connect('ws://localhost:3000/chat');
    
    this.ws.messages$.subscribe((message) => {
      this.handleMessage(message);
    });
  }

  sendMessage(text: string, roomId: string): void {
    const message = {
      type: 'chat_message',
      payload: {
        text,
        roomId,
        timestamp: new Date().toISOString()
      }
    };
    this.ws.send(message);
  }

  joinRoom(roomId: string): void {
    this.ws.send({
      type: 'join_room',
      payload: { roomId }
    });
  }

  leaveRoom(roomId: string): void {
    this.ws.send({
      type: 'leave_room',
      payload: { roomId }
    });
  }

  sendTypingIndicator(roomId: string, isTyping: boolean): void {
    this.ws.send({
      type: 'typing_indicator',
      payload: { roomId, isTyping }
    });
  }

  private handleMessage(message: any): void {
    switch (message.type) {
      case 'chat_message':
        this.chatMessagesSubject$.next(message.payload);
        break;
      case 'typing_update':
        this.typingUsersSubject$.next(message.payload.users);
        break;
      default:
        console.log('Unknown message type:', message.type);
    }
  }
}

// 3. Real-time Chat Component
@Component({
  selector: 'app-chat',
  template: `
    <div class="chat-container">
      <div class="connection-status" [class.connected]="isConnected$ | async">
        {{ (isConnected$ | async) ? 'Connected' : 'Disconnected' }}
      </div>

      <div class="messages" #messageContainer>
        <div *ngFor="let msg of messages$ | async" class="message">
          <strong>{{ msg.username }}:</strong> {{ msg.text }}
          <span class="timestamp">{{ msg.timestamp | date:'short' }}</span>
        </div>
      </div>

      <div *ngIf="(typingUsers$ | async)?.length" class="typing-indicator">
        {{ (typingUsers$ | async)?.join(', ') }} 
        {{ (typingUsers$ | async)?.length === 1 ? 'is' : 'are' }} typing...
      </div>

      <div class="input-container">
        <input
          #messageInput
          [(ngModel)]="messageText"
          (keyup.enter)="sendMessage()"
          (input)="onTyping()"
          placeholder="Type a message...">
        <button (click)="sendMessage()">Send</button>
      </div>
    </div>
  `
})
export class ChatComponent implements OnInit, OnDestroy {
  @ViewChild('messageContainer') messageContainer!: ElementRef;
  
  messageText = '';
  roomId = 'general';
  
  messages$ = this.chatService.chatMessages$;
  typingUsers$ = this.chatService.typingUsers$;
  isConnected$ = this.wsService.connectionStatus$;
  
  private typingTimeout: any;
  private destroy$ = new Subject<void>();

  constructor(
    private chatService: ChatService,
    private wsService: WebSocketService
  ) {}

  ngOnInit(): void {
    this.chatService.joinRoom(this.roomId);
    
    // Auto-scroll to bottom on new messages
    this.messages$.pipe(
      takeUntil(this.destroy$)
    ).subscribe(() => {
      setTimeout(() => this.scrollToBottom(), 100);
    });
  }

  ngOnDestroy(): void {
    this.chatService.leaveRoom(this.roomId);
    this.destroy$.next();
    this.destroy$.complete();
  }

  sendMessage(): void {
    if (this.messageText.trim()) {
      this.chatService.sendMessage(this.messageText, this.roomId);
      this.messageText = '';
      this.chatService.sendTypingIndicator(this.roomId, false);
    }
  }

  onTyping(): void {
    this.chatService.sendTypingIndicator(this.roomId, true);
    
    clearTimeout(this.typingTimeout);
    this.typingTimeout = setTimeout(() => {
      this.chatService.sendTypingIndicator(this.roomId, false);
    }, 2000);
  }

  private scrollToBottom(): void {
    const container = this.messageContainer.nativeElement;
    container.scrollTop = container.scrollHeight;
  }
}

// 4. Server-Sent Events (SSE) for Notifications
@Injectable({ providedIn: 'root' })
export class NotificationService {
  private notificationsSubject$ = new Subject<Notification>();
  public notifications$ = this.notificationsSubject$.asObservable();

  private eventSource: EventSource | null = null;

  connect(userId: string): void {
    if (this.eventSource) {
      this.eventSource.close();
    }

    this.eventSource = new EventSource(`/api/notifications/stream/${userId}`);

    this.eventSource.onmessage = (event) => {
      const notification = JSON.parse(event.data);
      this.notificationsSubject$.next(notification);
    };

    this.eventSource.onerror = (error) => {
      console.error('SSE error:', error);
      this.eventSource?.close();
      
      // Reconnect after 5 seconds
      setTimeout(() => this.connect(userId), 5000);
    };
  }

  disconnect(): void {
    if (this.eventSource) {
      this.eventSource.close();
      this.eventSource = null;
    }
  }
}

// 5. Real-time Collaborative Editor (CRDT-like)
@Injectable({ providedIn: 'root' })
export class CollaborativeEditorService {
  private ws = inject(WebSocketService);
  private documentId = '';
  
  private contentSubject$ = new BehaviorSubject<string>('');
  public content$ = this.contentSubject$.asObservable();

  private cursorsSubject$ = new BehaviorSubject<Map<string, CursorPosition>>(
    new Map()
  );
  public cursors$ = this.cursorsSubject$.asObservable();

  constructor() {
    this.ws.messages$.subscribe((message) => {
      this.handleMessage(message);
    });
  }

  openDocument(documentId: string): void {
    this.documentId = documentId;
    this.ws.send({
      type: 'open_document',
      payload: { documentId }
    });
  }

  applyOperation(operation: TextOperation): void {
    this.ws.send({
      type: 'text_operation',
      payload: {
        documentId: this.documentId,
        operation
      }
    });
  }

  updateCursor(position: CursorPosition): void {
    this.ws.send({
      type: 'cursor_update',
      payload: {
        documentId: this.documentId,
        position
      }
    });
  }

  private handleMessage(message: any): void {
    switch (message.type) {
      case 'document_content':
        this.contentSubject$.next(message.payload.content);
        break;
        
      case 'text_operation':
        const currentContent = this.contentSubject$.value;
        const newContent = this.applyOperationToText(
          currentContent,
          message.payload.operation
        );
        this.contentSubject$.next(newContent);
        break;
        
      case 'cursor_update':
        const cursors = new Map(this.cursorsSubject$.value);
        cursors.set(message.payload.userId, message.payload.position);
        this.cursorsSubject$.next(cursors);
        break;
    }
  }

  private applyOperationToText(text: string, op: TextOperation): string {
    // Simplified OT implementation
    if (op.type === 'insert') {
      return text.slice(0, op.position) + op.text + text.slice(op.position);
    } else if (op.type === 'delete') {
      return text.slice(0, op.position) + text.slice(op.position + op.length);
    }
    return text;
  }
}

interface TextOperation {
  type: 'insert' | 'delete';
  position: number;
  text?: string;
  length?: number;
}

interface CursorPosition {
  line: number;
  column: number;
}
```

### 20. What's your strategy for code review and maintaining code quality?

I enforce coding standards with ESLint and Prettier, use pre-commit hooks with Husky, implement branch protection rules, conduct thorough PR reviews focusing on architecture, performance, security, and maintainability. I promote pair programming for complex features, maintain comprehensive documentation, use static analysis tools like SonarQube, and foster a culture of constructive feedback. Code quality is a team responsibility, not individual.

### 21. How do you handle large file uploads in Angular?

I implement chunked uploads for large files, show progress indicators using HTTP progress events, handle retry logic for failed chunks, validate file types and sizes on client and server, use FormData API for multipart uploads, implement pause/resume functionality, and consider using cloud storage services with pre-signed URLs for direct uploads. Background upload with service workers is useful for improving UX.

### 22. Explain your approach to CSS architecture in Angular applications.

I use component-scoped styles with ViewEncapsulation, implement design systems with CSS variables for theming, use SCSS for advanced features like mixins and functions, follow BEM or similar methodology for clarity, implement responsive design with mobile-first approach, leverage CSS Grid and Flexbox, and consider CSS-in-JS solutions when appropriate. I optimize CSS delivery through code splitting and critical CSS extraction for performance.

### 23. How do you mentor junior developers in your team?

Mentoring involves code reviews with educational feedback, pair programming sessions, creating documentation and best practice guides, conducting knowledge sharing sessions, assigning gradually complex tasks, encouraging questions and experimentation in safe environments, and being available for guidance. I focus on developing problem-solving skills rather than just providing solutions, and I ensure psychological safety for learning and growth.

### 24. What's your experience with PWA development in Angular?

I've built Progressive Web Apps using Angular Service Worker, implementing offline functionality with caching strategies (cache-first, network-first), background sync, push notifications, and app manifest for installability. Challenges include cache invalidation, storage quota management, testing offline scenarios, and handling different network conditions. PWAs bridge the gap between web and native apps, providing enhanced user experiences.

### 25. How do you stay current with Angular ecosystem and web technologies?

I follow Angular blog and release notes, participate in Angular community on Twitter/Discord, attend conferences (ng-conf, AngularConnect), contribute to open source, experiment with new features in side projects, follow thought leaders, read RFCs and proposals, take online courses, and share knowledge through blogs or internal tech talks. Continuous learning is essential in our rapidly evolving field, and I dedicate regular time for professional development.

### 26. Describe a challenging architectural decision you made and its impact.

In a previous project, I decided to migrate from a monolithic architecture to micro-frontends to enable independent team deployments. This involved evaluating Module Federation vs single-spa, designing shared component libraries, establishing communication contracts, and planning incremental migration. The impact was significant: reduced deployment dependencies, faster feature delivery, better team autonomy, but also increased DevOps complexity and initial learning curve. The decision required stakeholder buy-in, comprehensive planning, and long-term commitment.

### 27. How do you balance technical debt with feature development?

Technical debt is inevitable but manageable. I advocate for allocating 15-20% of sprint capacity to debt reduction, using metrics to identify high-impact areas, prioritizing debt that blocks future features or poses security risks, and making incremental improvements rather than big rewrites. I document debt in backlogs, communicate impacts to stakeholders, and promote a culture where quality is everyone's responsibility. Sometimes saying no to features to address critical debt is necessary for long-term sustainability.
