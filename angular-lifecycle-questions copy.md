# Angular Lifecycle Hooks - 25 Interview Questions

## Table of Contents
- [Lifecycle Basics](#lifecycle-basics)
- [ngOnInit vs Constructor](#ngoninit-vs-constructor)
- [ngOnChanges Deep Dive](#ngonchanges-deep-dive)
- [ngDoCheck Performance](#ngdocheck-performance)
- [Content vs View Children](#content-vs-view-children)
- [ngOnDestroy & Cleanup](#ngondestroy--cleanup)
- [Advanced Patterns](#advanced-patterns)
- [Performance Optimization](#performance-optimization)
- [Common Pitfalls](#common-pitfalls)
- [Best Practices](#best-practices)

---

## Lifecycle Basics

### Q1: What is the complete lifecycle hook execution order in Angular?

**Difficulty:** Easy

**Answer:**
```typescript
@Component({
  selector: 'app-lifecycle-demo',
  template: '<p>{{ message }}</p>'
})
export class LifecycleDemoComponent implements 
  OnChanges, OnInit, DoCheck, AfterContentInit, 
  AfterContentChecked, AfterViewInit, AfterViewChecked, OnDestroy {
  
  @Input() data: any;
  message = '';
  
  constructor() {
    console.log('1. Constructor called');
    // Component instance created
    // Dependency injection happens here
  }
  
  ngOnChanges(changes: SimpleChanges) {
    console.log('2. ngOnChanges called', changes);
    // Called BEFORE ngOnInit
    // Called when @Input properties change
    // Can be called multiple times
  }
  
  ngOnInit() {
    console.log('3. ngOnInit called');
    // Called ONCE after first ngOnChanges
    // Component initialized
    // Best place for initialization logic
  }
  
  ngDoCheck() {
    console.log('4. ngDoCheck called');
    // Called on every change detection run
    // Use with caution - performance impact
  }
  
  ngAfterContentInit() {
    console.log('5. ngAfterContentInit called');
    // Called ONCE after content (ng-content) initialized
    // Content children are available
  }
  
  ngAfterContentChecked() {
    console.log('6. ngAfterContentChecked called');
    // Called after every check of content
    // Called after ngDoCheck
  }
  
  ngAfterViewInit() {
    console.log('7. ngAfterViewInit called');
    // Called ONCE after view initialized
    // View children are available (@ViewChild)
    // DOM is ready
  }
  
  ngAfterViewChecked() {
    console.log('8. ngAfterViewChecked called');
    // Called after every check of view
    // Called after ngAfterContentChecked
  }
  
  ngOnDestroy() {
    console.log('9. ngOnDestroy called');
    // Called ONCE before component destruction
    // Cleanup subscriptions, timers, etc.
  }
}
```

**Execution Order:**
```
1. constructor()
2. ngOnChanges()      ← First time only (if @Input exists)
3. ngOnInit()         ← Once
4. ngDoCheck()        ← Every CD cycle
5. ngAfterContentInit() ← Once
6. ngAfterContentChecked() ← Every CD cycle
7. ngAfterViewInit()  ← Once
8. ngAfterViewChecked() ← Every CD cycle

[Change Detection Cycles...]
   ngOnChanges()      ← When @Input changes
   ngDoCheck()
   ngAfterContentChecked()
   ngAfterViewChecked()

9. ngOnDestroy()      ← Once (component destroyed)
```

**Key Points:**
- Constructor is NOT a lifecycle hook (TypeScript feature)
- ngOnChanges is the only hook that receives parameters
- Some hooks run once, others run on every change detection
- ngOnDestroy is crucial for cleanup

---

## ngOnInit vs Constructor

### Q2: What's the difference between constructor and ngOnInit?

**Difficulty:** Medium

**Code:**
```typescript
// ❌ BAD: DOM manipulation in constructor
@Component({
  selector: 'app-bad-practice',
  template: '<div #myDiv>Content</div>'
})
export class BadPracticeComponent {
  @ViewChild('myDiv') myDiv: ElementRef;
  @Input() title: string;
  
  constructor(private http: HttpClient) {
    console.log('Constructor');
    
    // ❌ WRONG: @Input not available yet
    console.log(this.title); // undefined
    
    // ❌ WRONG: @ViewChild not available yet
    console.log(this.myDiv); // undefined
    
    // ❌ WRONG: Too early for API calls
    this.http.get('/api/data').subscribe(); // Anti-pattern
  }
}

// ✅ GOOD: Proper separation of concerns
@Component({
  selector: 'app-good-practice',
  template: '<div #myDiv>{{ title }}</div>'
})
export class GoodPracticeComponent implements OnInit, AfterViewInit {
  @ViewChild('myDiv') myDiv: ElementRef;
  @Input() title: string;
  
  constructor(private http: HttpClient) {
    console.log('Constructor');
    // ✅ GOOD: Only dependency injection
    // ✅ GOOD: Initialize simple properties
    // ✅ GOOD: No side effects
  }
  
  ngOnInit() {
    console.log('ngOnInit');
    // ✅ GOOD: @Input is available
    console.log(this.title); // Works!
    
    // ✅ GOOD: Perfect place for API calls
    this.http.get('/api/data').subscribe(data => {
      // Handle data
    });
    
    // ✅ GOOD: Component initialization logic
    this.setupForm();
    this.loadInitialData();
  }
  
  ngAfterViewInit() {
    // ✅ GOOD: @ViewChild is available here
    console.log(this.myDiv.nativeElement); // Works!
  }
}
```

**Answer:**

| Aspect | Constructor | ngOnInit |
|--------|------------|----------|
| **When called** | When instance created | After first ngOnChanges |
| **@Input available** | ❌ No | ✅ Yes |
| **@ViewChild available** | ❌ No | ❌ No (use AfterViewInit) |
| **Purpose** | Dependency injection | Component initialization |
| **API calls** | ❌ Avoid | ✅ Perfect place |
| **Times called** | Once | Once |
| **Angular feature** | No (TypeScript) | Yes (Angular) |

**Best Practices:**
```typescript
@Component({})
export class BestPracticeComponent implements OnInit {
  // ✅ Constructor: Dependency injection only
  constructor(
    private service: DataService,
    private router: Router,
    private route: ActivatedRoute
  ) {
    // Minimal or no logic here
  }
  
  // ✅ ngOnInit: Initialization logic
  ngOnInit() {
    // Load data
    this.loadData();
    
    // Setup subscriptions
    this.setupSubscriptions();
    
    // Initialize properties based on @Input
    this.processInputs();
  }
}
```

---

### Q3: Why can't we make the constructor async in Angular?

**Difficulty:** Hard

**Code:**
```typescript
// ❌ SYNTAX ERROR: Can't make constructor async
@Component({})
export class BadComponent {
  // This won't compile!
  async constructor(private http: HttpClient) {
    const data = await this.http.get('/api/data').toPromise();
  }
}

// ✅ SOLUTION 1: Use ngOnInit with async
@Component({})
export class Solution1Component implements OnInit {
  data: any;
  
  constructor(private http: HttpClient) {}
  
  async ngOnInit() {
    // ✅ GOOD: ngOnInit can be async
    this.data = await this.http.get('/api/data').toPromise();
  }
}

// ✅ SOLUTION 2: Use promises in ngOnInit
@Component({})
export class Solution2Component implements OnInit {
  constructor(private http: HttpClient) {}
  
  ngOnInit() {
    this.http.get('/api/data').toPromise()
      .then(data => {
        this.data = data;
      })
      .catch(error => {
        console.error('Error:', error);
      });
  }
}

// ✅ SOLUTION 3: Use observables (most Angular way)
@Component({
  template: '<div *ngIf="data$ | async as data">{{ data.name }}</div>'
})
export class Solution3Component implements OnInit {
  data$: Observable<any>;
  
  constructor(private http: HttpClient) {}
  
  ngOnInit() {
    // ✅ BEST: Reactive approach
    this.data$ = this.http.get('/api/data');
  }
}

// ✅ SOLUTION 4: Use resolver (best for route data)
@Injectable()
export class DataResolver implements Resolve<any> {
  constructor(private http: HttpClient) {}
  
  resolve(route: ActivatedRouteSnapshot): Observable<any> {
    return this.http.get('/api/data');
  }
}

// Route configuration
const routes: Routes = [{
  path: 'page',
  component: PageComponent,
  resolve: { data: DataResolver }
}];

@Component({})
export class PageComponent implements OnInit {
  data: any;
  
  constructor(private route: ActivatedRoute) {}
  
  ngOnInit() {
    // Data already loaded by resolver
    this.data = this.route.snapshot.data['data'];
  }
}
```

**Answer:**
Constructors can't be async because:
1. **TypeScript/JavaScript limitation** - constructors must return an instance
2. **Angular needs immediate instance** - can't wait for async operations
3. **Dependency injection** - must complete synchronously
4. **Component tree construction** - happens synchronously

**Why it matters:**
- Constructor creates the object instance
- Angular needs the instance immediately for DI
- Lifecycle hooks are designed for async operations
- ngOnInit is the proper place for async initialization

---

## ngOnChanges Deep Dive

### Q4: How does ngOnChanges work and when is it called?

**Difficulty:** Medium

**Code:**
```typescript
// Parent Component
@Component({
  selector: 'app-parent',
  template: `
    <app-child 
      [name]="userName" 
      [age]="userAge"
      [address]="userAddress"
      [hobbies]="userHobbies">
    </app-child>
    
    <button (click)="changePrimitive()">Change Name</button>
    <button (click)="changeObject()">Change Address</button>
    <button (click)="mutateObject()">Mutate Address</button>
    <button (click)="changeArray()">Change Hobbies</button>
  `
})
export class ParentComponent {
  userName = 'John';
  userAge = 25;
  userAddress = { city: 'New York', zip: '10001' };
  userHobbies = ['reading', 'gaming'];
  
  changePrimitive() {
    this.userName = 'Jane'; // ✅ ngOnChanges triggered
    this.userAge = 26;      // ✅ ngOnChanges triggered
  }
  
  changeObject() {
    // ✅ ngOnChanges triggered (new reference)
    this.userAddress = { city: 'Boston', zip: '02101' };
  }
  
  mutateObject() {
    // ❌ ngOnChanges NOT triggered (same reference)
    this.userAddress.city = 'Chicago';
  }
  
  changeArray() {
    // ✅ ngOnChanges triggered (new reference)
    this.userHobbies = [...this.userHobbies, 'cooking'];
  }
}

// Child Component
@Component({
  selector: 'app-child',
  template: `
    <div>
      <p>Name: {{ name }}</p>
      <p>Age: {{ age }}</p>
      <p>City: {{ address.city }}</p>
      <p>Hobbies: {{ hobbies.join(', ') }}</p>
    </div>
  `
})
export class ChildComponent implements OnChanges, OnInit {
  @Input() name: string;
  @Input() age: number;
  @Input() address: { city: string; zip: string };
  @Input() hobbies: string[];
  
  ngOnChanges(changes: SimpleChanges) {
    console.log('ngOnChanges called');
    
    // Check if specific input changed
    if (changes['name']) {
      console.log('Name changed:');
      console.log('  Previous:', changes['name'].previousValue);
      console.log('  Current:', changes['name'].currentValue);
      console.log('  First change?', changes['name'].firstChange);
    }
    
    if (changes['address']) {
      console.log('Address changed:');
      console.log('  Previous:', changes['address'].previousValue);
      console.log('  Current:', changes['address'].currentValue);
      
      // ⚠️ Note: This won't detect mutations
      // Only detects reference changes
    }
    
    // Access all changes
    Object.keys(changes).forEach(key => {
      const change = changes[key];
      console.log(`${key} changed:`, {
        previous: change.previousValue,
        current: change.currentValue,
        firstChange: change.firstChange
      });
    });
  }
  
  ngOnInit() {
    console.log('ngOnInit called');
    // ngOnInit is called AFTER first ngOnChanges
  }
}
```

**When ngOnChanges is Called:**
```typescript
// ✅ Triggers ngOnChanges
@Input() primitive: string;
this.primitive = 'new value';  // New primitive value

@Input() object: any;
this.object = { ...this.object }; // New object reference
this.object = { new: 'object' };  // New object

@Input() array: any[];
this.array = [...this.array];     // New array reference
this.array = [1, 2, 3];           // New array

// ❌ Does NOT trigger ngOnChanges
@Input() object: any;
this.object.property = 'value';    // Mutation (same reference)

@Input() array: any[];
this.array.push(item);             // Mutation (same reference)
this.array[0] = 'value';           // Mutation (same reference)
```

**Key Points:**
- ngOnChanges only detects **reference changes**
- Called **before ngOnInit** (first time)
- Called whenever **@Input properties change**
- Receives `SimpleChanges` object with old/new values
- Does NOT detect object/array mutations

---

### Q5: How to detect deep object changes in ngOnChanges?

**Difficulty:** Hard

**Code:**
```typescript
// Problem: ngOnChanges doesn't detect mutations
@Component({
  selector: 'app-user-profile',
  template: '<div>{{ user.name }} - {{ user.address.city }}</div>'
})
export class UserProfileComponent implements OnChanges {
  @Input() user: User;
  
  ngOnChanges(changes: SimpleChanges) {
    // ❌ This won't detect user.name or user.address.city changes
    console.log('User changed:', changes['user']);
  }
}

// ✅ SOLUTION 1: Use immutable patterns in parent
@Component({})
export class ParentComponent {
  user = { name: 'John', address: { city: 'NYC' } };
  
  updateUserName(newName: string) {
    // ✅ Create new object reference
    this.user = {
      ...this.user,
      name: newName
    };
  }
  
  updateCity(newCity: string) {
    // ✅ Create new nested object reference
    this.user = {
      ...this.user,
      address: {
        ...this.user.address,
        city: newCity
      }
    };
  }
}

// ✅ SOLUTION 2: Use ngDoCheck for deep comparison
@Component({
  selector: 'app-user-profile',
  template: '<div>{{ user.name }} - {{ user.address.city }}</div>'
})
export class UserProfileWithDoCheckComponent implements OnChanges, DoCheck {
  @Input() user: User;
  private previousUser: User;
  
  ngOnChanges(changes: SimpleChanges) {
    console.log('Reference changed');
  }
  
  ngDoCheck() {
    // ⚠️ Performance warning: This runs on EVERY change detection
    if (this.hasUserChanged()) {
      console.log('Deep change detected');
      this.previousUser = JSON.parse(JSON.stringify(this.user));
      this.onUserDeepChange();
    }
  }
  
  private hasUserChanged(): boolean {
    return JSON.stringify(this.user) !== JSON.stringify(this.previousUser);
  }
  
  private onUserDeepChange() {
    // React to deep changes
  }
}

// ✅ SOLUTION 3: Use setter for @Input
@Component({
  selector: 'app-user-profile',
  template: '<div>{{ displayUser.name }}</div>'
})
export class UserProfileWithSetterComponent {
  private _user: User;
  displayUser: User;
  
  @Input()
  set user(value: User) {
    console.log('User setter called');
    this._user = value;
    
    // Deep clone to detect future changes
    this.displayUser = JSON.parse(JSON.stringify(value));
    
    // Custom logic when user changes
    this.processUser(value);
  }
  
  get user(): User {
    return this._user;
  }
  
  private processUser(user: User) {
    // React to user changes
  }
}

// ✅ SOLUTION 4: Use RxJS for reactive changes
@Component({
  selector: 'app-user-profile',
  template: '<div *ngIf="user$ | async as user">{{ user.name }}</div>'
})
export class UserProfileReactiveComponent implements OnInit, OnDestroy {
  @Input() set user(value: User) {
    this.userSubject.next(value);
  }
  
  private userSubject = new BehaviorSubject<User>(null);
  user$ = this.userSubject.asObservable();
  private destroy$ = new Subject<void>();
  
  ngOnInit() {
    // React to user changes reactively
    this.user$.pipe(
      filter(user => !!user),
      distinctUntilChanged((prev, curr) => 
        JSON.stringify(prev) === JSON.stringify(curr)
      ),
      takeUntil(this.destroy$)
    ).subscribe(user => {
      console.log('User changed (deep):', user);
      this.processUser(user);
    });
  }
  
  private processUser(user: User) {
    // React to user changes
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// ✅ SOLUTION 5: Use ChangeDetectorRef with OnPush
@Component({
  selector: 'app-user-profile',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: '<div>{{ user.name }}</div>'
})
export class UserProfileOnPushComponent {
  @Input() user: User;
  
  constructor(private cdr: ChangeDetectorRef) {}
  
  // Manual change detection when needed
  refreshUser() {
    this.cdr.markForCheck();
  }
}
```

**Best Practices:**
1. **Prefer immutable patterns** - always create new references
2. **Avoid ngDoCheck** - performance impact (runs every CD)
3. **Use setters** - for custom logic on input changes
4. **Consider RxJS** - reactive approach for complex scenarios
5. **OnPush strategy** - forces immutable patterns

---

## ngDoCheck Performance

### Q6: What are the performance implications of ngDoCheck?

**Difficulty:** Hard

**Code:**
```typescript
// ❌ DANGEROUS: ngDoCheck runs on EVERY change detection
@Component({
  selector: 'app-expensive-check',
  template: '<div>{{ items.length }} items</div>'
})
export class ExpensiveCheckComponent implements DoCheck {
  @Input() items: any[];
  private previousItems: any[];
  
  ngDoCheck() {
    // ❌ VERY EXPENSIVE: Deep comparison on every CD cycle
    if (JSON.stringify(this.items) !== JSON.stringify(this.previousItems)) {
      console.log('Items changed');
      this.previousItems = JSON.parse(JSON.stringify(this.items));
      this.processItems();
    }
  }
  
  private processItems() {
    // Heavy computation
  }
}

// Performance Impact:
// - ngDoCheck called 10-50+ times per second
// - JSON.stringify on large arrays = 10-100ms per call
// - Result: UI freeze, poor performance

// ✅ SOLUTION 1: Use IterableDiffer for arrays
@Component({})
export class OptimizedArrayCheckComponent implements DoCheck {
  @Input() items: any[];
  private differ: IterableDiffer<any>;
  
  constructor(private differs: IterableDiffers) {
    this.differ = this.differs.find([]).create(null);
  }
  
  ngDoCheck() {
    const changes = this.differ.diff(this.items);
    if (changes) {
      console.log('Array changed');
      changes.forEachAddedItem(item => {
        console.log('Added:', item.item);
      });
      changes.forEachRemovedItem(item => {
        console.log('Removed:', item.item);
      });
    }
  }
}

// ✅ SOLUTION 2: Use KeyValueDiffer for objects
@Component({})
export class OptimizedObjectCheckComponent implements DoCheck {
  @Input() config: any;
  private differ: KeyValueDiffer<string, any>;
  
  constructor(private differs: KeyValueDiffers) {
    this.differ = this.differs.find({}).create();
  }
  
  ngDoCheck() {
    const changes = this.differ.diff(this.config);
    if (changes) {
      console.log('Config changed');
      changes.forEachChangedItem(item => {
        console.log(`${item.key}: ${item.previousValue} → ${item.currentValue}`);
      });
    }
  }
}

// ✅ SOLUTION 3: Avoid ngDoCheck - use OnPush + immutability
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class OnPushComponent implements OnChanges {
  @Input() items: any[];
  
  ngOnChanges(changes: SimpleChanges) {
    // ✅ Only called when reference changes
    if (changes['items']) {
      this.processItems();
    }
  }
  
  private processItems() {
    // This runs much less frequently
  }
}

// ✅ SOLUTION 4: Throttle/debounce ngDoCheck
@Component({})
export class ThrottledCheckComponent implements DoCheck, OnDestroy {
  @Input() data: any;
  private checkScheduled = false;
  private destroy$ = new Subject<void>();
  
  ngDoCheck() {
    if (!this.checkScheduled) {
      this.checkScheduled = true;
      
      // Throttle to max once per 100ms
      setTimeout(() => {
        this.checkScheduled = false;
        this.actualCheck();
      }, 100);
    }
  }
  
  private actualCheck() {
    // Your actual checking logic here
    console.log('Throttled check');
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**Performance Comparison:**

| Approach | CD Cycles/sec | Performance Impact |
|----------|---------------|-------------------|
| ngDoCheck + JSON.stringify | 50+ | ❌ Severe (100-500ms) |
| ngDoCheck + IterableDiffer | 50+ | ⚠️ Moderate (5-20ms) |
| ngOnChanges + OnPush | 1-5 | ✅ Minimal (<1ms) |
| Immutable + OnPush | 1-2 | ✅ Excellent (<0.5ms) |

**When to Use ngDoCheck:**
- ✅ Detecting changes in third-party libraries
- ✅ Custom change detection logic (use Differs)
- ❌ Regular input change detection (use ngOnChanges)
- ❌ Performance-critical components

---

### Q7: How to properly use IterableDiffer and KeyValueDiffer?

**Difficulty:** Hard

**Code:**
```typescript
// Complete example with both differs
@Component({
  selector: 'app-differ-demo',
  template: `
    <div>
      <h3>Array Changes:</h3>
      <ul>
        <li *ngFor="let item of items">{{ item.name }}</li>
      </ul>
      
      <h3>Object Changes:</h3>
      <div *ngFor="let key of objectKeys(config)">
        {{ key }}: {{ config[key] }}
      </div>
    </div>
  `
})
export class DifferDemoComponent implements DoCheck, OnInit {
  @Input() items: any[];
  @Input() config: Record<string, any>;
  
  private itemsDiffer: IterableDiffer<any>;
  private configDiffer: KeyValueDiffer<string, any>;
  
  constructor(
    private iterableDiffers: IterableDiffers,
    private keyValueDiffers: KeyValueDiffers
  ) {}
  
  ngOnInit() {
    // ✅ Initialize differs in ngOnInit
    this.itemsDiffer = this.iterableDiffers.find(this.items || []).create(null);
    this.configDiffer = this.keyValueDiffers.find(this.config || {}).create();
  }
  
  ngDoCheck() {
    this.checkArrayChanges();
    this.checkObjectChanges();
  }
  
  private checkArrayChanges() {
    const changes = this.itemsDiffer.diff(this.items);
    
    if (changes) {
      console.log('=== Array Changes Detected ===');
      
      // Detect additions
      changes.forEachAddedItem(record => {
        console.log('Added at index', record.currentIndex, ':', record.item);
        this.onItemAdded(record.item, record.currentIndex);
      });
      
      // Detect removals
      changes.forEachRemovedItem(record => {
        console.log('Removed from index', record.previousIndex, ':', record.item);
        this.onItemRemoved(record.item, record.previousIndex);
      });
      
      // Detect moves
      changes.forEachMovedItem(record => {
        console.log('Moved from', record.previousIndex, 'to', record.currentIndex);
        this.onItemMoved(record.item, record.previousIndex, record.currentIndex);
      });
      
      // Detect identity changes
      changes.forEachIdentityChange(record => {
        console.log('Identity changed at', record.currentIndex);
      });
    }
  }
  
  private checkObjectChanges() {
    const changes = this.configDiffer.diff(this.config);
    
    if (changes) {
      console.log('=== Object Changes Detected ===');
      
      // Detect property additions
      changes.forEachAddedItem(record => {
        console.log(`Added: ${record.key} = ${record.currentValue}`);
        this.onConfigAdded(record.key, record.currentValue);
      });
      
      // Detect property removals
      changes.forEachRemovedItem(record => {
        console.log(`Removed: ${record.key} was ${record.previousValue}`);
        this.onConfigRemoved(record.key, record.previousValue);
      });
      
      // Detect property changes
      changes.forEachChangedItem(record => {
        console.log(`Changed: ${record.key} from ${record.previousValue} to ${record.currentValue}`);
        this.onConfigChanged(record.key, record.previousValue, record.currentValue);
      });
    }
  }
  
  // Callback methods
  private onItemAdded(item: any, index: number) {
    // Handle item addition
  }
  
  private onItemRemoved(item: any, index: number) {
    // Handle item removal
  }
  
  private onItemMoved(item: any, fromIndex: number, toIndex: number) {
    // Handle item reordering
  }
  
  private onConfigAdded(key: string, value: any) {
    // Handle config property addition
  }
  
  private onConfigRemoved(key: string, value: any) {
    // Handle config property removal
  }
  
  private onConfigChanged(key: string, oldValue: any, newValue: any) {
    // Handle config property change
  }
  
  objectKeys(obj: any): string[] {
    return Object.keys(obj || {});
  }
}

// ✅ Advanced: Custom TrackBy with IterableDiffer
@Component({
  selector: 'app-custom-trackby',
  template: `
    <div *ngFor="let item of items; trackBy: trackByFn">
      {{ item.id }}: {{ item.name }}
    </div>
  `
})
export class CustomTrackByComponent implements DoCheck {
  @Input() items: Array<{id: number; name: string}>;
  private differ: IterableDiffer<any>;
  
  constructor(private differs: IterableDiffers) {
    // ✅ Custom trackBy function for differ
    this.differ = this.differs.find([]).create((index, item) => item.id);
  }
  
  ngDoCheck() {
    const changes = this.differ.diff(this.items);
    if (changes) {
      // Only detects changes based on item.id
      console.log('Items changed (tracked by id)');
    }
  }
  
  trackByFn(index: number, item: any): any {
    return item.id; // Track by id, not by reference
  }
}

// ✅ Real-world example: Form array with differ
@Component({
  selector: 'app-dynamic-form',
  template: `
    <form [formGroup]="form">
      <div formArrayName="fields">
        <div *ngFor="let field of fields.controls; let i = index">
          <input [formControlName]="i">
        </div>
      </div>
    </form>
  `
})
export class DynamicFormComponent implements DoCheck {
  form = new FormGroup({
    fields: new FormArray([])
  });
  
  private differ: IterableDiffer<AbstractControl>;
  
  get fields(): FormArray {
    return this.form.get('fields') as FormArray;
  }
  
  constructor(private differs: IterableDiffers) {
    this.differ = this.differs.