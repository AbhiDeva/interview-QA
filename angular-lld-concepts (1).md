# Low Level Design Concepts in Angular

## Table of Contents
1. [SOLID Principles in Angular](#solid-principles)
2. [Design Patterns](#design-patterns)
3. [State Management Patterns](#state-management)
4. [Component Architecture](#component-architecture)
5. [Service Layer Design](#service-layer-design)
6. [Error Handling Patterns](#error-handling)
7. [Performance Optimization Patterns](#performance-optimization)
8. [Testing Patterns](#testing-patterns)

---

## 1. SOLID Principles in Angular {#solid-principles}

### Single Responsibility Principle (SRP)

**Concept**: Each class/component should have one and only one reason to change.

```typescript
// ❌ BAD: Component doing too much
@Component({
  selector: 'app-user-profile',
  template: `...`
})
export class UserProfileComponent {
  user: User;
  
  // Responsibility 1: Data fetching
  loadUser() {
    this.http.get('/api/users/1').subscribe(user => this.user = user);
  }
  
  // Responsibility 2: Validation
  validateUser() {
    return this.user.email.includes('@') && this.user.age > 18;
  }
  
  // Responsibility 3: Business logic
  calculateDiscount() {
    return this.user.isPremium ? 0.2 : 0.1;
  }
}

// ✅ GOOD: Separated responsibilities
// Component - Only UI logic
@Component({
  selector: 'app-user-profile',
  template: `
    <div *ngIf="user$ | async as user">
      <h2>{{ user.name }}</h2>
      <p>Discount: {{ discountService.calculate(user) }}</p>
    </div>
  `
})
export class UserProfileComponent {
  user$ = this.userService.getUser(1);
  
  constructor(
    private userService: UserService,
    public discountService: DiscountService
  ) {}
}

// Service - Data fetching
@Injectable({ providedIn: 'root' })
export class UserService {
  constructor(private http: HttpClient) {}
  
  getUser(id: number): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`);
  }
}

// Service - Business logic
@Injectable({ providedIn: 'root' })
export class DiscountService {
  calculate(user: User): number {
    return user.isPremium ? 0.2 : 0.1;
  }
}

// Validator - Validation logic
export class UserValidator {
  static validate(user: User): boolean {
    return user.email.includes('@') && user.age > 18;
  }
}
```

### Open/Closed Principle (OCP)

**Concept**: Software entities should be open for extension but closed for modification.

```typescript
// Base notification service
export abstract class NotificationService {
  abstract send(message: string, recipient: string): void;
}

// Email implementation
@Injectable({ providedIn: 'root' })
export class EmailNotificationService extends NotificationService {
  send(message: string, recipient: string): void {
    console.log(`Sending email to ${recipient}: ${message}`);
    // Email sending logic
  }
}

// SMS implementation
@Injectable({ providedIn: 'root' })
export class SmsNotificationService extends NotificationService {
  send(message: string, recipient: string): void {
    console.log(`Sending SMS to ${recipient}: ${message}`);
    // SMS sending logic
  }
}

// Push notification implementation
@Injectable({ providedIn: 'root' })
export class PushNotificationService extends NotificationService {
  send(message: string, recipient: string): void {
    console.log(`Sending push notification to ${recipient}: ${message}`);
    // Push notification logic
  }
}

// Usage with factory pattern
@Injectable({ providedIn: 'root' })
export class NotificationFactory {
  create(type: 'email' | 'sms' | 'push'): NotificationService {
    switch (type) {
      case 'email':
        return new EmailNotificationService();
      case 'sms':
        return new SmsNotificationService();
      case 'push':
        return new PushNotificationService();
    }
  }
}
```

### Liskov Substitution Principle (LSP)

**Concept**: Derived classes must be substitutable for their base classes.

```typescript
// Base class
export abstract class DataSource<T> {
  abstract fetchData(): Observable<T[]>;
  abstract saveData(item: T): Observable<T>;
}

// Implementation 1: API data source
@Injectable()
export class ApiDataSource<T> extends DataSource<T> {
  constructor(private http: HttpClient, private endpoint: string) {
    super();
  }
  
  fetchData(): Observable<T[]> {
    return this.http.get<T[]>(this.endpoint);
  }
  
  saveData(item: T): Observable<T> {
    return this.http.post<T>(this.endpoint, item);
  }
}

// Implementation 2: Local storage data source
@Injectable()
export class LocalStorageDataSource<T> extends DataSource<T> {
  constructor(private storageKey: string) {
    super();
  }
  
  fetchData(): Observable<T[]> {
    const data = JSON.parse(localStorage.getItem(this.storageKey) || '[]');
    return of(data);
  }
  
  saveData(item: T): Observable<T> {
    const current = JSON.parse(localStorage.getItem(this.storageKey) || '[]');
    current.push(item);
    localStorage.setItem(this.storageKey, JSON.stringify(current));
    return of(item);
  }
}

// Component can use any implementation
@Component({
  selector: 'app-data-list',
  template: `<div *ngFor="let item of data$ | async">{{ item }}</div>`
})
export class DataListComponent {
  data$: Observable<any[]>;
  
  constructor(@Inject('DATA_SOURCE') private dataSource: DataSource<any>) {
    this.data$ = this.dataSource.fetchData();
  }
}
```

### Interface Segregation Principle (ISP)

**Concept**: Clients should not be forced to depend on interfaces they don't use.

```typescript
// ❌ BAD: Fat interface
interface CrudOperations<T> {
  create(item: T): Observable<T>;
  read(id: number): Observable<T>;
  update(id: number, item: T): Observable<T>;
  delete(id: number): Observable<void>;
  search(query: string): Observable<T[]>;
  export(): Observable<Blob>;
  import(file: File): Observable<void>;
}

// ✅ GOOD: Segregated interfaces
interface Readable<T> {
  read(id: number): Observable<T>;
}

interface Writable<T> {
  create(item: T): Observable<T>;
  update(id: number, item: T): Observable<T>;
}

interface Deletable {
  delete(id: number): Observable<void>;
}

interface Searchable<T> {
  search(query: string): Observable<T[]>;
}

interface Exportable {
  export(): Observable<Blob>;
}

// Implement only what you need
@Injectable({ providedIn: 'root' })
export class ReadOnlyUserService implements Readable<User>, Searchable<User> {
  constructor(private http: HttpClient) {}
  
  read(id: number): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`);
  }
  
  search(query: string): Observable<User[]> {
    return this.http.get<User[]>(`/api/users/search?q=${query}`);
  }
}

@Injectable({ providedIn: 'root' })
export class FullUserService implements 
  Readable<User>, 
  Writable<User>, 
  Deletable, 
  Searchable<User> {
  
  constructor(private http: HttpClient) {}
  
  read(id: number): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`);
  }
  
  create(user: User): Observable<User> {
    return this.http.post<User>('/api/users', user);
  }
  
  update(id: number, user: User): Observable<User> {
    return this.http.put<User>(`/api/users/${id}`, user);
  }
  
  delete(id: number): Observable<void> {
    return this.http.delete<void>(`/api/users/${id}`);
  }
  
  search(query: string): Observable<User[]> {
    return this.http.get<User[]>(`/api/users/search?q=${query}`);
  }
}
```

### Dependency Inversion Principle (DIP)

**Concept**: High-level modules should not depend on low-level modules. Both should depend on abstractions.

```typescript
// ❌ BAD: Direct dependency on concrete class
@Injectable({ providedIn: 'root' })
export class OrderService {
  constructor(private emailService: EmailNotificationService) {}
  
  placeOrder(order: Order) {
    // Process order
    this.emailService.send('Order placed', order.customerEmail);
  }
}

// ✅ GOOD: Depend on abstraction
export abstract class INotificationService {
  abstract send(message: string, recipient: string): void;
}

@Injectable({ providedIn: 'root' })
export class EmailNotificationService implements INotificationService {
  send(message: string, recipient: string): void {
    console.log(`Email sent to ${recipient}`);
  }
}

@Injectable({ providedIn: 'root' })
export class OrderService {
  constructor(@Inject('INotificationService') private notifier: INotificationService) {}
  
  placeOrder(order: Order) {
    // Process order
    this.notifier.send('Order placed', order.customerEmail);
  }
}

// Provider configuration
{
  provide: 'INotificationService',
  useClass: EmailNotificationService
}
```

---

## 2. Design Patterns {#design-patterns}

### Singleton Pattern

```typescript
// Using Angular's providedIn: 'root'
@Injectable({ providedIn: 'root' })
export class ConfigService {
  private config: AppConfig;
  
  constructor() {
    this.loadConfig();
  }
  
  private loadConfig() {
    this.config = {
      apiUrl: 'https://api.example.com',
      timeout: 5000
    };
  }
  
  getConfig(): AppConfig {
    return this.config;
  }
}

// Manual singleton implementation
export class Logger {
  private static instance: Logger;
  private logs: string[] = [];
  
  private constructor() {}
  
  static getInstance(): Logger {
    if (!Logger.instance) {
      Logger.instance = new Logger();
    }
    return Logger.instance;
  }
  
  log(message: string): void {
    this.logs.push(`${new Date().toISOString()}: ${message}`);
  }
  
  getLogs(): string[] {
    return [...this.logs];
  }
}
```

### Factory Pattern

```typescript
// Abstract product
export interface Dialog {
  render(): void;
  close(): void;
}

// Concrete products
export class ModalDialog implements Dialog {
  render(): void {
    console.log('Rendering modal dialog');
  }
  
  close(): void {
    console.log('Closing modal dialog');
  }
}

export class SlideDialog implements Dialog {
  render(): void {
    console.log('Rendering slide dialog');
  }
  
  close(): void {
    console.log('Closing slide dialog');
  }
}

export class FullscreenDialog implements Dialog {
  render(): void {
    console.log('Rendering fullscreen dialog');
  }
  
  close(): void {
    console.log('Closing fullscreen dialog');
  }
}

// Factory
@Injectable({ providedIn: 'root' })
export class DialogFactory {
  create(type: 'modal' | 'slide' | 'fullscreen'): Dialog {
    switch (type) {
      case 'modal':
        return new ModalDialog();
      case 'slide':
        return new SlideDialog();
      case 'fullscreen':
        return new FullscreenDialog();
      default:
        throw new Error('Invalid dialog type');
    }
  }
}

// Usage
@Component({
  selector: 'app-dialog-example',
  template: `<button (click)="openDialog('modal')">Open Modal</button>`
})
export class DialogExampleComponent {
  constructor(private dialogFactory: DialogFactory) {}
  
  openDialog(type: 'modal' | 'slide' | 'fullscreen') {
    const dialog = this.dialogFactory.create(type);
    dialog.render();
  }
}
```

### Observer Pattern (RxJS)

```typescript
// Subject for event broadcasting
@Injectable({ providedIn: 'root' })
export class EventBusService {
  private eventSubject = new Subject<AppEvent>();
  public events$ = this.eventSubject.asObservable();
  
  emit(event: AppEvent): void {
    this.eventSubject.next(event);
  }
}

// Event types
export interface AppEvent {
  type: string;
  payload?: any;
}

// Publisher component
@Component({
  selector: 'app-publisher',
  template: `<button (click)="publish()">Publish Event</button>`
})
export class PublisherComponent {
  constructor(private eventBus: EventBusService) {}
  
  publish() {
    this.eventBus.emit({
      type: 'USER_LOGGED_IN',
      payload: { userId: 123, username: 'john' }
    });
  }
}

// Subscriber component
@Component({
  selector: 'app-subscriber',
  template: `<div>{{ message }}</div>`
})
export class SubscriberComponent implements OnInit, OnDestroy {
  message = '';
  private subscription: Subscription;
  
  constructor(private eventBus: EventBusService) {}
  
  ngOnInit() {
    this.subscription = this.eventBus.events$
      .pipe(filter(event => event.type === 'USER_LOGGED_IN'))
      .subscribe(event => {
        this.message = `User ${event.payload.username} logged in`;
      });
  }
  
  ngOnDestroy() {
    this.subscription.unsubscribe();
  }
}
```

### Strategy Pattern

```typescript
// Strategy interface
export interface SortStrategy<T> {
  sort(items: T[]): T[];
}

// Concrete strategies
export class AscendingSortStrategy<T> implements SortStrategy<T> {
  sort(items: T[]): T[] {
    return [...items].sort((a, b) => a > b ? 1 : -1);
  }
}

export class DescendingSortStrategy<T> implements SortStrategy<T> {
  sort(items: T[]): T[] {
    return [...items].sort((a, b) => a < b ? 1 : -1);
  }
}

export class CustomSortStrategy<T> implements SortStrategy<T> {
  constructor(private compareFn: (a: T, b: T) => number) {}
  
  sort(items: T[]): T[] {
    return [...items].sort(this.compareFn);
  }
}

// Context
@Component({
  selector: 'app-sortable-list',
  template: `
    <button (click)="setSortStrategy('asc')">Sort Ascending</button>
    <button (click)="setSortStrategy('desc')">Sort Descending</button>
    <div *ngFor="let item of sortedItems">{{ item }}</div>
  `
})
export class SortableListComponent {
  items = [5, 2, 8, 1, 9];
  sortedItems: number[] = [];
  private sortStrategy: SortStrategy<number>;
  
  constructor() {
    this.sortStrategy = new AscendingSortStrategy();
    this.sortedItems = this.sortStrategy.sort(this.items);
  }
  
  setSortStrategy(type: 'asc' | 'desc') {
    if (type === 'asc') {
      this.sortStrategy = new AscendingSortStrategy();
    } else {
      this.sortStrategy = new DescendingSortStrategy();
    }
    this.sortedItems = this.sortStrategy.sort(this.items);
  }
}
```

### Decorator Pattern

```typescript
// Base component interface
export interface DataService {
  getData(): Observable<any[]>;
}

// Concrete component
@Injectable()
export class BasicDataService implements DataService {
  constructor(private http: HttpClient) {}
  
  getData(): Observable<any[]> {
    return this.http.get<any[]>('/api/data');
  }
}

// Decorators
@Injectable()
export class CachedDataService implements DataService {
  private cache: any[] | null = null;
  
  constructor(private dataService: DataService) {}
  
  getData(): Observable<any[]> {
    if (this.cache) {
      return of(this.cache);
    }
    
    return this.dataService.getData().pipe(
      tap(data => this.cache = data)
    );
  }
}

@Injectable()
export class LoggedDataService implements DataService {
  constructor(private dataService: DataService) {}
  
  getData(): Observable<any[]> {
    console.log('Fetching data...');
    return this.dataService.getData().pipe(
      tap(data => console.log('Data received:', data.length, 'items'))
    );
  }
}

// Usage with providers
{
  provide: DataService,
  useFactory: (http: HttpClient) => {
    const basic = new BasicDataService(http);
    const cached = new CachedDataService(basic);
    return new LoggedDataService(cached);
  },
  deps: [HttpClient]
}
```

### Facade Pattern

```typescript
// Complex subsystems
@Injectable({ providedIn: 'root' })
export class AuthenticationService {
  login(credentials: Credentials): Observable<AuthToken> {
    // Complex authentication logic
    return of({ token: 'abc123' });
  }
}

@Injectable({ providedIn: 'root' })
export class UserProfileService {
  loadProfile(userId: string): Observable<UserProfile> {
    return of({ id: userId, name: 'John Doe' });
  }
}

@Injectable({ providedIn: 'root' })
export class PermissionsService {
  loadPermissions(userId: string): Observable<string[]> {
    return of(['read', 'write']);
  }
}

@Injectable({ providedIn: 'root' })
export class PreferencesService {
  loadPreferences(userId: string): Observable<Preferences> {
    return of({ theme: 'dark', language: 'en' });
  }
}

// Facade - Simplified interface
@Injectable({ providedIn: 'root' })
export class UserFacade {
  constructor(
    private auth: AuthenticationService,
    private profile: UserProfileService,
    private permissions: PermissionsService,
    private preferences: PreferencesService
  ) {}
  
  login(credentials: Credentials): Observable<UserSession> {
    return this.auth.login(credentials).pipe(
      switchMap(token => 
        forkJoin({
          profile: this.profile.loadProfile(token.userId),
          permissions: this.permissions.loadPermissions(token.userId),
          preferences: this.preferences.loadPreferences(token.userId),
          token: of(token)
        })
      ),
      map(({ profile, permissions, preferences, token }) => ({
        profile,
        permissions,
        preferences,
        token
      }))
    );
  }
}

// Simple usage in component
@Component({
  selector: 'app-login',
  template: `...`
})
export class LoginComponent {
  constructor(private userFacade: UserFacade) {}
  
  login(credentials: Credentials) {
    this.userFacade.login(credentials).subscribe(session => {
      // Everything loaded in one call
      console.log('Session ready:', session);
    });
  }
}
```

---

## 3. State Management Patterns {#state-management}

### Simple State Service Pattern

```typescript
export interface AppState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

@Injectable({ providedIn: 'root' })
export class StateService {
  private state = new BehaviorSubject<AppState>({
    user: null,
    loading: false,
    error: null
  });
  
  public state$ = this.state.asObservable();
  
  // Selectors
  public user$ = this.state$.pipe(map(state => state.user));
  public loading$ = this.state$.pipe(map(state => state.loading));
  public error$ = this.state$.pipe(map(state => state.error));
  
  // Actions
  setUser(user: User): void {
    this.updateState({ user });
  }
  
  setLoading(loading: boolean): void {
    this.updateState({ loading });
  }
  
  setError(error: string | null): void {
    this.updateState({ error });
  }
  
  reset(): void {
    this.state.next({
      user: null,
      loading: false,
      error: null
    });
  }
  
  private updateState(partial: Partial<AppState>): void {
    this.state.next({ ...this.state.value, ...partial });
  }
  
  getState(): AppState {
    return this.state.value;
  }
}
```

### Store Pattern (Mini Redux)

```typescript
// Action types
export enum ActionType {
  ADD_ITEM = '[Items] Add Item',
  REMOVE_ITEM = '[Items] Remove Item',
  UPDATE_ITEM = '[Items] Update Item',
  LOAD_ITEMS = '[Items] Load Items',
  LOAD_ITEMS_SUCCESS = '[Items] Load Items Success'
}

// Actions
export interface Action {
  type: ActionType;
  payload?: any;
}

// State
export interface ItemsState {
  items: Item[];
  loading: boolean;
}

// Reducer
export function itemsReducer(
  state: ItemsState = { items: [], loading: false },
  action: Action
): ItemsState {
  switch (action.type) {
    case ActionType.ADD_ITEM:
      return {
        ...state,
        items: [...state.items, action.payload]
      };
      
    case ActionType.REMOVE_ITEM:
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload)
      };
      
    case ActionType.UPDATE_ITEM:
      return {
        ...state,
        items: state.items.map(item =>
          item.id === action.payload.id ? action.payload : item
        )
      };
      
    case ActionType.LOAD_ITEMS:
      return { ...state, loading: true };
      
    case ActionType.LOAD_ITEMS_SUCCESS:
      return {
        items: action.payload,
        loading: false
      };
      
    default:
      return state;
  }
}

// Store
@Injectable({ providedIn: 'root' })
export class Store {
  private state = new BehaviorSubject<ItemsState>({ items: [], loading: false });
  public state$ = this.state.asObservable();
  
  dispatch(action: Action): void {
    const currentState = this.state.value;
    const newState = itemsReducer(currentState, action);
    this.state.next(newState);
  }
  
  select<T>(selector: (state: ItemsState) => T): Observable<T> {
    return this.state$.pipe(
      map(selector),
      distinctUntilChanged()
    );
  }
}

// Usage in component
@Component({
  selector: 'app-items',
  template: `
    <div *ngFor="let item of items$ | async">{{ item.name }}</div>
    <button (click)="addItem()">Add Item</button>
  `
})
export class ItemsComponent {
  items$ = this.store.select(state => state.items);
  
  constructor(private store: Store) {}
  
  addItem() {
    this.store.dispatch({
      type: ActionType.ADD_ITEM,
      payload: { id: Date.now(), name: 'New Item' }
    });
  }
}
```

---

## 4. Component Architecture {#component-architecture}

### Smart vs Presentational Components

```typescript
// Presentational Component (Dumb/Pure)
@Component({
  selector: 'app-user-card',
  template: `
    <div class="card">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
      <button (click)="onEdit.emit(user)">Edit</button>
      <button (click)="onDelete.emit(user.id)">Delete</button>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserCardComponent {
  @Input() user!: User;
  @Output() onEdit = new EventEmitter<User>();
  @Output() onDelete = new EventEmitter<number>();
}

// Smart Component (Container)
@Component({
  selector: 'app-user-list',
  template: `
    <app-user-card
      *ngFor="let user of users$ | async"
      [user]="user"
      (onEdit)="editUser($event)"
      (onDelete)="deleteUser($event)">
    </app-user-card>
  `
})
export class UserListComponent implements OnInit {
  users$: Observable<User[]>;
  
  constructor(private userService: UserService) {}
  
  ngOnInit() {
    this.users$ = this.userService.getUsers();
  }
  
  editUser(user: User) {
    // Handle edit logic
    this.userService.updateUser(user).subscribe();
  }
  
  deleteUser(id: number) {
    // Handle delete logic
    this.userService.deleteUser(id).subscribe();
  }
}
```

### Component Communication Patterns

```typescript
// 1. Parent to Child via @Input
@Component({
  selector: 'app-child',
  template: `<p>{{ message }}</p>`
})
export class ChildComponent {
  @Input() message: string;
}

// 2. Child to Parent via @Output
@Component({
  selector: 'app-child',
  template: `<button (click)="sendMessage()">Send</button>`
})
export class ChildComponent {
  @Output() messageEvent = new EventEmitter<string>();
  
  sendMessage() {
    this.messageEvent.emit('Hello from child');
  }
}

// 3. Sibling Communication via Service
@Injectable({ providedIn: 'root' })
export class MessageService {
  private messageSource = new Subject<string>();
  message$ = this.messageSource.asObservable();
  
  sendMessage(message: string) {
    this.messageSource.next(message);
  }
}

// 4. ViewChild for direct access
@Component({
  selector: 'app-parent',
  template: `<app-child #childComponent></app-child>`
})
export class ParentComponent {
  @ViewChild('childComponent') child: ChildComponent;
  
  ngAfterViewInit() {
    this.child.someMethod();
  }
}
```

### Dynamic Component Loading

```typescript
@Component({
  selector: 'app-dynamic-host',
  template: `<ng-container #dynamicContainer></ng-container>`
})
export class DynamicHostComponent implements OnInit {
  @ViewChild('dynamicContainer', { read: ViewContainerRef })
  container: ViewContainerRef;
  
  constructor(
    private componentFactoryResolver: ComponentFactoryResolver
  ) {}
  
  ngOnInit() {
    this.loadComponent(DynamicComponent);
  }
  
  loadComponent(component: Type<any>) {
    this.container.clear();
    const componentRef = this.container.createComponent(component);
    
    // Pass data to dynamic component
    componentRef.instance.data = { message: 'Hello' };
  }
}
```

---

## 5. Service Layer Design {#service-layer-design}

### Repository Pattern

```typescript
// Generic repository interface
export interface IRepository<T> {
  getAll(): Observable<T[]>;
  getById(id: number): Observable<T>;
  create(item: T): Observable<T>;
  update(id: number, item: T): Observable<T>;
  delete(id: number): Observable<void>;
}

// Base repository implementation
export abstract class BaseRepository<T> implements IRepository<T> {
  constructor(
    protected http: HttpClient,
    protected endpoint: string
  ) {}
  
  getAll(): Observable<T[]> {
    return this.http.get<T[]>(this.endpoint);
  }
  
  getById(id: number): Observable<T> {
    return this.http.get<T>(`${this.endpoint}/${id}`);
  }
  
  create(item: T): Observable<T> {
    return this.http.post<T>(this.endpoint, item);
  }
  
  update(id: number, item: T): Observable<T> {
    return this.http.put<T>(`${this.endpoint}/${id}`, item);
  }
  
  delete(id: number): Observable<void> {
    return this.http.delete<void>(`${this.endpoint}/${id}`);
  }
}

// Specific repository
@Injectable({ providedIn: 'root' })
export class UserRepository extends BaseRepository<User> {
  constructor(http: HttpClient) {
    super(http, '/api/users');
  }
  
  // Add custom methods
  findByEmail(email: string): Observable<User> {
    return this.http.get<User>(`${this.endpoint}/search?email=${email}`);
  }
  
  getActiveUsers(): Observable<User[]> {
    return this.http.get<User[]>(`${this.endpoint}/active`);
  }
}
```

### Unit of Work Pattern

```typescript
@Injectable({ providedIn: 'root' })
export class UnitOfWork {
  private operations: Observable<any>[] = [];
  
  constructor(
    public users: UserRepository,
    public orders: OrderRepository,
    public products: ProductRepository
  ) {}
  
  addOperation(operation: Observable<any>): void {
    this.operations.push(operation);
  }
  
  commit(): Observable<any[]> {
    if (this.operations.length === 0) {
      return of([]);
    }
    
    return forkJoin(this.operations).pipe(
      tap(() => this.operations = []),
      catchError(error => {
        this.operations = [];
        return throwError(() => error);
      })
    );
  }
  
  rollback(): void {
    this.operations = [];
  }
}

// Usage
export class OrderService {
  constructor(private uow: UnitOfWork) {}
  
  createOrder(order: Order): Observable<any> {
    this.uow.ad