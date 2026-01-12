# Angular Components - 50 Interview Questions

## Table of Contents
- [Component Basics](#component-basics)
- [Component Communication](#component-communication)
- [Input/Output Deep Dive](#inputoutput-deep-dive)
- [Component Architecture](#component-architecture)
- [Dynamic Components](#dynamic-components)
- [Component Styling](#component-styling)
- [Component Interaction](#component-interaction)
- [Performance Optimization](#performance-optimization)
- [Advanced Patterns](#advanced-patterns)
- [Best Practices](#best-practices)

---

## Component Basics

### Q1: What are the core building blocks of an Angular component?

**Difficulty:** Easy

**Code:**
```typescript
// Complete component structure
@Component({
  // Required: Component metadata
  selector: 'app-user-profile',           // How to use in templates
  templateUrl: './user-profile.component.html',  // External template
  // OR
  template: `<div>Inline template</div>`,      // Inline template
  
  styleUrls: ['./user-profile.component.css'],  // External styles
  // OR
  styles: [`
    .container { padding: 20px; }
  `],
  
  // Optional metadata
  changeDetection: ChangeDetectionStrategy.OnPush,
  encapsulation: ViewEncapsulation.Emulated,
  providers: [UserService],
  animations: [/* ... */]
})
export class UserProfileComponent implements OnInit, OnDestroy {
  // Properties
  @Input() userId: string;
  @Output() userLoaded = new EventEmitter<User>();
  @ViewChild('userCard') userCard: ElementRef;
  @ContentChild('headerTemplate') headerTemplate: TemplateRef<any>;
  
  // Lifecycle hooks
  constructor(private userService: UserService) {
    console.log('Constructor called');
  }
  
  ngOnInit() {
    console.log('Component initialized');
  }
  
  ngOnDestroy() {
    console.log('Component destroyed');
  }
}
```

---

## Component Communication

### Q12: What are all the ways components can communicate?

**Difficulty:** Medium

**Code:**
```typescript
// 1️⃣ Parent to Child: @Input
@Component({
  selector: 'app-parent',
  template: '<app-child [message]="parentMessage"></app-child>'
})
export class ParentComponent {
  parentMessage = 'Hello from parent';
}

@Component({
  selector: 'app-child',
  template: '<p>{{ message }}</p>'
})
export class ChildComponent {
  @Input() message: string;
}

// 2️⃣ Child to Parent: @Output and EventEmitter
@Component({
  selector: 'app-child',
  template: '<button (click)="sendMessage()">Send</button>'
})
export class ChildComponent {
  @Output() messageEvent = new EventEmitter<string>();
  
  sendMessage() {
    this.messageEvent.emit('Hello from child');
  }
}

// 3️⃣ ViewChild/ContentChild
@Component({})
export class ParentComponent implements AfterViewInit {
  @ViewChild(ChildComponent) child: ChildComponent;
  
  ngAfterViewInit() {
    // Access child component methods/properties
    this.child.someMethod();
  }
}

// 4️⃣ Service with Subject
@Injectable()
export class DataService {
  private dataSubject = new BehaviorSubject<any>(null);
  data$ = this.dataSubject.asObservable();
  
  updateData(data: any) {
    this.dataSubject.next(data);
  }
}
```

**Best Practices:**
- Always unsubscribe in ngOnDestroy
- Use takeUntil pattern for observables
- Clean up event listeners
- Clear intervals and timeouts
- Destroy dynamic components
- Close WebSocket connections
- Stop media streams

---

## Best Practices

### Q25: What are the lifecycle hook best practices?

**Difficulty:** Medium

**Complete Guide:**
```typescript
// ✅ BEST PRACTICES CHECKLIST

// 1️⃣ Constructor: Minimal logic
@Component({})
export class GoodComponent {
  constructor(
    private service: DataService,
    private router: Router
  ) {
    // ✅ ONLY dependency injection
    // ❌ NO: API calls, subscriptions, DOM manipulation
  }
}

// 2️⃣ ngOnInit: Initialization logic
ngOnInit() {
  // ✅ API calls
  this.loadData();
  
  // ✅ Setup subscriptions
  this.setupSubscriptions();
  
  // ✅ Initialize properties
  this.initializeComponent();
}

// 3️⃣ ngOnChanges: React to @Input changes
ngOnChanges(changes: SimpleChanges) {
  if (changes['data'] && !changes['data'].firstChange) {
    this.processData(changes['data'].currentValue);
  }
}

// 4️⃣ ngDoCheck: Use sparingly (performance)
ngDoCheck() {
  // Only for custom change detection
  // Use IterableDiffer or KeyValueDiffer
}

// 5️⃣ ngAfterViewInit: DOM manipulation
ngAfterViewInit() {
  // Safe to access @ViewChild
  this.element.nativeElement.focus();
}

// 6️⃣ ngOnDestroy: Clean everything
ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
  clearInterval(this.interval);
  // ... all cleanup
}
```

**Best Practices:**
1. Use constructor for dependency injection only
2. Use ngOnInit for initialization logic
3. Use ngOnChanges to react to input changes
4. Avoid ngDoCheck unless absolutely necessary
5. Query children in ngAfterViewInit/ngAfterContentInit
6. Always cleanup in ngOnDestroy
7. Use takeUntil pattern for subscriptions
8. Destroy dynamic components properly

---

## Performance Optimization

### Q12: How can lifecycle hooks impact performance?

**Difficulty:** Hard

**Code:**
```typescript
// ❌ BAD: Performance killers
@Component({})
export class BadPerformanceComponent implements DoCheck, AfterViewChecked {
  @Input() data: any[];
  
  ngDoCheck() {
    // ❌ Runs on EVERY change detection cycle
    // ❌ Expensive operation
    console.log('Checking...', JSON.stringify(this.data));
    this.processData();
  }
  
  ngAfterViewChecked() {
    // ❌ Runs on EVERY change detection
    console.log('View checked');
    this.updateUI(); // Expensive operation
  }
}

// ✅ GOOD: Optimized lifecycle usage
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class OptimizedComponent implements OnInit, OnChanges, OnDestroy {
  @Input() data: any;
  private destroy$ = new Subject<void>();
  
  // ✅ Use ngOnChanges for input changes
  ngOnChanges(changes: SimpleChanges) {
    if (changes['data']) {
      this.processData();
    }
  }
  
  // ✅ Use ngOnInit for initialization
  ngOnInit() {
    this.loadData();
  }
  
  ngOnDestroy() {
    // ✅ Always cleanup
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**Performance Comparison:**

| Hook | Frequency | Use Case | Performance |
|------|-----------|----------|-------------|
| ngOnChanges | Per @Input change | Input validation | ✅ Good |
| ngOnInit | Once | Initialization | ✅ Excellent |
| ngDoCheck | Every CD | Custom change detection | ⚠️ Use carefully |
| AfterViewInit | Once | DOM ready | ✅ Excellent |
| ngOnDestroy | Once | Cleanup | ✅ Required |

---

## Conclusion

### Key Takeaways:

1. **Lifecycle Order**: Understand the exact execution sequence
2. **Constructor vs ngOnInit**: Use constructor only for DI
3. **ngOnChanges**: Only detects reference changes
4. **ngDoCheck**: Powerful but expensive - use sparingly
5. **Content vs View**: Know the difference and when each is ready
6. **ngOnDestroy**: Critical for cleanup - prevent memory leaks
7. **Dynamic Components**: Always destroy properly
8. **Performance**: Choose the right hook for the right job

### Best Practices:
- ✅ Use ngOnInit for initialization logic
- ✅ Use ngOnChanges for @Input tracking
- ✅ Avoid ngDoCheck unless necessary
- ✅ Always implement ngOnDestroy for cleanup
- ✅ Use OnPush with immutable patterns
- ✅ Query children in After*Init hooks
- ✅ Clean up subscriptions, intervals, listeners
- ✅ Understand static vs dynamic queries

---

**End of Lifecycle Hooks Guide**

*Angular Version: 15-18+*
*Last Updated: 2025*