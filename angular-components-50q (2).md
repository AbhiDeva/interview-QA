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
      
      // Error message now  initTheme() {
    const saved = localStorage.getItem('theme') || 'light';
    this.setTheme(saved);
  }
}

// App component with theming
@Component({
  selector: 'app-root',
  template: `
    <div [class]="'theme-' + (theme$ | async)">
      <header>
        <button (click)="toggleTheme()">Toggle Theme</button>
      </header>
      
      <main>
        <app-card>Content here</app-card>
      </main>
    </div>
  `,
  styles: [`
    /* Light theme (default) */
    .theme-light {
      --bg-primary: #ffffff;
      --bg-secondary: #f5f5f5;
      --text-primary: #333333;
      --text-secondary: #666666;
      --border-color: #dddddd;
      --accent-color: #007bff;
    }
    
    /* Dark theme */
    .theme-dark {
      --bg-primary: #1a1a1a;
      --bg-secondary: #2a2a2a;
      --text-primary: #ffffff;
      --text-secondary: #cccccc;
      --border-color: #444444;
      --accent-color: #0096ff;
    }
    
    /* Apply theme variables */
    :host {
      background: var(--bg-primary);
      color: var(--text-primary);
      display: block;
      min-height: 100vh;
    }
    
    header {
      background: var(--bg-secondary);
      border-bottom: 1px solid var(--border-color);
      padding: 16px;
    }
    
    button {
      background: var(--accent-color);
      color: white;
      border: none;
      padding: 8px 16px;
      border-radius: 4px;
      cursor: pointer;
    }
    
    main {
      padding: 20px;
    }
  `]
})
export class AppComponent implements OnInit {
  theme$ = this.themeService.theme$;
  
  constructor(private themeService: ThemeService) {}
  
  ngOnInit() {
    this.themeService.initTheme();
  }
  
  toggleTheme() {
    this.themeService.toggleTheme();
  }
}

// Themed component
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <ng-content></ng-content>
    </div>
  `,
  styles: [`
    .card {
      background: var(--bg-primary);
      border: 1px solid var(--border-color);
      border-radius: 8px;
      padding: 20px;
      color: var(--text-primary);
    }
  `]
})
export class CardComponent {}

// Advanced: Multiple theme support
interface Theme {
  name: string;
  properties: Record<string, string>;
}

const THEMES: Record<string, Theme> = {
  light: {
    name: 'Light',
    properties: {
      '--bg-primary': '#ffffff',
      '--bg-secondary': '#f5f5f5',
      '--text-primary': '#333333',
      '--accent-color': '#007bff'
    }
  },
  dark: {
    name: 'Dark',
    properties: {
      '--bg-primary': '#1a1a1a',
      '--bg-secondary': '#2a2a2a',
      '--text-primary': '#ffffff',
      '--accent-color': '#0096ff'
    }
  },
  ocean: {
    name: 'Ocean',
    properties: {
      '--bg-primary': '#0a192f',
      '--bg-secondary': '#172a45',
      '--text-primary': '#8892b0',
      '--accent-color': '#64ffda'
    }
  }
};

@Injectable({ providedIn: 'root' })
export class AdvancedThemeService {
  private themeSubject = new BehaviorSubject<string>('light');
  theme$ = this.themeSubject.asObservable();
  
  setTheme(themeName: string) {
    const theme = THEMES[themeName];
    if (!theme) return;
    
    // Apply CSS variables to document root
    const root = document.documentElement;
    Object.entries(theme.properties).forEach(([key, value]) => {
      root.style.setProperty(key, value);
    });
    
    this.themeSubject.next(themeName);
    localStorage.setItem('theme', themeName);
  }
  
  getAvailableThemes(): string[] {
    return Object.keys(THEMES);
  }
  
  getThemeDisplayName(themeName: string): string {
    return THEMES[themeName]?.name || themeName;
  }
}
```

---

### Q30: How to style projected content (ng-content)?

**Difficulty:** Medium

**Code:**
```typescript
// Component with content projection
@Component({
  selector: 'app-panel',
  template: `
    <div class="panel">
      <div class="panel-header">
        <ng-content select="[header]"></ng-content>
      </div>
      
      <div class="panel-body">
        <ng-content></ng-content>
      </div>
      
      <div class="panel-footer">
        <ng-content select="[footer]"></ng-content>
      </div>
    </div>
  `,
  styles: [`
    .panel {
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    
    .panel-header {
      background: #f5f5f5;
      padding: 12px;
      border-bottom: 1px solid #ddd;
    }
    
    .panel-body {
      padding: 16px;
    }
    
    .panel-footer {
      background: #f5f5f5;
      padding: 12px;
      border-top: 1px solid #ddd;
    }
    
    /* ❌ This won't style projected content (encapsulation) */
    h2 {
      color: blue; /* Won't work on projected <h2> */
    }
    
    /* ✅ Use ::ng-deep to style projected content */
    :host ::ng-deep h2 {
      color: blue;
      margin: 0;
    }
    
    /* ✅ Or use CSS classes on projected elements */
    :host ::ng-deep .panel-title {
      font-size: 18px;
      font-weight: bold;
    }
  `]
})
export class PanelComponent {}

// Usage
@Component({
  template: `
    <app-panel>
      <div header>
        <h2 class="panel-title">Panel Header</h2>
      </div>
      
      <p>This is the panel body content.</p>
      
      <div footer>
        <button>Action</button>
      </div>
    </app-panel>
  `
})
export class AppComponent {}

// Better approach: Use ViewEncapsulation.None for wrapper
@Component({
  selector: 'app-styled-wrapper',
  encapsulation: ViewEncapsulation.None,
  template: `
    <div class="styled-wrapper">
      <ng-content></ng-content>
    </div>
  `,
  styles: [`
    /* These styles are global but scoped by class */
    .styled-wrapper h1 {
      color: #333;
      font-size: 24px;
    }
    
    .styled-wrapper p {
      line-height: 1.6;
      color: #666;
    }
    
    .styled-wrapper button {
      background: #007bff;
      color: white;
      border: none;
      padding: 8px 16px;
      border-radius: 4px;
    }
  `]
})
export class StyledWrapperComponent {}

// Best approach: Provide CSS classes for consumers
@Component({
  selector: 'app-card-slot',
  template: `
    <div class="card">
      <div class="card-media">
        <ng-content select=".card-image"></ng-content>
      </div>
      
      <div class="card-content">
        <ng-content select=".card-title"></ng-content>
        <ng-content select=".card-description"></ng-content>
      </div>
      
      <div class="card-actions">
        <ng-content select=".card-action"></ng-content>
      </div>
    </div>
  `,
  styles: [`
    .card {
      border: 1px solid #ddd;
      border-radius: 8px;
      overflow: hidden;
    }
    
    .card-media {
      width: 100%;
      height: 200px;
      overflow: hidden;
    }
    
    /* Style the slots */
    :host ::ng-deep .card-image {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }
    
    .card-content {
      padding: 16px;
    }
    
    :host ::ng-deep .card-title {
      font-size: 20px;
      font-weight: bold;
      margin-bottom: 8px;
    }
    
    :host ::ng-deep .card-description {
      color: #666;
      line-height: 1.6;
    }
    
    .card-actions {
      padding: 12px 16px;
      background: #f5f5f5;
      display: flex;
      gap: 8px;
    }
    
    :host ::ng-deep .card-action {
      padding: 8px 16px;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
  `]
})
export class CardSlotComponent {}

// Usage with proper classes
@Component({
  template: `
    <app-card-slot>
      <img class="card-image" src="image.jpg" alt="Card">
      <h2 class="card-title">Card Title</h2>
      <p class="card-description">This is the card description.</p>
      <button class="card-action">Read More</button>
      <button class="card-action">Share</button>
    </app-card-slot>
  `
})
export class AppComponent2 {}
```

---

## Dynamic Components

### Q31: How to create components dynamically?

**Difficulty:** Hard

**Code:**
```typescript
// Component to create dynamically
@Component({
  selector: 'app-alert',
  template: `
    <div class="alert" [class]="'alert-' + type">
      <strong>{{ title }}</strong>
      <p>{{ message }}</p>
      <button (click)="close()">Close</button>
    </div>
  `,
  styles: [`
    .alert {
      padding: 16px;
      border-radius: 4px;
      margin: 8px 0;
    }
    
    .alert-success { background: #d4edda; border: 1px solid #c3e6cb; }
    .alert-error { background: #f8d7da; border: 1px solid #f5c6cb; }
    .alert-warning { background: #fff3cd; border: 1px solid #ffeaa7; }
  `]
})
export class AlertComponent {
  @Input() title: string;
  @Input() message: string;
  @Input() type: 'success' | 'error' | 'warning' = 'success';
  @Output() closed = new EventEmitter<void>();
  
  close() {
    this.closed.emit();
  }
}

// Service to create dynamic components
@Injectable({ providedIn: 'root' })
export class DynamicComponentService {
  constructor(
    private appRef: ApplicationRef,
    private injector: Injector
  ) {}
  
  // Method 1: Create and append to body
  createComponent<T>(
    component: Type<T>,
    inputs?: Partial<T>
  ): ComponentRef<T> {
    // Create component
    const componentRef = createComponent(component, {
      environmentInjector: this.appRef.injector
    });
    
    // Set inputs
    if (inputs) {
      Object.assign(componentRef.instance, inputs);
    }
    
    // Attach to app
    this.appRef.attachView(componentRef.hostView);
    
    // Get DOM element
    const domElem = (componentRef.hostView as EmbeddedViewRef<any>)
      .rootNodes[0] as HTMLElement;
    
    // Append to body
    document.body.appendChild(domElem);
    
    return componentRef;
  }
  
  // Method 2: Create in ViewContainerRef
  createInContainer<T>(
    container: ViewContainerRef,
    component: Type<T>,
    inputs?: Partial<T>
  ): ComponentRef<T> {
    const componentRef = container.createComponent(component);
    
    if (inputs) {
      Object.assign(componentRef.instance, inputs);
    }
    
    return componentRef;
  }
  
  // Destroy component
  destroyComponent(componentRef: ComponentRef<any>) {
    this.appRef.detachView(componentRef.hostView);
    componentRef.destroy();
  }
}

// Alert service using dynamic components
@Injectable({ providedIn: 'root' })
export class AlertService {
  private alerts: ComponentRef<AlertComponent>[] = [];
  
  constructor(private dynamicService: DynamicComponentService) {}
  
  showSuccess(title: string, message: string, duration = 3000) {
    return this.showAlert('success', title, message, duration);
  }
  
  showError(title: string, message: string, duration = 5000) {
    return this.showAlert('error', title, message, duration);
  }
  
  showWarning(title: string, message: string, duration = 4000) {
    return this.showAlert('warning', title, message, duration);
  }
  
  private showAlert(
    type: 'success' | 'error' | 'warning',
    title: string,
    message: string,
    duration: number
  ) {
    // Create alert component
    const alertRef = this.dynamicService.createComponent(AlertComponent, {
      title,
      message,
      type
    });
    
    // Subscribe to close event
    alertRef.instance.closed.subscribe(() => {
      this.closeAlert(alertRef);
    });
    
    // Store reference
    this.alerts.push(alertRef);
    
    // Auto-close after duration
    if (duration > 0) {
      setTimeout(() => {
        this.closeAlert(alertRef);
      }, duration);
    }
    
    return alertRef;
  }
  
  private closeAlert(alertRef: ComponentRef<AlertComponent>) {
    const index = this.alerts.indexOf(alertRef);
    if (index > -1) {
      this.alerts.splice(index, 1);
    }
    this.dynamicService.destroyComponent(alertRef);
  }
  
  closeAll() {
    this.alerts.forEach(alert => {
      this.dynamicService.destroyComponent(alert);
    });
    this.alerts = [];
  }
}

// Usage in component
@Component({
  selector: 'app-demo',
  template: `
    <button (click)="showSuccess()">Show Success</button>
    <button (click)="showError()">Show Error</button>
    <button (click)="showWarning()">Show Warning</button>
  `
})
export class DemoComponent {
  constructor(private alertService: AlertService) {}
  
  showSuccess() {
    this.alertService.showSuccess(
      'Success!',
      'Operation completed successfully'
    );
  }
  
  showError() {
    this.alertService.showError(
      'Error!',
      'Something went wrong'
    );
  }
  
  showWarning() {
    this.alertService.showWarning(
      'Warning!',
      'Please review your input'
    );
  }
}

// Advanced: Modal service
@Injectable({ providedIn: 'root' })
export class ModalService {
  private modalRef: ComponentRef<ModalComponent> | null = null;
  
  constructor(private dynamicService: DynamicComponentService) {}
  
  open<T>(
    contentComponent: Type<T>,
    config?: { title?: string; width?: string; data?: any }
  ): ComponentRef<ModalComponent> {
    // Close existing modal
    this.close();
    
    // Create modal wrapper
    this.modalRef = this.dynamicService.createComponent(ModalComponent, {
      title: config?.title,
      width: config?.width
    });
    
    // Create content component inside modal
    const contentRef = this.dynamicService.createInContainer(
      this.modalRef.instance.contentContainer,
      contentComponent,
      config?.data
    );
    
    // Subscribe to close
    this.modalRef.instance.closeEvent.subscribe(() => {
      this.close();
    });
    
    return this.modalRef;
  }
  
  close() {
    if (this.modalRef) {
      this.dynamicService.destroyComponent(this.modalRef);
      this.modalRef = null;
    }
  }
}

@Component({
  selector: 'app-modal',
  template: `
    <div class="modal-backdrop" (click)="close()">
      <div class="modal-content" (click)="$event.stopPropagation()" [style.width]="width">
        <div class="modal-header">
          <h2>{{ title }}</h2>
          <button (click)="close()">×</button>
        </div>
        <div class="modal-body">
          <ng-container #contentContainer></ng-container>
        </div>
      </div>
    </div>
  `,
  styles: [`
    .modal-backdrop {
      position: fixed;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background: rgba(0,0,0,0.5);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 1000;
    }
    
    .modal-content {
      background: white;
      border-radius: 8px;
      max-height: 90vh;
      overflow: auto;
    }
    
    .modal-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 16px;
      border-bottom: 1px solid #ddd;
    }
    
    .modal-body {
      padding: 16px;
    }
  `]
})
export class ModalComponent {
  @ViewChild('contentContainer', { read: ViewContainerRef }) 
  contentContainer: ViewContainerRef;
  
  @Input() title: string = 'Modal';
  @Input() width: string = '600px';
  @Output() closeEvent = new EventEmitter<void>();
  
  close() {
    this.closeEvent.emit();
  }
}
```

---

### Q32: How to pass data to dynamically created components?

**Difficulty:** Medium

**Code:**
```typescript
// Component with inputs and outputs
@Component({
  selector: 'app-dynamic-form',
  template: `
    <form (ngSubmit)="submit()">
      <h2>{{ title }}</h2>
      
      <input [(ngModel)]="formData.name" name="name" placeholder="Name">
      <input [(ngModel)]="formData.email" name="email" placeholder="Email">
      
      <button type="submit">Submit</button>
      <button type="button" (click)="cancel()">Cancel</button>
    </form>
  `
})
export class DynamicFormComponent {
  @Input() title: string = 'Form';
  @Input() initialData: any = {};
  @Output() submitted = new EventEmitter<any>();
  @Output() cancelled = new EventEmitter<void>();
  
  formData: any = {};
  
  ngOnInit() {
    this.formData = { ...this.initialData };
  }
  
  submit() {
    this.submitted.emit(this.formData);
  }
  
  cancel() {
    this.cancelled.emit();
  }
}

// Service to create with data
@Injectable({ providedIn: 'root' })
export class FormService {
  constructor(private dynamicService: DynamicComponentService) {}
  
  openForm(title: string, initialData?: any): Promise<any> {
    return new Promise((resolve, reject) => {
      // Create component
      const formRef = this.dynamicService.createComponent(
        DynamicFormComponent,
        {
          title,
          initialData: initialData || {}
        }
      );
      
      // Handle submission
      formRef.instance.submitted.subscribe((data) => {
        resolve(data);
        this.dynamicService.destroyComponent(formRef);
      });
      
      // Handle cancellation
      formRef.instance.cancelled.subscribe(() => {
        reject(new Error('Form cancelled'));
        this.dynamicService.destroyComponent(formRef);
      });
    });
  }
}

// Usage
@Component({})
export class AppComponent {
  constructor(private formService: FormService) {}
  
  async openUserForm() {
    try {
      const result = await this.formService.openForm('New User', {
        name: 'John Doe',
        email: 'john@example.com'
      });
      
      console.log('Form submitted:', result);
    } catch (error) {
      console.log('Form cancelled');
    }
  }
}

// Advanced: Injecting services into dynamic components
@Injectable()
export class DynamicComponentDataService {
  private dataSubject = new BehaviorSubject<any>(null);
  data$ = this.dataSubject.asObservable();
  
  setData(data: any) {
    this.dataSubject.next(data);
  }
}

@Component({
  selector: 'app-data-consumer',
  template: `
    <div *ngIf="data$ | async as data">
      <p>{{ data.message }}</p>
    </div>
  `
})
export class DataConsumerComponent {
  data$ = this.dataService.data$;
  
  constructor(private dataService: DynamicComponentDataService) {}
}

// Create with custom injector
@Injectable({ providedIn: 'root' })
export class AdvancedDynamicService {
  constructor(
    private appRef: ApplicationRef,
    private injector: Injector
  ) {}
  
  createWithData<T>(
    component: Type<T>,
    data: any
  ): ComponentRef<T> {
    // Create custom injector with data
    const dataService = new DynamicComponentDataService();
    dataService.setData(data);
    
    const customInjector = Injector.create({
      providers: [
        { provide: DynamicComponentDataService, useValue: dataService }
      ],
      parent: this.injector
    });
    
    // Create component with custom injector
    const componentRef = createComponent(component, {
      environmentInjector: this.appRef.injector,
      elementInjector: customInjector
    });
    
    this.appRef.attachView(componentRef.hostView);
    
    const domElem = (componentRef.hostView as EmbeddedViewRef<any>)
      .rootNodes[0] as HTMLElement;
    document.body.appendChild(domElem);
    
    return componentRef;
  }
}
```

---

### Q33: How to lazy load components dynamically?

**Difficulty:** Hard

**Code:**
```typescript
// Lazy-loaded component
@Component({
  selector: 'app-heavy-chart',
  template: `
    <div class="chart">
      <canvas #chart></canvas>
      <p>Chart with {{ dataPoints }} data points</p>
    </div>
  `
})
export class HeavyChartComponent {
  @Input() dataPoints: number = 1000;
}

// Feature module for lazy loading
@NgModule({
  declarations: [HeavyChartComponent],
  imports: [CommonModule]
})
export class ChartsModule {}

// Service for lazy loading
@Injectable({ providedIn: 'root' })
export class LazyLoaderService {
  private loadedModules = new Map<string, any>();
  
  constructor(
    private compiler: Compiler,
    private injector: Injector,
    private appRef: ApplicationRef
  ) {}
  
  async loadComponent(
    modulePath: () => Promise<any>,
    componentName: string
  ): Promise<Type<any>> {
    // Check cache
    const cacheKey = componentName;
    if (this.loadedModules.has(cacheKey)) {
      return this.loadedModules.get(cacheKey);
    }
    
    // Load module
    const module = await modulePath();
    const moduleFactory = await this.compiler.compileModuleAsync(module.ChartsModule);
    const moduleRef = moduleFactory.create(this.injector);
    
    // Get component
    const component = moduleRef.instance[componentName];
    
    // Cache
    this.loadedModules.set(cacheKey, component);
    
    return component;
  }
  
  // Angular 13+ with createComponent
  async loadAndCreate<T>(
    modulePath: () => Promise<any>,
    componentName: string,
    container: ViewContainerRef,
    inputs?: Partial<T>
  ): Promise<ComponentRef<T>> {
    const component = await this.loadComponent(modulePath, componentName);
    
    const componentRef = container.createComponent(component);
    
    if (inputs) {
      Object.assign(componentRef.instance, inputs);
    }
    
    return componentRef;
  }
}

// Usage component
@Component({
  selector: 'app-dashboard',
  template: `
    <button (click)="loadChart()" [disabled]="chartLoaded">
      Load Chart
    </button>
    
    <div class="chart-container" #chartContainer></div>
  `
})
export class DashboardComponent {
  @ViewChild('chartContainer', { read: ViewContainerRef }) 
  container: ViewContainerRef;
  
  chartLoaded = false;
  
  constructor(private lazyLoader: LazyLoaderService) {}
  
  async loadChart() {
    if (this.chartLoaded) return;
    
    try {
      await this.lazyLoader.loadAndCreate(
        () => import('./charts/charts.module').then(m => m.ChartsModule),
        'HeavyChartComponent',
        this.container,
        { dataPoints: 5000 }
      );
      
      this.chartLoaded = true;
    } catch (error) {
      console.error('Failed to load chart:', error);
    }
  }
}

// Modern approach: Standalone components (Angular 14+)
// No module needed!
@Component({
  selector: 'app-standalone-chart',
  standalone: true,
  imports: [CommonModule],
  template: `<div>Standalone Chart</div>`
})
export class StandaloneChartComponent {
  @Input() data: number[];
}

// Lazy load standalone component
@Component({
  selector: 'app-modern-dashboard',
  template: `
    <button (click)="loadChart()">Load Chart</button>
    <ng-container #chartContainer></ng-container>
  `
})
export class ModernDashboardComponent {
  @ViewChild('chartContainer', { read: ViewContainerRef })
  container: ViewContainerRef;
  
  async loadChart() {
    // Dynamically import standalone component
    const { StandaloneChartComponent } = await import('./standalone-chart.component');
    
    // Create it directly
    const componentRef = this.container.createComponent(StandaloneChartComponent);
    componentRef.setInput('data', [1, 2, 3, 4, 5]);
  }
}
```

---

## Component Architecture

### Q34: What are Smart and Presentational components?

**Difficulty:** Medium

**Code:**
```typescript
// ❌ BAD: Mixed concerns
@Component({
  selector: 'app-user-list-bad',
  template: `
    <div *ngFor="let user of users">
      <div class="user-card">
        <h3>{{ user.name }}</h3>
        <p>{{ user.email }}</p>
        <button (click)="deleteUser(user.id)">Delete</button>
      </div>
    </div>
  `
})
export class UserListBadComponent implements OnInit {
  users: User[] = [];
  
  constructor(
    private userService: UserService,
    private router: Router
  ) {}
  
  ngOnInit() {
    this.loadUsers();
  }
  
  loadUsers() {
    this.userService.getUsers().subscribe(users => {
      this.users = users;
    });
  }
  
  deleteUser(id: string) {
    this.userService.deleteUser(id).subscribe(() => {
      this.loadUsers();
    });
  }
}

// ✅ GOOD: Separated concerns

// 1. Presentational Component (Dumb/Pure)
@Component({
  selector: 'app-user-card',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="user-card">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
      <button (click)="delete.emit(user.id)">Delete</button>
      <button (click)="edit.emit(user)">Edit</button>
    </div>
  `,
  styles: [`
    .user-card {
      border: 1px solid #ddd;
      padding: 16px;
      margin: 8px 0;
      border-radius: 4px;
    }
  `]
})
export class UserCardComponent {
  @Input() user: User;
  @Output() delete = new EventEmitter<string>();
  @Output() edit = new EventEmitter<User>();
}

// 2. Smart Component (Container)
@Component({
  selector: 'app-user-list-container',
  template: `
    <div class="user-list">
      <h2>Users</h2>
      
      <button (click)="refresh()">Refresh</button>
      
      <app-user-card
        *ngFor="let user of users$ | async"
        [user]="user"
        (delete)="handleDelete($event)"
        (edit)="handleEdit($event)">
      </app-user-card>
      
      <div *ngIf="loading$ | async">Loading...</div>
      <div *ngIf="error$ | async as error">Error: {{ error }}</div>
    </div>
  `
})
export class UserListContainerComponent implements OnInit {
  users$ = this.userService.users$;
  loading$ = this.userService.loading$;
  error$ = this.userService.error$;
  
  constructor(
    private userService: UserService,
    private router: Router
  ) {}
  
  ngOnInit() {
    this.userService.loadUsers();
  }
  
  refresh() {
    this.userService.loadUsers();  submit() {
    if (this.form.invalid) {
      this.form.control.markAllAsTouched();
      
      // Error message now visible after CD
      setTimeout(() => {
        if (this.errorMsg) {
          this.errorMsg.nativeElement.scrollIntoView();
        }
      });
    }
  }
}
```

**When to use static:**
- **static: true**: Element always in DOM (no *ngIf, *ngFor)
- **static: false**: Element might be conditional (default)

---

### Q22: How to query multiple elements with ViewChildren?

**Difficulty:** Medium

**Code:**
```typescript
@Component({
  selector: 'app-list',
  template: `
    <div class="items">
      <div #item *ngFor="let data of items" class="item">
        {{ data }}
      </div>
    </div>
    
    <button (click)="highlightAll()">Highlight All</button>
    <button (click)="scrollToLast()">Scroll to Last</button>
  `
})
export class ListComponent implements AfterViewInit {
  items = ['Item 1', 'Item 2', 'Item 3', 'Item 4'];
  
  @ViewChildren('item') itemElements: QueryList<ElementRef>;
  
  ngAfterViewInit() {
    console.log('Number of items:', this.itemElements.length); // 4
    
    // Iterate through elements
    this.itemElements.forEach((element, index) => {
      console.log(`Item ${index}:`, element.nativeElement.textContent);
    });
    
    // Listen to changes
    this.itemElements.changes.subscribe(items => {
      console.log('Items changed:', items.length);
    });
    
    // Access specific items
    const firstItem = this.itemElements.first;
    const lastItem = this.itemElements.last;
    const itemsArray = this.itemElements.toArray();
  }
  
  highlightAll() {
    this.itemElements.forEach(element => {
      element.nativeElement.style.backgroundColor = 'yellow';
    });
  }
  
  scrollToLast() {
    const last = this.itemElements.last;
    if (last) {
      last.nativeElement.scrollIntoView({ behavior: 'smooth' });
    }
  }
  
  addItem() {
    this.items.push(`Item ${this.items.length + 1}`);
    // QueryList automatically updates after change detection
  }
}

// Query component instances
@Component({
  selector: 'app-card',
  template: '<div>Card {{ id }}</div>'
})
export class CardComponent {
  @Input() id: number;
  
  highlight() {
    console.log(`Card ${this.id} highlighted`);
  }
}

@Component({
  selector: 'app-dashboard',
  template: `
    <app-card *ngFor="let i of [1,2,3,4,5]" [id]="i"></app-card>
    <button (click)="highlightAllCards()">Highlight All</button>
  `
})
export class DashboardComponent implements AfterViewInit {
  @ViewChildren(CardComponent) cards: QueryList<CardComponent>;
  
  ngAfterViewInit() {
    console.log('Number of cards:', this.cards.length);
  }
  
  highlightAllCards() {
    this.cards.forEach(card => card.highlight());
  }
}

// Query with read option
@Component({
  template: `
    <input #input1 type="text">
    <input #input2 type="text">
    <input #input3 type="text">
  `
})
export class MultiReadComponent implements AfterViewInit {
  // Read as ElementRef (default)
  @ViewChildren('input1,input2,input3') inputs: QueryList<ElementRef>;
  
  // Read as ViewContainerRef
  @ViewChildren('input1,input2', { read: ViewContainerRef }) 
  containers: QueryList<ViewContainerRef>;
  
  ngAfterViewInit() {
    console.log('Inputs:', this.inputs.length);
    console.log('Containers:', this.containers.length);
  }
}
```

---

### Q23: How to use ContentChildren with descendants?

**Difficulty:** Hard

**Code:**
```typescript
// descendants: true (default) - queries all descendants
@Component({
  selector: 'app-accordion',
  template: `
    <div class="accordion">
      <ng-content></ng-content>
    </div>
  `
})
export class AccordionComponent implements AfterContentInit {
  // Queries ALL accordion items, even nested ones
  @ContentChildren(AccordionItemComponent, { descendants: true })
  allItems: QueryList<AccordionItemComponent>;
  
  ngAfterContentInit() {
    console.log('All items (including nested):', this.allItems.length);
  }
}

// descendants: false - queries only direct children
@Component({
  selector: 'app-accordion-strict',
  template: `
    <div class="accordion">
      <ng-content></ng-content>
    </div>
  `
})
export class AccordionStrictComponent implements AfterContentInit {
  // Queries ONLY direct children, not nested
  @ContentChildren(AccordionItemComponent, { descendants: false })
  directItems: QueryList<AccordionItemComponent>;
  
  ngAfterContentInit() {
    console.log('Direct items only:', this.directItems.length);
  }
}

@Component({
  selector: 'app-accordion-item',
  template: `
    <div class="accordion-item">
      <div class="header" (click)="toggle()">
        <ng-content select="[header]"></ng-content>
      </div>
      <div class="content" [hidden]="!isOpen">
        <ng-content select="[content]"></ng-content>
      </div>
    </div>
  `
})
export class AccordionItemComponent {
  @Input() title: string;
  isOpen = false;
  
  toggle() {
    this.isOpen = !this.isOpen;
  }
}

// Usage demonstrating descendants
@Component({
  template: `
    <app-accordion>
      <!-- Direct child - counted by both -->
      <app-accordion-item>
        <div header>Item 1</div>
        <div content>Content 1</div>
      </app-accordion-item>
      
      <!-- Direct child - counted by both -->
      <app-accordion-item>
        <div header>Item 2</div>
        <div content>
          <!-- Nested accordion -->
          <app-accordion>
            <!-- Nested item - only counted by descendants:true -->
            <app-accordion-item>
              <div header>Nested Item</div>
              <div content>Nested Content</div>
            </app-accordion-item>
          </app-accordion>
        </div>
      </app-accordion-item>
    </app-accordion>
  `
})
export class AppComponent {}
// descendants: true → finds 3 items (2 direct + 1 nested)
// descendants: false → finds 2 items (only direct)

// Real-world: Menu system
@Component({
  selector: 'app-menu',
  template: `
    <nav class="menu">
      <ng-content></ng-content>
    </nav>
  `
})
export class MenuComponent implements AfterContentInit {
  // Get only top-level menu items
  @ContentChildren(MenuItemComponent, { descendants: false })
  topLevelItems: QueryList<MenuItemComponent>;
  
  // Get all menu items including nested
  @ContentChildren(MenuItemComponent, { descendants: true })
  allItems: QueryList<MenuItemComponent>;
  
  ngAfterContentInit() {
    console.log('Top level items:', this.topLevelItems.length);
    console.log('All items (including submenus):', this.allItems.length);
    
    // Setup keyboard navigation for top-level only
    this.setupKeyboardNav();
  }
  
  private setupKeyboardNav() {
    // Navigation logic for top-level items
  }
}

@Component({
  selector: 'app-menu-item',
  template: `
    <div class="menu-item" [class.has-children]="hasSubmenu">
      <ng-content select="[label]"></ng-content>
      
      <div class="submenu" *ngIf="hasSubmenu">
        <ng-content select="app-menu-item"></ng-content>
      </div>
    </div>
  `
})
export class MenuItemComponent implements AfterContentInit {
  @ContentChildren(MenuItemComponent) submenuItems: QueryList<MenuItemComponent>;
  
  hasSubmenu = false;
  
  ngAfterContentInit() {
    this.hasSubmenu = this.submenuItems.length > 0;
  }
}
```

---

### Q24: How to programmatically access and manipulate child components?

**Difficulty:** Hard

**Code:**
```typescript
// Child component with public API
@Component({
  selector: 'app-video-player',
  template: `
    <video #videoElement [src]="src">
      Your browser does not support video.
    </video>
    <div class="controls">
      <button (click)="play()">Play</button>
      <button (click)="pause()">Pause</button>
    </div>
  `
})
export class VideoPlayerComponent {
  @ViewChild('videoElement') videoElement: ElementRef<HTMLVideoElement>;
  @Input() src: string;
  
  private _isPlaying = false;
  
  get isPlaying(): boolean {
    return this._isPlaying;
  }
  
  play() {
    this.videoElement.nativeElement.play();
    this._isPlaying = true;
  }
  
  pause() {
    this.videoElement.nativeElement.pause();
    this._isPlaying = false;
  }
  
  seek(time: number) {
    this.videoElement.nativeElement.currentTime = time;
  }
  
  getCurrentTime(): number {
    return this.videoElement.nativeElement.currentTime;
  }
  
  getDuration(): number {
    return this.videoElement.nativeElement.duration;
  }
}

// Parent controlling child
@Component({
  selector: 'app-video-playlist',
  template: `
    <app-video-player 
      #player1 
      [src]="videos[0]">
    </app-video-player>
    
    <app-video-player 
      #player2 
      [src]="videos[1]">
    </app-video-player>
    
    <div class="playlist-controls">
      <button (click)="playAll()">Play All</button>
      <button (click)="pauseAll()">Pause All</button>
      <button (click)="playNext()">Play Next</button>
    </div>
  `
})
export class VideoPlaylistComponent implements AfterViewInit {
  @ViewChildren(VideoPlayerComponent) players: QueryList<VideoPlayerComponent>;
  
  videos = ['video1.mp4', 'video2.mp4'];
  currentIndex = 0;
  
  ngAfterViewInit() {
    console.log('Number of players:', this.players.length);
  }
  
  playAll() {
    this.players.forEach(player => player.play());
  }
  
  pauseAll() {
    this.players.forEach(player => player.pause());
  }
  
  playNext() {
    const playersArray = this.players.toArray();
    
    // Pause current
    if (playersArray[this.currentIndex]) {
      playersArray[this.currentIndex].pause();
    }
    
    // Move to next
    this.currentIndex = (this.currentIndex + 1) % playersArray.length;
    
    // Play next
    playersArray[this.currentIndex].play();
  }
  
  seekAllTo(time: number) {
    this.players.forEach(player => player.seek(time));
  }
  
  getPlayingStatus() {
    return this.players.map(player => ({
      isPlaying: player.isPlaying,
      currentTime: player.getCurrentTime(),
      duration: player.getDuration()
    }));
  }
}

// Advanced: Coordination between components
@Component({
  selector: 'app-form-step',
  template: `
    <div class="step" [class.active]="isActive">
      <ng-content></ng-content>
    </div>
  `
})
export class FormStepComponent {
  @Input() title: string;
  isActive = false;
  isValid = false;
  
  activate() {
    this.isActive = true;
  }
  
  deactivate() {
    this.isActive = false;
  }
  
  validate(): boolean {
    // Validation logic
    this.isValid = true; // Simplified
    return this.isValid;
  }
}

@Component({
  selector: 'app-stepper',
  template: `
    <div class="stepper">
      <div class="step-headers">
        <button 
          *ngFor="let step of steps; let i = index"
          [class.active]="i === currentStep"
          [disabled]="i > currentStep"
          (click)="goToStep(i)">
          {{ i + 1 }}. {{ step.title }}
        </button>
      </div>
      
      <div class="step-content">
        <ng-content></ng-content>
      </div>
      
      <div class="step-navigation">
        <button (click)="previous()" [disabled]="currentStep === 0">
          Previous
        </button>
        <button (click)="next()" [disabled]="currentStep === steps.length - 1">
          Next
        </button>
      </div>
    </div>
  `
})
export class StepperComponent implements AfterContentInit {
  @ContentChildren(FormStepComponent) steps: QueryList<FormStepComponent>;
  
  currentStep = 0;
  
  ngAfterContentInit() {
    // Activate first step
    if (this.steps.length > 0) {
      this.activateStep(0);
    }
    
    // Listen for dynamic step changes
    this.steps.changes.subscribe(() => {
      console.log('Steps changed');
    });
  }
  
  next() {
    const currentStepComponent = this.steps.toArray()[this.currentStep];
    
    // Validate current step
    if (!currentStepComponent.validate()) {
      console.log('Current step is invalid');
      return;
    }
    
    if (this.currentStep < this.steps.length - 1) {
      this.currentStep++;
      this.activateStep(this.currentStep);
    }
  }
  
  previous() {
    if (this.currentStep > 0) {
      this.currentStep--;
      this.activateStep(this.currentStep);
    }
  }
  
  goToStep(index: number) {
    if (index <= this.currentStep) {
      this.currentStep = index;
      this.activateStep(index);
    }
  }
  
  private activateStep(index: number) {
    this.steps.forEach((step, i) => {
      if (i === index) {
        step.activate();
      } else {
        step.deactivate();
      }
    });
  }
  
  isAllValid(): boolean {
    return this.steps.toArray().every(step => step.isValid);
  }
}
```

---

### Q25: How does read option work with queries?

**Difficulty:** Medium

**Code:**
```typescript
@Component({
  selector: 'app-read-demo',
  template: `
    <div #myDiv class="container">
      <p>Content here</p>
    </div>
  `
})
export class ReadDemoComponent implements AfterViewInit {
  // Default: Read as ElementRef
  @ViewChild('myDiv') divElement: ElementRef;
  
  // Read as ViewContainerRef
  @ViewChild('myDiv', { read: ViewContainerRef }) 
  divContainer: ViewContainerRef;
  
  // Read as TemplateRef (only works with ng-template)
  @ViewChild('myTemplate', { read: TemplateRef })
  template: TemplateRef<any>;
  
  ngAfterViewInit() {
    // ElementRef: Access native element
    console.log('Native element:', this.divElement.nativeElement);
    this.divElement.nativeElement.style.color = 'red';
    
    // ViewContainerRef: Create dynamic components
    console.log('ViewContainerRef:', this.divContainer);
    // Can use divContainer.createComponent() here
  }
}

// Real-world: Dynamic component loader
@Component({
  selector: 'app-dynamic-loader',
  template: `
    <div class="widget-container">
      <ng-template #widgetHost></ng-template>
    </div>
    
    <button (click)="loadComponent('chart')">Load Chart</button>
    <button (click)="loadComponent('table')">Load Table</button>
    <button (click)="clear()">Clear</button>
  `
})
export class DynamicLoaderComponent implements AfterViewInit {
  // Must read as ViewContainerRef for dynamic components
  @ViewChild('widgetHost', { read: ViewContainerRef })
  widgetHost: ViewContainerRef;
  
  private componentRef: ComponentRef<any>;
  
  ngAfterViewInit() {
    console.log('Widget host ready');
  }
  
  loadComponent(type: string) {
    // Clear previous
    this.clear();
    
    // Load new component
    if (type === 'chart') {
      this.componentRef = this.widgetHost.createComponent(ChartComponent);
      this.componentRef.instance.data = [1, 2, 3, 4, 5];
    } else if (type === 'table') {
      this.componentRef = this.widgetHost.createComponent(TableComponent);
      this.componentRef.instance.rows = ['Row 1', 'Row 2'];
    }
  }
  
  clear() {
    if (this.componentRef) {
      this.componentRef.destroy();
    }
    this.widgetHost.clear();
  }
}

@Component({
  selector: 'app-chart',
  template: '<div>Chart: {{ data }}</div>'
})
export class ChartComponent {
  @Input() data: number[];
}

@Component({
  selector: 'app-table',
  template: '<div>Table: {{ rows }}</div>'
})
export class TableComponent {
  @Input() rows: string[];
}

// Multiple read options
@Component({
  template: `
    <input #myInput type="text" value="Hello">
  `
})
export class MultipleReadComponent {
  // Same element, different perspectives
  @ViewChild('myInput') inputElement: ElementRef;
  @ViewChild('myInput', { read: ViewContainerRef }) inputContainer: ViewContainerRef;
  
  ngAfterViewInit() {
    // ElementRef: DOM manipulation
    console.log('Value:', this.inputElement.nativeElement.value);
    
    // ViewContainerRef: Dynamic component insertion
    // Could create components next to this input
  }
}
```

**read Options:**

| Type | Use Case |
|------|----------|
| **ElementRef** | DOM access (default) |
| **ViewContainerRef** | Dynamic components |
| **TemplateRef** | Template rendering |
| **Component** | Component instance |

---

## Component Styling

### Q26: What are the different ways to style components?

**Difficulty:** Easy

**Code:**
```typescript
// 1. External stylesheet
@Component({
  selector: 'app-user',
  templateUrl: './user.component.html',
  styleUrls: ['./user.component.css']
})
export class UserComponent {}

// 2. Inline styles
@Component({
  selector: 'app-inline',
  template: '<p>Styled inline</p>',
  styles: [`
    p {
      color: blue;
      font-size: 16px;
    }
  `]
})
export class InlineStyleComponent {}

// 3. Multiple stylesheets
@Component({
  selector: 'app-multi',
  templateUrl: './multi.component.html',
  styleUrls: [
    './multi.component.css',
    './multi-theme.css',
    './multi-responsive.css'
  ]
})
export class MultiStyleComponent {}

// 4. Template inline styles
@Component({
  selector: 'app-template-style',
  template: `
    <div style="color: red; font-size: 18px;">
      Inline style in template
    </div>
  `
})
export class TemplateStyleComponent {}

// 5. Dynamic styles with property binding
@Component({
  selector: 'app-dynamic',
  template: `
    <div [style.color]="textColor" [style.font-size.px]="fontSize">
      Dynamic style
    </div>
    
    <!-- Multiple styles -->
    <div [style]="dynamicStyles">
      Multiple dynamic styles
    </div>
  `
})
export class DynamicStyleComponent {
  textColor = 'blue';
  fontSize = 20;
  
  dynamicStyles = {
    'color': 'green',
    'font-size': '24px',
    'font-weight': 'bold'
  };
}

// 6. NgStyle directive
@Component({
  selector: 'app-ng-style',
  template: `
    <div [ngStyle]="{
      'color': isActive ? 'green' : 'red',
      'font-size': fontSize + 'px',
      'background-color': bgColor
    }">
      NgStyle example
    </div>
  `
})
export class NgStyleComponent {
  isActive = true;
  fontSize = 18;
  bgColor = '#f0f0f0';
}

// 7. CSS classes
@Component({
  selector: 'app-classes',
  template: `
    <!-- Static class -->
    <div class="container">Static class</div>
    
    <!-- Dynamic class -->
    <div [class.active]="isActive">Conditional class</div>
    
    <!-- Multiple classes -->
    <div [class]="'header ' + theme">Multiple classes</div>
    
    <!-- NgClass -->
    <div [ngClass]="{
      'active': isActive,
      'disabled': isDisabled,
      'large': size === 'large'
    }">
      NgClass example
    </div>
  `,
  styles: [`
    .container { padding: 20px; }
    .active { color: green; }
    .disabled { opacity: 0.5; }
    .large { font-size: 24px; }
  `]
})
export class ClassesComponent {
  isActive = true;
  isDisabled = false;
  size = 'large';
  theme = 'dark';
}
```

---

### Q27: How does View Encapsulation work?

**Difficulty:** Medium

**Code:**
```typescript
// Emulated (Default) - Scoped styles
@Component({
  selector: 'app-emulated',
  encapsulation: ViewEncapsulation.Emulated,
  template: '<div class="box">Emulated</div>',
  styles: [`
    .box {
      color: blue;
      border: 1px solid blue;
    }
  `]
})
export class EmulatedComponent {}
// Generated: .box[_ngcontent-c0] { color: blue; }

// None - Global styles
@Component({
  selector: 'app-none',
  encapsulation: ViewEncapsulation.None,
  template: '<div class="box">None</div>',
  styles: [`
    .box {
      color: red;
      border: 1px solid red;
    }
  `]
})
export class NoneComponent {}
// Generated: .box { color: red; } - affects entire app!

// ShadowDom - True encapsulation
@Component({
  selector: 'app-shadow',
  encapsulation: ViewEncapsulation.ShadowDom,
  template: '<div class="box">Shadow DOM</div>',
  styles: [`
    .box {
      color: green;
      border: 1px solid green;
    }
  `]
})
export class ShadowComponent {}
// Uses native Shadow DOM

// Comparison example
@Component({
  selector: 'app-root',
  template: `
    <style>
      /* Global style */
      .box {
        padding: 20px;
        margin: 10px;
      }
    </style>
    
    <app-emulated></app-emulated>
    <app-none></app-none>
    <app-shadow></app-shadow>
    
    <div class="box">Root box</div>
  `,
  styles: [`
    .box {
      background: yellow;
    }
  `]
})
export class AppComponent {}
```

**Results:**
- **Emulated** .box: blue text, global padding/margin, yellow background
- **None** .box: red text (overrides all), global padding/margin
- **Shadow** .box: green text, NO global styles apply
- **Root** .box: yellow background, global padding/margin

---

### Q28: How to use :host, :host-context, and ::ng-deep?

**Difficulty:** Hard

**Code:**
```typescript
// :host - Style the component itself
@Component({
  selector: 'app-card',
  template: `
    <div class="content">
      <ng-content></ng-content>
    </div>
  `,
  styles: [`
    /* Style the host element */
    :host {
      display: block;
      border: 1px solid #ccc;
      border-radius: 8px;
      padding: 16px;
      background: white;
    }
    
    /* Style host when it has a class */
    :host(.highlighted) {
      border-color: blue;
      box-shadow: 0 0 10px rgba(0,0,255,0.3);
    }
    
    /* Style host based on state */
    :host(:hover) {
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }
    
    /* Style host based on attribute */
    :host([disabled]) {
      opacity: 0.5;
      pointer-events: none;
    }
    
    .content {
      color: #333;
    }
  `]
})
export class CardComponent {}
// Usage: <app-card class="highlighted" disabled></app-card>

// :host-context - Style based on ancestor
@Component({
  selector: 'app-button',
  template: '<button><ng-content></ng-content></button>',
  styles: [`
    /* Default style */
    button {
      background: blue;
      color: white;
    }
    
    /* Style when inside .dark-theme ancestor */
    :host-context(.dark-theme) button {
      background: #333;
      color: #fff;
    }
    
    /* Style when inside .light-theme ancestor */
    :host-context(.light-theme) button {
      background: #fff;
      color: #333;
      border: 1px solid #333;
    }
    
    /* Multiple ancestors */
    :host-context(.mobile) :host-context(.dark-theme) button {
      font-size: 18px;
    }
  `]
})
export class ButtonComponent {}

// ::ng-deep (deprecated but still used)
@Component({
  selector: 'app-parent',
  template: `
    <div class="wrapper">
      <app-child></app-child>
    </div>
  `,
  styles: [`
    /* This won't work - encapsulated */
    .child-class {
      color: red;
    }
    
    /* This works - pierces encapsulation */
    ::ng-deep .child-class {
      color: blue;
    }
    
    /* Better: Scope with :host */
    :host ::ng-deep .child-class {
      color: green;
    }
    
    /* Even better: Use ViewEncapsulation.None on child */
  `]
})
export class ParentComponent {}

// Real-world example: Theming
@Component({
  selector: 'app-themed-card',
  template: `
    <div class="card-header">
      <ng-content select="[header]"></ng-content>
    </div>
    <div class="card-body">
      <ng-content></ng-content>
    </div>
  `,
  styles: [`
    :host {
      display: block;
      border-radius: 8px;
      overflow: hidden;
    }
    
    /* Default theme */
    .card-header {
      background: #f5f5f5;
      padding: 16px;
      border-bottom: 1px solid #ddd;
    }
    
    .card-body {
      padding: 16px;
      background: white;
    }
    
    /* Dark theme via ancestor */
    :host-context(.dark-mode) {
      .card-header {
        background: #2a2a2a;
        border-bottom-color: #444;
        color: white;
      }
      
      .card-body {
        background: #1a1a1a;
        color: #ccc;
      }
    }
    
    /* Compact variant */
    :host(.compact) {
      .card-header,
      .card-body {
        padding: 8px;
      }
    }
    
    /* Interactive states */
    :host(:not([disabled]):hover) {
      box-shadow: 0 4px 12px rgba(0,0,0,0.15);
      transform: translateY(-2px);
      transition: all 0.2s ease;
    }
    
    :host([disabled]) {
      opacity: 0.6;
      cursor: not-allowed;
    }
  `]
})
export class ThemedCardComponent {}

// Usage
@Component({
  template: `
    <div class="dark-mode">
      <app-themed-card class="compact">
        <div header>Card Title</div>
        <p>Card content</p>
      </app-themed-card>
    </div>
  `
})
export class AppComponent {}
```

**Special Selectors:**
- **:host**: Style the component element
- **:host()**: Style host with condition
- **:host-context()**: Style based on ancestor
- **::ng-deep**: Pierce encapsulation (deprecated)

---

### Q29: How to implement dynamic theming?

**Difficulty:** Hard

**Code:**
```typescript
// Theme service
@Injectable({ providedIn: 'root' })
export class ThemeService {
  private themeSubject = new BehaviorSubject<string>('light');
  theme$ = this.themeSubject.asObservable();
  
  setTheme(theme: string) {
    this.themeSubject.next(theme);
    document.body.className = `theme-${theme}`;
    localStorage.setItem('theme', theme);
  }
  
  getTheme(): string {
    return this.themeSubject.value;
  }
  
  toggleTheme() {
    const current = this.getTheme();
    const newTheme = current === 'light' ? 'dark' : 'light';
    this.setTheme(newTheme);
  }
  
  initTheme() {
    const saved = localStorage.getItem('theme') || 'light';
    this# Angular Components - 50 Interview Questions

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
