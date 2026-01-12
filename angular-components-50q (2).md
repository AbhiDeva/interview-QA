# Angular Components - 50 Interview Questions

## Table of Contents
1. [Component Basics (Q1-Q10)](#component-basics)
2. [Component Communication (Q11-Q20)](#component-communication)
3. [Input/Output Deep Dive (Q21-Q25)](#inputoutput-deep-dive)
4. [ViewChild & ContentChild (Q26-Q30)](#viewchild--contentchild)
5. [Component Styling (Q31-Q35)](#component-styling)
6. [Dynamic Components (Q36-Q40)](#dynamic-components)
7. [Component Architecture (Q41-Q45)](#component-architecture)
8. [Performance & Optimization (Q46-Q50)](#performance--optimization)

---

## Component Basics

### Q1: What is an Angular component and what are its core parts?

**Difficulty:** Easy

**Code:**
```typescript
// A complete Angular component structure
@Component({
  // 1. SELECTOR: How to use the component
  selector: 'app-user-card',
  
  // 2. TEMPLATE: The HTML view
  templateUrl: './user-card.component.html',
  // OR inline template
  template: `
    <div class="card">
      <h2>{{ user.name }}</h2>
      <p>{{ user.email }}</p>
    </div>
  `,
  
  // 3. STYLES: Component styling
  styleUrls: ['./user-card.component.css'],
  // OR inline styles
  styles: [`
    .card {
      border: 1px solid #ccc;
      padding: 20px;
    }
  `],
  
  // 4. METADATA: Additional configuration
  changeDetection: ChangeDetectionStrategy.OnPush,
  encapsulation: ViewEncapsulation.Emulated,
  providers: [UserService],
  animations: []
})
export class UserCardComponent implements OnInit {
  // 5. PROPERTIES
  @Input() user: User;
  @Output() userSelected = new EventEmitter<User>();
  
  // 6. CONSTRUCTOR: Dependency injection
  constructor(private userService: UserService) {}
  
  // 7. LIFECYCLE HOOKS
  ngOnInit() {
    console.log('Component initialized');
  }
  
  // 8. METHODS
  selectUser() {
    this.userSelected.emit(this.user);
  }
}
```

**Answer:**
A component consists of:
1. **@Component decorator**: Metadata
2. **Selector**: Element name in templates
3. **Template**: HTML structure
4. **Styles**: CSS styling
5. **Class**: TypeScript logic
6. **Lifecycle hooks**: Initialization and cleanup
7. **Properties**: @Input, @Output, @ViewChild, etc.

---

### Q2: What's the difference between template and templateUrl?

**Difficulty:** Easy

**Code:**
```typescript
// Option 1: Inline template (for small templates)
@Component({
  selector: 'app-simple',
  template: '<h1>{{ title }}</h1>'
})
export class SimpleComponent {
  title = 'Hello World';
}

// Option 2: Multi-line inline template
@Component({
  selector: 'app-inline',
  template: `
    <div class="container">
      <h1>{{ title }}</h1>
      <p>{{ description }}</p>
      <button (click)="handleClick()">Click Me</button>
    </div>
  `
})
export class InlineComponent {
  title = 'Inline Template';
  description = 'Using backticks for multi-line';
}

// Option 3: External template file (recommended for larger templates)
@Component({
  selector: 'app-external',
  templateUrl: './external.component.html'
})
export class ExternalComponent {
  // Template in separate HTML file
}
```

**When to use each:**
- **template**: Simple components (<10 lines), quick prototypes
- **templateUrl**: Production code, complex templates, better IDE support

---

### Q3: What are component selectors and their types?

**Difficulty:** Easy

**Code:**
```typescript
// 1. Element selector (most common)
@Component({
  selector: 'app-button',
  template: '<button>Click me</button>'
})
export class ButtonComponent {}
// Usage: <app-button></app-button>

// 2. Attribute selector
@Component({
  selector: '[app-highlight]',
  template: '<ng-content></ng-content>'
})
export class HighlightComponent {}
// Usage: <div app-highlight>Content</div>

// 3. Class selector (rarely used)
@Component({
  selector: '.app-badge',
  template: '<span><ng-content></ng-content></span>'
})
export class BadgeComponent {}
// Usage: <span class="app-badge">Badge</span>

// 4. Combined selectors
@Component({
  selector: 'button[app-custom-button]',
  template: '<ng-content></ng-content>'
})
export class CustomButtonComponent {}
// Usage: <button app-custom-button>Custom</button>

// 5. Multiple selectors
@Component({
  selector: 'app-icon, [app-icon]',
  template: '<i class="icon"></i>'
})
export class IconComponent {}
// Usage: <app-icon></app-icon> OR <div app-icon></div>
```

**Best Practices:**
- Use element selectors for components: `app-user-card`
- Use attribute selectors for directives: `[appHighlight]`
- Avoid class selectors (not semantic)
- Use prefix to avoid naming conflicts: `app-`, `lib-`

---

### Q4: How do components work with modules?

**Difficulty:** Easy

**Code:**
```typescript
// Feature Module
@NgModule({
  // 1. DECLARATIONS: Components, directives, pipes in THIS module
  declarations: [
    UserListComponent,
    UserCardComponent,
    UserDetailComponent
  ],
  
  // 2. IMPORTS: Other modules needed by this module
  imports: [
    CommonModule,
    FormsModule,
    RouterModule
  ],
  
  // 3. EXPORTS: Make components available to other modules
  exports: [
    UserListComponent,  // Other modules can use this
    UserCardComponent   // Other modules can use this
    // UserDetailComponent NOT exported - only used internally
  ],
  
  // 4. PROVIDERS: Services available to this module
  providers: [
    UserService
  ]
})
export class UserModule {}

// Using the components in another module
@NgModule({
  declarations: [AppComponent],
  imports: [
    UserModule  // Now can use UserListComponent and UserCardComponent
  ]
})
export class AppModule {}

// In AppComponent template
@Component({
  template: `
    <app-user-list></app-user-list>  <!-- ✅ Works - exported -->
    <app-user-card></app-user-card>   <!-- ✅ Works - exported -->
    <app-user-detail></app-user-detail> <!-- ❌ Error - not exported -->
  `
})
export class AppComponent {}
```

**Key Points:**
- **declarations**: Components belong to ONE module only
- **imports**: Modules this module depends on
- **exports**: Components other modules can use
- **providers**: Services (prefer `providedIn: 'root'`)

---

### Q5: What is the component lifecycle sequence?

**Difficulty:** Medium

**Code:**
```typescript
@Component({
  selector: 'app-lifecycle-demo',
  template: '<p>{{ message }}</p>'
})
export class LifecycleDemoComponent implements 
  OnChanges, OnInit, DoCheck, 
  AfterContentInit, AfterContentChecked,
  AfterViewInit, AfterViewChecked, OnDestroy {
  
  @Input() data: any;
  message = '';
  
  constructor() {
    console.log('1. Constructor');
  }
  
  ngOnChanges(changes: SimpleChanges) {
    console.log('2. ngOnChanges', changes);
  }
  
  ngOnInit() {
    console.log('3. ngOnInit');
  }
  
  ngDoCheck() {
    console.log('4. ngDoCheck');
  }
  
  ngAfterContentInit() {
    console.log('5. ngAfterContentInit');
  }
  
  ngAfterContentChecked() {
    console.log('6. ngAfterContentChecked');
  }
  
  ngAfterViewInit() {
    console.log('7. ngAfterViewInit');
  }
  
  ngAfterViewChecked() {
    console.log('8. ngAfterViewChecked');
  }
  
  ngOnDestroy() {
    console.log('9. ngOnDestroy');
  }
}
```

**Execution Order:**
```
Initial load:
1. constructor
2. ngOnChanges (if @Input exists)
3. ngOnInit
4. ngDoCheck
5. ngAfterContentInit
6. ngAfterContentChecked
7. ngAfterViewInit
8. ngAfterViewChecked

On every change detection:
- ngOnChanges (if @Input changes)
- ngDoCheck
- ngAfterContentChecked
- ngAfterViewChecked

On destroy:
- ngOnDestroy
```

---

### Q6: What's the difference between constructor and ngOnInit?

**Difficulty:** Medium

**Code:**
```typescript
// ❌ BAD: Wrong usage
@Component({})
export class BadComponent {
  @ViewChild('myDiv') myDiv: ElementRef;
  @Input() userId: string;
  
  constructor(private http: HttpClient) {
    // ❌ @Input not available yet
    console.log(this.userId); // undefined
    
    // ❌ @ViewChild not available yet
    console.log(this.myDiv); // undefined
    
    // ❌ Too early for API calls
    this.http.get('/api/user').subscribe();
  }
}

// ✅ GOOD: Proper usage
@Component({})
export class GoodComponent implements OnInit, AfterViewInit {
  @ViewChild('myDiv') myDiv: ElementRef;
  @Input() userId: string;
  
  constructor(private http: HttpClient) {
    // ✅ ONLY dependency injection
    console.log('Dependencies injected');
  }
  
  ngOnInit() {
    // ✅ @Input is available
    console.log(this.userId); // Works!
    
    // ✅ Perfect for API calls
    this.http.get(`/api/user/${this.userId}`).subscribe();
  }
  
  ngAfterViewInit() {
    // ✅ @ViewChild is available
    console.log(this.myDiv); // Works!
  }
}
```

| Feature | Constructor | ngOnInit |
|---------|------------|----------|
| **When called** | Instance creation | After first ngOnChanges |
| **@Input available** | ❌ No | ✅ Yes |
| **@ViewChild available** | ❌ No | ❌ No (AfterViewInit) |
| **Purpose** | DI only | Initialization |
| **API calls** | ❌ Avoid | ✅ Perfect |

---

### Q7: How does standalone component work?

**Difficulty:** Medium

**Code:**
```typescript
// Traditional component (requires NgModule)
@Component({
  selector: 'app-traditional',
  template: '<p>Traditional component</p>'
})
export class TraditionalComponent {}

@NgModule({
  declarations: [TraditionalComponent],
  exports: [TraditionalComponent]
})
export class TraditionalModule {}

// ✅ Standalone component (Angular 14+)
@Component({
  selector: 'app-standalone',
  standalone: true,  // ← Key difference
  imports: [CommonModule, FormsModule],  // Import what you need
  template: '<p>Standalone component</p>'
})
export class StandaloneComponent {}

// Using standalone components
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [
    StandaloneComponent,  // Import components directly
    CommonModule,
    RouterModule
  ],
  template: '<app-standalone></app-standalone>'
})
export class AppComponent {}

// Bootstrap standalone app
bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes),
    provideHttpClient()
  ]
});
```

**Benefits of Standalone:**
- No NgModule boilerplate
- Better tree-shaking
- Simpler dependency management
- Easier lazy loading
- More explicit imports

---

### Q8: What are the view encapsulation strategies?

**Difficulty:** Medium

**Code:**
```typescript
// 1. Emulated (default)
@Component({
  selector: 'app-emulated',
  encapsulation: ViewEncapsulation.Emulated,
  template: '<div class="container">Emulated</div>',
  styles: [`
    .container {
      color: blue;
    }
  `]
})
export class EmulatedComponent {}
// Generated CSS: .container[_ngcontent-c0] { color: blue; }
// Styles are scoped to this component only

// 2. None (no encapsulation)
@Component({
  selector: 'app-none',
  encapsulation: ViewEncapsulation.None,
  template: '<div class="container">None</div>',
  styles: [`
    .container {
      color: red;
    }
  `]
})
export class NoneComponent {}
// Generated CSS: .container { color: red; }
// Styles are global - affects entire app!

// 3. ShadowDom (uses native Shadow DOM)
@Component({
  selector: 'app-shadow',
  encapsulation: ViewEncapsulation.ShadowDom,
  template: '<div class="container">Shadow</div>',
  styles: [`
    .container {
      color: green;
    }
  `]
})
export class ShadowComponent {}
// Uses browser's Shadow DOM
// True encapsulation, but limited browser support

// Example: Style isolation
@Component({
  selector: 'app-parent',
  template: `
    <app-emulated></app-emulated>
    <app-none></app-none>
    <div class="container">Parent container</div>
  `,
  styles: [`
    .container {
      color: purple;
    }
  `]
})
export class ParentComponent {}
// Result:
// - app-emulated .container: blue (isolated)
// - app-none .container: red (global - overrides parent)
// - parent .container: purple
```

| Strategy | Scoping | Use Case |
|----------|---------|----------|
| **Emulated** | Scoped | Default, most common |
| **None** | Global | Global styles, themes |
| **ShadowDom** | True isolation | Web components |

---

### Q9: How to pass data using template reference variables?

**Difficulty:** Easy

**Code:**
```typescript
@Component({
  selector: 'app-template-ref',
  template: `
    <!-- 1. Basic template reference -->
    <input #usernameInput type="text">
    <button (click)="logValue(usernameInput.value)">Log Value</button>
    
    <!-- 2. Reference to component -->
    <app-child #childComponent></app-child>
    <button (click)="childComponent.childMethod()">Call Child Method</button>
    
    <!-- 3. Reference in structural directives -->
    <div *ngFor="let item of items; let i = index" #itemElement>
      Item {{ i }}: {{ item }}
    </div>
    
    <!-- 4. Pass reference to component -->
    <input #searchInput type="text">
    <app-search [inputElement]="searchInput"></app-search>
    
    <!-- 5. Use in template expressions -->
    <input #password type="password">
    <div *ngIf="password.value.length < 8">
      Password too short: {{ password.value.length }}/8
    </div>
  `
})
export class TemplateRefComponent {
  items = ['Apple', 'Banana', 'Cherry'];
  
  logValue(value: string) {
    console.log('Input value:', value);
  }
}

@Component({
  selector: 'app-child',
  template: '<p>Child Component</p>'
})
export class ChildComponent {
  childMethod() {
    console.log('Child method called from parent!');
  }
}

@Component({
  selector: 'app-search',
  template: '<button (click)="search()">Search</button>'
})
export class SearchComponent {
  @Input() inputElement: HTMLInputElement;
  
  search() {
    console.log('Searching for:', this.inputElement.value);
  }
}
```

**Use Cases:**
- Access input values without ngModel
- Call child component methods
- Pass DOM elements to other components
- Conditional rendering based on element state

---

### Q10: What are component providers and their scope?

**Difficulty:** Hard

**Code:**
```typescript
// Service
@Injectable()
export class CounterService {
  count = 0;
  
  increment() {
    this.count++;
  }
}

// 1. Root level (singleton across entire app)
@Injectable({
  providedIn: 'root'  // Recommended for most services
})
export class GlobalService {
  // One instance for entire app
}

// 2. Module level
@NgModule({
  providers: [CounterService]  // One instance per module
})
export class FeatureModule {}

// 3. Component level (new instance per component)
@Component({
  selector: 'app-counter',
  providers: [CounterService],  // New instance for each component
  template: `
    <p>Count: {{ counterService.count }}</p>
    <button (click)="counterService.increment()">Increment</button>
  `
})
export class CounterComponent {
  constructor(public counterService: CounterService) {}
}

// Practical example: Different scopes
@Component({
  selector: 'app-parent',
  providers: [CounterService],  // Parent has its own instance
  template: `
    <h2>Parent Count: {{ counterService.count }}</h2>
    <button (click)="counterService.increment()">Parent +</button>
    
    <!-- Child 1 shares parent's service -->
    <app-child-1></app-child-1>
    
    <!-- Child 2 has its own service -->
    <app-child-2></app-child-2>
  `
})
export class ParentComponent {
  constructor(public counterService: CounterService) {}
}

@Component({
  selector: 'app-child-1',
  // No providers - uses parent's service
  template: `
    <h3>Child 1 Count: {{ counterService.count }}</h3>
    <button (click)="counterService.increment()">Child 1 +</button>
  `
})
export class Child1Component {
  constructor(public counterService: CounterService) {}
  // Gets parent's instance - shares count
}

@Component({
  selector: 'app-child-2',
  providers: [CounterService],  // Own instance
  template: `
    <h3>Child 2 Count: {{ counterService.count }}</h3>
    <button (click)="counterService.increment()">Child 2 +</button>
  `
})
export class Child2Component {
  constructor(public counterService: CounterService) {}
  // Gets its own instance - independent count
}
```

**Provider Hierarchy:**
```
Root (providedIn: 'root')
  └─ Module (NgModule providers)
      └─ Component (component providers)
          └─ Child Components (inherit unless they provide their own)
```

---

## Component Communication

### Q11: What are all the ways components can communicate?

**Difficulty:** Medium

**Code:**
```typescript
// 1️⃣ Parent → Child: @Input
@Component({
  selector: 'app-parent',
  template: '<app-child [user]="currentUser" [age]="25"></app-child>'
})
export class ParentComponent {
  currentUser = { name: 'John', email: 'john@example.com' };
}

@Component({
  selector: 'app-child',
  template: '<div>{{ user.name }} - {{ age }}</div>'
})
export class ChildComponent {
  @Input() user: User;
  @Input() age: number;
}

// 2️⃣ Child → Parent: @Output + EventEmitter
@Component({
  selector: 'app-child',
  template: '<button (click)="sendData()">Send to Parent</button>'
})
export class ChildComponent {
  @Output() dataEmit = new EventEmitter<string>();
  
  sendData() {
    this.dataEmit.emit('Hello from child');
  }
}

@Component({
  selector: 'app-parent',
  template: '<app-child (dataEmit)="handleData($event)"></app-child>'
})
export class ParentComponent {
  handleData(data: string) {
    console.log('Received:', data);
  }
}

// 3️⃣ Parent → Child: @ViewChild
@Component({
  selector: 'app-parent',
  template: '<app-child></app-child>'
})
export class ParentComponent implements AfterViewInit {
  @ViewChild(ChildComponent) child: ChildComponent;
  
  ngAfterViewInit() {
    this.child.childMethod();
    console.log(this.child.childProperty);
  }
}

// 4️⃣ Service with Subject (any component to any component)
@Injectable({ providedIn: 'root' })
export class DataService {
  private messageSubject = new BehaviorSubject<string>('');
  message$ = this.messageSubject.asObservable();
  
  sendMessage(msg: string) {
    this.messageSubject.next(msg);
  }
}

@Component({})
export class SenderComponent {
  constructor(private dataService: DataService) {}
  
  send() {
    this.dataService.sendMessage('Hello!');
  }
}

@Component({})
export class ReceiverComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  
  constructor(private dataService: DataService) {}
  
  ngOnInit() {
    this.dataService.message$.pipe(
      takeUntil(this.destroy$)
    ).subscribe(msg => console.log(msg));
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// 5️⃣ Template reference variable
@Component({
  selector: 'app-parent',
  template: `
    <app-child #childRef></app-child>
    <button (click)="childRef.childMethod()">Call Child</button>
  `
})
export class ParentComponent {}

// 6️⃣ Local storage / Session storage
@Component({})
export class Component1 {
  saveData() {
    localStorage.setItem('data', JSON.stringify({ value: 'test' }));
  }
}

@Component({})
export class Component2 implements OnInit {
  ngOnInit() {
    const data = JSON.parse(localStorage.getItem('data'));
    console.log(data);
  }
}

// 7️⃣ Route parameters
@Component({})
export class Component1 {
  constructor(private router: Router) {}
  
  navigate() {
    this.router.navigate(['/detail', 123], {
      queryParams: { name: 'John' }
    });
  }
}

@Component({})
export class Component2 implements OnInit {
  constructor(private route: ActivatedRoute) {}
  
  ngOnInit() {
    const id = this.route.snapshot.params['id'];
    const name = this.route.snapshot.queryParams['name'];
  }
}
```

**Communication Methods Summary:**

| Method | Direction | Use Case |
|--------|-----------|----------|
| @Input | Parent → Child | Pass data down |
| @Output | Child → Parent | Events up |
| @ViewChild | Parent → Child | Direct access |
| Service | Any ↔ Any | Shared state |
| Template Ref | Parent → Child | Template access |
| Storage | Any ↔ Any | Persist data |
| Router | Any → Any | Navigate with data |

---

### Q12: How to implement two-way binding in custom components?

**Difficulty:** Medium

**Code:**
```typescript
// ❌ Manual two-way binding (tedious)
@Component({
  selector: 'app-manual',
  template: `
    <input [value]="value" (input)="value = $event.target.value">
  `
})
export class ManualComponent {
  @Input() value: string;
  @Output() valueChange = new EventEmitter<string>();
  
  updateValue(newValue: string) {
    this.value = newValue;
    this.valueChange.emit(newValue);
  }
}
// Usage: <app-manual [value]="name" (valueChange)="name=$event"></app-manual>

// ✅ Banana-in-a-box syntax [(ngModel)] pattern
@Component({
  selector: 'app-custom-input',
  template: `
    <input 
      [value]="value"
      (input)="onInputChange($event.target.value)">
  `
})
export class CustomInputComponent {
  @Input() value: string;
  @Output() valueChange = new EventEmitter<string>();
  
  onInputChange(newValue: string) {
    this.value = newValue;
    this.valueChange.emit(newValue);
  }
}
// Usage: <app-custom-input [(value)]="name"></app-custom-input>

// ✅ Implementing ControlValueAccessor (forms integration)
@Component({
  selector: 'app-custom-input-advanced',
  template: `
    <input 
      type="text"
      [value]="value"
      (input)="onChange($event.target.value)"
      (blur)="onTouched()"
      [disabled]="disabled">
  `,
  providers: [{
    provide: NG_VALUE_ACCESSOR,
    useExisting: forwardRef(() => CustomInputAdvancedComponent),
    multi: true
  }]
})
export class CustomInputAdvancedComponent implements ControlValueAccessor {
  value: string = '';
  disabled: boolean = false;
  
  onChange = (value: any) => {};
  onTouched = () => {};
  
  // Called by forms API to write value to the view
  writeValue(value: any): void {
    this.value = value || '';
  }
  
  // Called by forms API to register onChange callback
  registerOnChange(fn: any): void {
    this.onChange = fn;
  }
  
  // Called by forms API to register onTouched callback
  registerOnTouched(fn: any): void {
    this.onTouched = fn;
  }
  
  // Called by forms API to set disabled state
  setDisabledState(isDisabled: boolean): void {
    this.disabled = isDisabled;
  }
}
// Usage: <app-custom-input-advanced [(ngModel)]="name"></app-custom-input-advanced>
// Or: <app-custom-input-advanced formControlName="name"></app-custom-input-advanced>

// Real-world example: Custom counter component
@Component({
  selector: 'app-counter',
  template: `
    <button (click)="decrement()" [disabled]="disabled">-</button>
    <span>{{ value }}</span>
    <button (click)="increment()" [disabled]="disabled">+</button>
  `,
  providers: [{
    provide: NG_VALUE_ACCESSOR,
    useExisting: forwardRef(() => CounterComponent),
    multi: true
  }]
})
export class CounterComponent implements ControlValueAccessor {
  value: number = 0;
  disabled: boolean = false;
  
  onChange = (value: any) => {};
  onTouched = () => {};
  
  increment() {
    this.value++;
    this.onChange(this.value);
  }
  
  decrement() {
    this.value--;
    this.onChange(this.value);
  }
  
  writeValue(value: number): void {
    this.value = value || 0;
  }
  
  registerOnChange(fn: any): void {
    this.onChange = fn;
  }
  
  registerOnTouched(fn: any): void {
    this.onTouched = fn;
  }
  
  setDisabledState(isDisabled: boolean): void {
    this.disabled = isDisabled;
  }
}
// Usage: <app-counter [(ngModel)]="count"></app-counter>
```

**Two-Way Binding Patterns:**
1. **Simple**: @Input + @Output with "Change" suffix
2. **Forms**: ControlValueAccessor interface
3. **Custom**: Any combination that updates both ways

---

### Q13: How to use @ContentChild and @ContentChildren?

**Difficulty:** Hard

**Code:**
```typescript
// Card component that projects content
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <div class="header">
        <ng-content select="[card-header]"></ng-content>
      </div>
      <div class="body">
        <ng-content select="[card-body]"></ng-content>
      </div>
      <div class="footer">
        <ng-content select="[card-footer]"></ng-content>
      </div>
    </div>
  `
})
export class CardComponent implements AfterContentInit {
  // Query single projected content
  @ContentChild('headerTemplate') headerTemplate: TemplateRef<any>;
  
  // Query multiple projected elements
  @ContentChildren('cardButton') buttons: QueryList<ElementRef>;
  
  ngAfterContentInit() {
    console.log('Header template:', this.headerTemplate);
    console.log('Number of buttons:', this.buttons.length);
    
    // Listen to changes
    this.buttons.changes.subscribe(buttons => {
      console.log('Buttons changed:', buttons.length);
    });
  }
}

// Usage
@Component({
  template: `
    <app-card>
      <div card-header>
        <ng-template #headerTemplate>
          <h2>Card Title</h2>
        </ng-template>
      </div>
      
      <div card-body>
        <p>Card content here</p>
        <button #cardButton>Button 1</button>
        <button #cardButton>Button 2</button>
      </div>
      
      <div card-footer>
        <button #cardButton>Button 3</button>
      </div>
    </app-card>
  `
})
export class AppComponent {}

// Advanced: Tabs component
@Component({
  selector: 'app-tabs',
  template: `
    <div class="tab-headers">
      <button 
        *ngFor="let tab of tabs; let i = index"
        (click)="selectTab(i)"
        [class.active]="i === activeIndex">
        {{ tab.title }}
      </button>
    </div>
    <div class="tab-content">
      <ng-content></ng-content>
    </div>
  `
})
export class TabsComponent implements AfterContentInit {
  @ContentChildren(TabComponent) tabs: QueryList<TabComponent>;
  activeIndex = 0;
  
  ngAfterContentInit() {
    // Set first tab as active
    if (this.tabs.length > 0) {
      this.selectTab(0);
    }
    
    // React to dynamic tab changes
    this.tabs.changes.subscribe(() => {
      console.log('Tabs changed');
    });
  }
  
  selectTab(index: number) {
    this.tabs.forEach((tab, i) => {
      tab.active = i === index;
    });
    this.activeIndex = index;
  }
}

@Component({
  selector: 'app-tab',
  template: `
    <div [hidden]="!active">
      <ng-content></ng-content>
    </div>
  `
})
export class TabComponent {
  @Input() title: string;
  active = false;
}

// Usage
@Component({
  template: `
    <app-tabs>
      <app-tab title="Tab 1">Content 1</app-tab>
      <app-tab title="Tab 2">Content 2</app-tab>
      <app-tab title="Tab 3">Content 3</app-tab>
    </app-tabs>
  `
})
export class AppComponent {}
```

---

### Q14: How does ViewChild differ from ContentChild?

**Difficulty:** Medium

**Code:**
```typescript
// Parent Component
@Component({
  selector: 'app-parent',
  template: `
    <!-- This is VIEW (defined in this component) -->
    <div #viewDiv>View Content</div>
    <input #viewInput type="text">
    
    <!-- Child component -->
    <app-child>
      <!-- This is CONTENT (projected to child) -->
      <div #contentDiv>Content Div</div>
      <button #contentButton>Content Button</button>
    </app-child>
  `
})
export class ParentComponent implements AfterViewInit {
  @ViewChild('viewDiv') viewDiv: ElementRef;
  @ViewChild('viewInput') viewInput: ElementRef;
  
  ngAfterViewInit() {
    console.log('View div:', this.viewDiv); // ✅ Available
    console.log('View input:', this.viewInput); // ✅ Available
  }
}

// Child Component
@Component({
  selector: 'app-child',
  template: `
    <div class="child-wrapper">
      <!-- VIEW: Defined in child's template -->
      <h3 #childTitle>Child Title</h3>
      
      <!-- CONTENT: Projected from parent -->
      <ng-content></ng-content>
      
      <!-- VIEW: Also in child's template -->
      <button #childButton>Child Button</button>
    </div>
  `
})
export class ChildComponent implements AfterContentInit, AfterViewInit {
  // CONTENT queries - projected from parent
  @ContentChild('contentDiv') contentDiv: ElementRef;
  @ContentChild('contentButton') contentButton: ElementRef;
  
  // VIEW queries - defined in this template
  @ViewChild('childTitle') childTitle: ElementRef;
  @ViewChild('childButton') childButton: ElementRef;
  
  ngAfterContentInit() {
    console.log('Content div:', this.contentDiv); // ✅ Available
    console.log('Content button:', this.contentButton); // ✅ Available
    
    console.log('Child title:', this.childTitle); // ❌ undefined yet
    console.log('Child button:', this.childButton); // ❌ undefined yet
  }
  
  ngAfterViewInit() {
    console.log('Content div:', this.contentDiv); // ✅ Still available
    console.log('Content button:', this.contentButton); // ✅ Still available
    
    console.log('Child title:', this.childTitle); // ✅ Now available
    console.log('Child button:', this.childButton); // ✅ Now available
  }
}
```

**Key Differences:**

| Aspect | ViewChild | ContentChild |
|--------|-----------|--------------|
| **Where defined** | Component's template | Projected from parent |
| **Query location** | Own template | `<ng-content>` |
| **Available in** | AfterViewInit | AfterContentInit |
| **Use case** | Internal elements | Projected content |

---

### Q15: How to communicate between sibling components?

**Difficulty:** Medium

**Code:**
```typescript
// Method 1: Through parent component
@Component({
  selector: 'app-sibling1',
  template: '<button (click)="sendMessage()">Send to Sibling 2</button>'
})
export class Sibling1Component {
  @Output() messageSent = new EventEmitter<string>();
  
  sendMessage() {
    this.messageSent.emit('Hello from Sibling 1');
  }
}

@Component({
  selector: 'app-sibling2',
  template: '<p>Message: {{ message }}</p>'
})
export class Sibling2Component {
  @Input() message: string;
}

@Component({
  selector: 'app-parent',
  template: `
    <app-sibling1 (messageSent)="handleMessage($event)"></app-sibling1>
    <app-sibling2 [message]="sharedMessage"></app-sibling2>
  `
})
export class ParentComponent {
  sharedMessage: string = '';
  
  handleMessage(msg: string) {
    this.sharedMessage = msg;
  }
}

// Method 2: Shared Service (Better for complex scenarios)
@Injectable({ providedIn: 'root' })
export class MessageService {
  private messageSubject = new BehaviorSubject<string>('');
  message$ = this.messageSubject.asObservable();
  
  sendMessage(msg: string) {
    this.messageSubject.next(msg);
  }
}

@Component({
  selector: 'app-sibling1',
  template: '<button (click)="send()">Send Message</button>'
})
export class Sibling1WithServiceComponent {
  constructor(private messageService: MessageService) {}
  
  send() {
    this.messageService.sendMessage('Hello from Sibling 1');
  }
}

@Component({
  selector: 'app-sibling2',
  template: '<p>Message: {{ message$ | async }}</p>'
})
export class Sibling2WithServiceComponent {
  message$ = this.messageService.message$;
  
  constructor(private messageService: MessageService) {}
}

// Method 3: Using RxJS Subject with complex data
@Injectable({ providedIn: 'root' })
export class DataSharingService {
  private dataSubject = new Subject<any>();
  data$ = this.dataSubject.asObservable();
  
  private selectedItemSubject = new BehaviorSubject<any>(null);
  selectedItem$ = this.selectedItemSubject.asObservable();
  
  shareData(data: any) {
    this.dataSubject.next(data);
  }
  
  selectItem(item: any) {
    this.selectedItemSubject.next(item);
  }
}

@Component({
  selector: 'app-item-list',
  template: `
    <div *ngFor="let item of items" (click)="select(item)">
      {{ item.name }}
    </div>
  `
})
export class ItemListComponent {
  items = [
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' }
  ];
  
  constructor(private dataService: DataSharingService) {}
  
  select(item: any) {
    this.dataService.selectItem(item);
  }
}

@Component({
  selector: 'app-item-detail',
  template: `
    <div *ngIf="selectedItem$ | async as item">
      <h2>{{ item.name }}</h2>
      <p>ID: {{ item.id }}</p>
    </div>
  `
})
export class ItemDetailComponent {
  selectedItem$ = this.dataService.selectedItem$;
  
  constructor(private dataService: DataSharingService) {}
}
```

**Sibling Communication Patterns:**
1. **Through Parent**: Simple, explicit
2. **Shared Service**: Decoupled, scalable
3. **State Management**: NgRx, Akita (enterprise)

---

## Input/Output Deep Dive

### Q16: What are Input setters and getters?

**Difficulty:** Medium

**Code:**
```typescript
// Basic @Input
@Component({
  selector: 'app-basic',
  template: '<p>{{ name }}</p>'
})
export class BasicComponent {
  @Input() name: string; // Simple property
}

// @Input with setter/getter
@Component({
  selector: 'app-advanced',
  template: '<p>{{ displayName }}</p>'
})
export class AdvancedComponent {
  private _name: string = '';
  displayName: string = '';
  
  @Input()
  set name(value: string) {
    console.log('Name setter called with:', value);
    
    // Validate input
    if (value && value.trim().length > 0) {
      this._name = value;
      // Transform input
      this.displayName = value.toUpperCase();
    } else {
      this._name = '';
      this.displayName = 'UNKNOWN';
    }
  }
  
  get name(): string {
    return this._name;
  }
}

// Real-world example: User component with validation
@Component({
  selector: 'app-user',
  template: `
    <div class="user-card" [class.active]="isActive">
      <img [src]="avatarUrl" [alt]="userName">
      <h3>{{ userName }}</h3>
      <p>{{ userEmail }}</p>
      <span class="status">{{ statusText }}</span>
    </div>
  `
})
export class UserComponent implements OnChanges {
  private _user: User;
  userName: string = '';
  userEmail: string = '';
  avatarUrl: string = '';
  statusText: string = '';
  isActive: boolean = false;
  
  @Input()
  set user(value: User) {
    console.log('User input changed:', value);
    this._user = value;
    
    if (value) {
      // Transform and validate
      this.userName = value.name || 'Unknown User';
      this.userEmail = value.email || 'no-email@example.com';
      this.avatarUrl = value.avatar || '/assets/default-avatar.png';
      this.isActive = value.status === 'active';
      this.statusText = value.status ? value.status.toUpperCase() : 'OFFLINE';
      
      // Trigger side effects
      this.loadUserDetails(value.id);
    }
  }
  
  get user(): User {
    return this._user;
  }
  
  @Input()
  set userId(id: string) {
    if (id) {
      this.loadUserDetails(id);
    }
  }
  
  ngOnChanges(changes: SimpleChanges) {
    // This still fires, but setter fires first
    console.log('ngOnChanges:', changes);
  }
  
  private loadUserDetails(id: string) {
    // API call or other logic
    console.log('Loading details for user:', id);
  }
}

// Computed properties with getters
@Component({
  selector: 'app-product',
  template: `
    <div>
      <p>Price: {{ price }}</p>
      <p>Discount: {{ discount }}%</p>
      <p>Final Price: {{ finalPrice }}</p>
      <p>Savings: {{ savings }}</p>
    </div>
  `
})
export class ProductComponent {
  @Input() price: number = 0;
  @Input() discount: number = 0;
  
  get finalPrice(): number {
    return this.price * (1 - this.discount / 100);
  }
  
  get savings(): number {
    return this.price - this.finalPrice;
  }
}
```

**Benefits of Input Setters:**
- Validate input data
- Transform input values
- Trigger side effects
- Compute derived values
- Add logging/debugging

---

### Q17: How to use @Output with custom events?

**Difficulty:** Easy

**Code:**
```typescript
// Basic @Output
@Component({
  selector: 'app-child',
  template: `
    <button (click)="notify()">Notify Parent</button>
  `
})
export class ChildComponent {
  @Output() notifyEvent = new EventEmitter<void>();
  
  notify() {
    this.notifyEvent.emit();
  }
}

// @Output with data
@Component({
  selector: 'app-user-form',
  template: `
    <input [(ngModel)]="name">
    <input [(ngModel)]="email">
    <button (click)="submit()">Submit</button>
  `
})
export class UserFormComponent {
  name: string = '';
  email: string = '';
  
  @Output() userSubmit = new EventEmitter<{name: string, email: string}>();
  
  submit() {
    this.userSubmit.emit({
      name: this.name,
      email: this.email
    });
  }
}

// Multiple outputs
@Component({
  selector: 'app-file-uploader',
  template: `
    <input type="file" (change)="onFileSelect($event)">
    <button (click)="upload()">Upload</button>
  `
})
export class FileUploaderComponent {
  @Output() fileSelected = new EventEmitter<File>();
  @Output() uploadStarted = new EventEmitter<void>();
  @Output() uploadProgress = new EventEmitter<number>();
  @Output() uploadComplete = new EventEmitter<string>();
  @Output() uploadError = new EventEmitter<Error>();
  
  selectedFile: File;
  
  onFileSelect(event: any) {
    this.selectedFile = event.target.files[0];
    this.fileSelected.emit(this.selectedFile);
  }
  
  upload() {
    if (!this.selectedFile) return;
    
    this.uploadStarted.emit();
    
    // Simulate upload
    let progress = 0;
    const interval = setInterval(() => {
      progress += 10;
      this.uploadProgress.emit(progress);
      
      if (progress >= 100) {
        clearInterval(interval);
        this.uploadComplete.emit('file-url');
      }
    }, 200);
  }
}

// Usage
@Component({
  template: `
    <app-file-uploader
      (fileSelected)="onFileSelected($event)"
      (uploadStarted)="onUploadStarted()"
      (uploadProgress)="onProgress($event)"
      (uploadComplete)="onComplete($event)"
      (uploadError)="onError($event)">
    </app-file-uploader>
  `
})
export class ParentComponent {
  onFileSelected(file: File) {
    console.log('File selected:', file.name);
  }
  
  onUploadStarted() {
    console.log('Upload started');
  }
  
  onProgress(progress: number) {
    console.log('Progress:', progress);
  }
  
  onComplete(url: string) {
    console.log('Upload complete:', url);
  }
  
  onError(error: Error) {
    console.error('Upload error:', error);
  }
}

// Custom event with type safety
interface UserEvent {
  action: 'create' | 'update' | 'delete';
  userId: string;
  timestamp: Date;
}

@Component({
  selector: 'app-user-manager',
  template: `
    <button (click)="createUser()">Create</button>
    <button (click)="updateUser()">Update</button>
    <button (click)="deleteUser()">Delete</button>
  `
})
export class UserManagerComponent {
  @Output() userAction = new EventEmitter<UserEvent>();
  
  createUser() {
    this.userAction.emit({
      action: 'create',
      userId: 'new-user-id',
      timestamp: new Date()
    });
  }
  
  updateUser() {
    this.userAction.emit({
      action: 'update',
      userId: 'user-123',
      timestamp: new Date()
    });
  }
  
  deleteUser() {
    this.userAction.emit({
      action: 'delete',
      userId: 'user-456',
      timestamp: new Date()
    });
  }
}
```

**@Output Best Practices:**
- Name events with clear verbs (e.g., `userSubmit`, `itemDeleted`)
- Use type-safe event data
- Emit immutable data
- Don't subscribe in the emitting component

---

### Q18: What are Input/Output aliases?

**Difficulty:** Easy

**Code:**
```typescript
// Without alias
@Component({
  selector: 'app-user',
  template: '<p>{{ userName }}</p>'
})
export class UserComponent {
  @Input() userName: string;
  @Output() userNameChange = new EventEmitter<string>();
}
// Usage: <app-user [userName]="name"></app-user>

// With alias
@Component({
  selector: 'app-user',
  template: '<p>{{ userName }}</p>'
})
export class UserComponentAliased {
  @Input('name') userName: string;
  @Output('nameChange') userNameChange = new EventEmitter<string>();
}
// Usage: <app-user [name]="name" (nameChange)="name=$event"></app-user>

// Real-world example: Better API
@Component({
  selector: 'app-button',
  template: `
    <button 
      [disabled]="isDisabled"
      [class.loading]="isLoading"
      (click)="handleClick()">
      {{ buttonText }}
    </button>
  `
})
export class ButtonComponent {
  @Input('text') buttonText: string = 'Click me';
  @Input('disabled') isDisabled: boolean = false;
  @Input('loading') isLoading: boolean = false;
  @Output('clicked') clickEvent = new EventEmitter<void>();
  
  handleClick() {
    if (!this.isDisabled && !this.isLoading) {
      this.clickEvent.emit();
    }
  }
}
// Usage: <app-button text="Submit" [loading]="saving" (clicked)="save()"></app-button>

// Backward compatibility with alias
@Component({
  selector: 'app-data-table',
  template: `<table>...</table>`
})
export class DataTableComponent {
  // Support both old and new property names
  @Input('data') // New name
  set tableData(value: any[]) {
    this._data = value;
  }
  
  @Input('items') // Old name (deprecated)
  set legacyItems(value: any[]) {
    console.warn('items is deprecated, use data instead');
    this._data = value;
  }
  
  private _data: any[] = [];
}
```

**When to use aliases:**
- Better public API
- Shorter property names
- Backward compatibility
- Avoid naming conflicts

---

### Q19: How to handle Input changes efficiently?

**Difficulty:** Hard

**Code:**
```typescript
// ❌ BAD: Inefficient change detection
@Component({
  template: `
    <div>
      <!-- Getter called multiple times per CD -->
      <p>Filtered: {{ getFilteredItems().length }}</p>
    </div>
  `
})
export class BadComponent {
  @Input() items: any[];
  @Input() filter: string;
  
  // Called 5-10+ times per change detection!
  getFilteredItems() {
    console.log('Filtering...'); // You'll see this A LOT
    return this.items.filter(item => 
      item.name.includes(this.filter)
    );
  }
}

// ✅ GOOD: Use ngOnChanges
@Component({
  template: `
    <div>
      <p>Filtered: {{ filteredItems.length }}</p>
    </div>
  `
})
export class GoodComponent implements OnChanges {
  @Input() items: any[];
  @Input() filter: string;
  
  filteredItems: any[] = [];
  
  ngOnChanges(changes: SimpleChanges) {
    // Only recalculate when inputs actually change
    if (changes['items'] || changes['filter']) {
      console.log('Filtering...'); // Called only when needed
      this.filteredItems = this.items.filter(item =>
        item.name.includes(this.filter)
      );
    }
  }
}

// ✅ BETTER: Use OnPush + computed properties
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div>
      <p>Filtered: {{ filteredItems.length }}</p>
      <div *ngFor="let item of filteredItems">
        {{ item.name }}
      </div>
    </div>
  `
})
export class BetterComponent implements OnChanges {
  @Input() items: any[];
  @Input() filter: string;
  
  filteredItems: any[] = [];
  
  ngOnChanges(changes: SimpleChanges) {
    if (changes['items'] || changes['filter']) {
      this.filterItems();
    }
  }
  
  private filterItems() {
    if (!this.items) {
      this.filteredItems = [];
      return;
    }
    
    if (!this.filter) {
      this.filteredItems = [...this.items];
      return;
    }
    
    this.filteredItems = this.items.filter(item =>
      item.name.toLowerCase().includes(this.filter.toLowerCase())
    );
  }
}

// ✅ BEST: Reactive approach with RxJS
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div>
      <p>Filtered: {{ (filteredItems$ | async)?.length }}</p>
      <div *ngFor="let item of filteredItems$ | async">
        {{ item.name }}
      </div>
    </div>
  `
})
export class BestComponent implements OnInit, OnDestroy {
  @Input() set items(value: any[]) {
    this.itemsSubject.next(value);
  }
  
  @Input() set filter(value: string) {
    this.filterSubject.next(value);
  }
  
  private itemsSubject = new BehaviorSubject<any[]>([]);
  private filterSubject = new BehaviorSubject<string>('');
  private destroy$ = new Subject<void>();
  
  filteredItems$ = combineLatest([
    this.itemsSubject,
    this.filterSubject
  ]).pipe(
    map(([items, filter]) => {
      if (!filter) return items;
      return items.filter(item =>
        item.name.toLowerCase().includes(filter.toLowerCase())
      );
    }),
    shareReplay(1),
    takeUntil(this.destroy$)
  );
  
  ngOnInit() {}
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**Performance Comparison:**

| Approach | CD Calls | Performance |
|----------|----------|-------------|
| Getter | 50+ per interaction | ❌ Poor |
| ngOnChanges | 1 per input change | ✅ Good |
| OnPush + ngOnChanges | 1 per input change | ✅ Better |
| RxJS Observables | 1 per actual change | ✅ Best |

---

### Q20: What are optional and required inputs (Angular 16+)?

**Difficulty:** Easy

**Code:**
```typescript
// Before Angular 16
@Component({
  selector: 'app-user-card'
})
export class UserCardOldComponent {
  @Input() name: string; // Optional by default
  @Input() email!: string; // Required (TypeScript only)
  // No compile-time check for required inputs
}

// Angular 16+: Required inputs
@Component({
  selector: 'app-user-card',
  template: `
    <div>
      <h3>{{ name }}</h3>
      <p>{{ email }}</p>
      <p *ngIf="age">Age: {{ age }}</p>
    </div>
  `
})
export class UserCardComponent {
  // Required inputs - compile error if not provided
  name = input.required<string>();
  email = input.required<string>();
  
  // Optional inputs
  age = input<number>(); // undefined by default
  avatar = input<string>('/default-avatar.png'); // with default value
  
  // Use in template or methods
  ngOnInit() {
    console.log(this.name()); // Call as function
    console.log(this.email());
  }
}

// Usage
@Component({
  template: `
    <!-- ✅ Valid - all required inputs provided -->
    <app-user-card
      [name]="'John'"
      [email]="'john@example.com'"
      [age]="25">
    </app-user-card>
    
    <!-- ❌ Compile error - missing required inputs -->
    <app-user-card></app-user-card>
  `
})
export class ParentComponent {}

// Transform inputs (Angular 16+)
@Component({
  selector: 'app-product',
  template: `
    <div>
      <p>Price: {{ price() }}</p>
      <p>Is Available: {{ available() }}</p>
    </div>
  `
})
export class ProductComponent {
  // Transform string to number
  price = input.required<number, string>({
    transform: (value: string) => parseFloat(value)
  });
  
  // Transform string to boolean
  available = input<boolean, string | boolean>(false, {
    transform: (value: string | boolean) => {
      if (typeof value === 'boolean') return value;
      return value === 'true' || value === '1' || value === '';
    }
  });
}

// Usage
@Component({
  template: `
    <app-product
      price="99.99"
      available="true">
    </app-product>
  `
})
export class ShopComponent {}

// Combining with computed signals
@Component({
  selector: 'app-cart',
  template: `
    <div>
      <p>Subtotal: {{ subtotal() }}</p>
      <p>Tax: {{ tax() }}</p>
      <p>Total: {{ total() }}</p>
    </div>
  `
})
export class CartComponent {
  items = input.required<CartItem[]>();
  taxRate = input<number>(0.08); // 8% default
  
  // Computed values
  subtotal = computed(() => {
    return this.items().reduce((sum, item) => 
      sum + (item.price * item.quantity), 0
    );
  });
  
  tax = computed(() => {
    return this.subtotal() * this.taxRate();
  });
  
  total = computed(() => {
    return this.subtotal() + this.tax();
  });
}
```

**Angular 16+ Input Features:**
- ✅ **input.required()**: Compile-time required checks
- ✅ **input()**: Optional with default values
- ✅ **transform**: Input transformation
- ✅ **computed()**: Derived values
- ✅ Better type safety

---

## ViewChild & ContentChild

### Q21: What's the difference between static and dynamic queries?

**Difficulty:** Medium

**Code:**
```typescript
// static: true - Element always present in template
@Component({
  template: `
    <!-- Always present -->
    <div #staticElement>Always here</div>
  `
})
export class StaticQueryComponent {
  // Available in ngOnInit
  @ViewChild('staticElement', { static: true }) element: ElementRef;
  
  constructor() {
    console.log(this.element); // undefined
  }
  
  ngOnInit() {
    console.log(this.element); // ✅ Available!
    this.element.nativeElement.style.color = 'red';
  }
}

// static: false - Element might not be present
@Component({
  template: `
    <!-- Conditional element -->
    <div *ngIf="showElement" #dynamicElement>
      Sometimes here
    </div>
    <button (click)="toggle()">Toggle</button>
  `
})
export class DynamicQueryComponent {
  showElement = false;
  
  // Not available in ngOnInit
  @ViewChild('dynamicElement', { static: false }) element: ElementRef;
  
  ngOnInit() {
    console.log(this.element); // undefined (showElement is false)
  }
  
  ngAfterViewInit() {
    console.log(this.element); // Still undefined (showElement is false)
  }
  
  toggle() {
    this.showElement = !this.showElement;
    
    // Need to wait for next tick
    setTimeout(() => {
      if (this.element) {
        console.log('Element now available:', this.element);
      }
    });
  }
}

// Real-world example: Form validation
@Component({
  template: `
    <form #userForm="ngForm">
      <input name="email" #emailInput="ngModel" [(ngModel)]="email" required email>
      
      <!-- Conditional error message -->
      <div *ngIf="emailInput.invalid && emailInput.touched" #errorMessage>
        Invalid email
      </div>
    </form>
    
    <button (click)="submit()">Submit</button>
  `
})
export class FormComponent implements AfterViewInit {
  @ViewChild('userForm', { static: true }) form: NgForm; // Always present
  @ViewChild('errorMessage', { static: false }) errorMsg: ElementRef; // Conditional
  
  email = '';
  
  ngAfterViewInit() {
    console.log('Form:', this.form); // ✅ Available
    console.log('Error message:', this.errorMsg); // undefined (not shown yet)
  }
  
  submit() {
    if (this.form.invalid) {
      this.form.control.markAllAsTouched();
      
      // Error message now