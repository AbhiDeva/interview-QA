# Angular Forms Performance - 25 Interview Questions

## Table of Contents
- [Forms Performance Basics](#forms-performance-basics)
- [Change Detection Optimization](#change-detection-optimization)
- [ValueChanges Performance](#valuechanges-performance)
- [FormArray Performance](#formarray-performance)
- [Validators Performance](#validators-performance)
- [Async Validators](#async-validators)
- [Dynamic Forms](#dynamic-forms)
- [Form State Management](#form-state-management)
- [Large Forms Optimization](#large-forms-optimization)
- [Memory Leaks](#memory-leaks)
- [Best Practices](#best-practices)

---

## Forms Performance Basics

### Q1: What's the performance difference between Template-driven and Reactive Forms?

**Difficulty:** Medium

**Question:**
```typescript
// Template-driven (less performant for complex forms)
@Component({
  template: `
    <form #form="ngForm">
      <input [(ngModel)]="user.name" name="name">
      <input [(ngModel)]="user.email" name="email">
    </form>
  `
})
export class TemplateFormComponent {
  user = { name: '', email: '' };
}

// Reactive (better performance)
@Component({
  template: `
    <form [formGroup]="userForm">
      <input formControlName="name">
      <input formControlName="email">
    </form>
  `
})
export class ReactiveFormComponent {
  userForm = new FormGroup({
    name: new FormControl(''),
    email: new FormControl('')
  });
}
```

**Answer:**
Reactive Forms are more performant because:
- Change detection is more efficient (explicit state management)
- No template parsing overhead
- Better for dynamic forms (programmatic control)
- Immutable data structure reduces unnecessary checks
- Easier to test and optimize
- Direct access to form state without template queries

**Performance Impact:** 2-5x faster for complex forms with 10+ fields

**When to use each:**
- Template-driven: Simple forms (1-5 fields), prototypes
- Reactive: Complex forms, dynamic forms, enterprise applications

---

## Change Detection Optimization

### Q2: How does OnPush strategy improve form performance?

**Difficulty:** Hard

**Code:**
```typescript
@Component({
  selector: 'app-user-form',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <form [formGroup]="userForm">
      <input formControlName="name">
      <input formControlName="email">
      <input formControlName="age">
    </form>
    <div>Form Status: {{ userForm.status }}</div>
  `
})
export class UserFormComponent {
  userForm = new FormGroup({
    name: new FormControl(''),
    email: new FormControl(''),
    age: new FormControl('')
  });
  
  constructor(private cdr: ChangeDetectorRef) {}
}
```

**Answer:**
OnPush with Reactive Forms:
- Skips unnecessary change detection cycles
- Only runs CD on:
  - @Input changes (reference change)
  - Events from template (click, input, etc.)
  - Async pipe emissions
  - Manual markForCheck()
- Reduces CD cycles by 50-90%
- Must use immutable patterns
- Works perfectly with Reactive Forms (they emit events)

**Performance Metrics:**
- Default CD: Checks every component on every event
- OnPush CD: Checks only affected components
- Improvement: 50-90% reduction in CD cycles

**Important Note:** OnPush requires proper reactive patterns. Form updates trigger CD automatically because FormControl events are Observable-based.

---

### Q3: What's the impact of using getters in templates with forms?

**Difficulty:** Medium

**Bad Code:**
```typescript
@Component({
  template: `
    <div *ngFor="let item of getFormArray().controls">
      <input [formControl]="item">
    </div>
    <button [disabled]="isFormInvalid()">Submit</button>
  `
})
export class BadFormComponent {
  form = new FormGroup({
    items: new FormArray([])
  });
  
  // Called multiple times per change detection!
  getFormArray(): FormArray {
    return this.form.get('items') as FormArray;
  }
  
  // Called on every CD cycle!
  isFormInvalid(): boolean {
    return !this.form.valid || this.isProcessing;
  }
}
```

**Good Code:**
```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div *ngFor="let item of items.controls; trackBy: trackByIndex">
      <input [formControl]="item">
    </div>
    <button [disabled]="isFormInvalid$ | async">Submit</button>
  `
})
export class GoodFormComponent {
  items!: FormArray;
  
  isFormInvalid$ = combineLatest([
    this.form.statusChanges.pipe(startWith(this.form.status)),
    this.isProcessing$
  ]).pipe(
    map(([status, processing]) => status !== 'VALID' || processing)
  );
  
  ngOnInit() {
    this.items = this.form.get('items') as FormArray;
  }
  
  trackByIndex(index: number): number {
    return index;
  }
}
```

**Answer:**
Issues with getters in templates:
- Called multiple times per CD cycle (2-10+ times)
- O(n²) complexity with nested getters
- Creates new references (breaks OnPush)
- Causes unnecessary re-renders

**Performance Impact:**
- Getters: 10-50 calls per user interaction
- Cached properties: 1 call per actual change
- Improvement: 10-50x fewer function calls

---

## ValueChanges Performance

### Q4: What's wrong with this valueChanges subscription?

**Difficulty:** Hard

**Bad Code:**
```typescript
export class SearchComponent implements OnInit {
  searchForm = new FormGroup({
    query: new FormControl('')
  });
  
  searchResults: any[] = [];
  
  ngOnInit() {
    // ❌ Multiple performance issues!
    this.searchForm.get('query').valueChanges.subscribe(value => {
      this.searchResults = this.searchService.search(value);
    });
  }
}
```

**Issues:**
1. ❌ No debounce - fires on every keystroke
2. ❌ No unsubscribe - memory leak
3. ❌ No distinctUntilChanged - duplicate calls
4. ❌ Synchronous operation blocks UI
5. ❌ No error handling
6. ❌ No loading state management

**Performance Impact:**
- API calls: 50-100+ per minute (typing "angular" = 7 calls)
- Memory leak: grows with each component instance
- UI blocking: janky input experience

---

### Q5: Fix the performance issues in valueChanges:

**Difficulty:** Medium

**Optimized Code:**
```typescript
export class SearchComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  
  searchForm = new FormGroup({
    query: new FormControl('')
  });
  
  searchResults$ = this.searchForm.get('query').valueChanges.pipe(
    debounceTime(300),              // Wait 300ms after typing stops
    distinctUntilChanged(),         // Ignore if same as previous
    filter(query => query.length >= 3), // Min 3 characters
    tap(() => this.loading = true), // Show loading state
    switchMap(query =>              // Cancel previous requests
      this.searchService.search(query).pipe(
        catchError(err => {
          console.error('Search error:', err);
          return of([]);
        })
      )
    ),
    tap(() => this.loading = false),
    takeUntil(this.destroy$)        // Auto-unsubscribe
  );
  
  loading = false;
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// Template
@Component({
  template: `
    <form [formGroup]="searchForm">
      <input formControlName="query" placeholder="Search...">
      <div *ngIf="loading">Searching...</div>
      <div *ngFor="let result of searchResults$ | async">
        {{ result.name }}
      </div>
    </form>
  `
})
```

**Performance Improvements:**
- `debounceTime(300)`: Reduces API calls by **90%** (from 50+ to ~5 per minute)
- `distinctUntilChanged()`: Prevents duplicate API calls
- `switchMap`: Cancels pending requests (prevents race conditions)
- `takeUntil`: Prevents memory leaks
- `filter`: Avoids unnecessary searches for short queries
- `catchError`: Graceful error handling

**Metrics:**
- Before: 50+ API calls/minute, memory leaks, janky UI
- After: ~5 API calls/minute, no leaks, smooth UX
- **Improvement: 10x better performance**

---

## FormArray Performance

### Q6: What's the performance issue with FormArray?

**Difficulty:** Hard

**Bad Code:**
```typescript
@Component({
  template: `
    <div *ngFor="let item of getItems().controls">
      <input [formControl]="item.get('name')">
      <input [formControl]="item.get('email')">
      <button (click)="removeItem(i)">Remove</button>
    </div>
  `
})
export class FormArrayComponent {
  form = new FormGroup({
    items: new FormArray([])
  });
  
  // ❌ Called multiple times per CD cycle!
  getItems(): FormArray {
    return this.form.get('items') as FormArray;
  }
  
  addItem() {
    const items = this.getItems();
    items.push(new FormGroup({
      name: new FormControl(''),
      email: new FormControl('')
    }));
  }
}
```

**Issues:**
1. ❌ Getter called 2-10+ times per CD cycle
2. ❌ No trackBy function (full DOM recreation)
3. ❌ Nested getter calls (O(n²) complexity)
4. ❌ Creates new reference each time
5. ❌ Triggers unnecessary re-renders

**Performance Impact:**
- 10 items: ~50 getter calls per interaction
- 100 items: ~500+ getter calls per interaction
- Full DOM recreation on any change

---

### Q7: Optimize FormArray performance:

**Difficulty:** Medium

**Optimized Code:**
```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div *ngFor="let item of items.controls; let i = index; trackBy: trackByIndex">
      <div [formGroup]="item">
        <input formControlName="name">
        <input formControlName="email">
        <button (click)="removeItem(i)">Remove</button>
      </div>
    </div>
    <button (click)="addItem()">Add Item</button>
  `
})
export class OptimizedFormArrayComponent implements OnInit {
  form = new FormGroup({
    items: new FormArray([])
  });
  
  // ✅ Cached reference - computed once
  items!: FormArray;
  
  ngOnInit() {
    this.items = this.form.get('items') as FormArray;
  }
  
  addItem() {
    this.items.push(new FormGroup({
      name: new FormControl(''),
      email: new FormControl('')
    }));
  }
  
  removeItem(index: number) {
    this.items.removeAt(index);
  }
  
  // ✅ TrackBy prevents DOM recreation
  trackByIndex(index: number): number {
    return index;
  }
}
```

**Performance Improvements:**
- ✅ Cached FormArray reference (single access)
- ✅ trackBy prevents full DOM recreation
- ✅ OnPush change detection
- ✅ Direct formGroup binding
- ✅ O(n) instead of O(n²) complexity

**Metrics:**
- Before: 500+ function calls for 100 items
- After: 1 access + incremental DOM updates
- **Improvement: 5-10x faster for large lists**

**Advanced trackBy with FormControl:**
```typescript
trackByFormControl(index: number, control: AbstractControl): any {
  return control; // Track by control instance
}
```

---

## Validators Performance

### Q8: What's inefficient about this custom validator?

**Difficulty:** Medium

**Bad Code:**
```typescript
function emailValidator(control: AbstractControl): ValidationErrors | null {
  // ❌ Regex compiled on EVERY validation call!
  const emailRegex = /^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,4}$/;
  const valid = emailRegex.test(control.value);
  return valid ? null : { email: true };
}

// Usage
@Component({})
export class FormComponent {
  form = new FormGroup({
    email: new FormControl('', emailValidator) // Called on every keystroke!
  });
}
```

**Issues:**
1. ❌ Regex compiled on every validation (expensive)
2. ❌ Runs on every keystroke (no debounce)
3. ❌ No early return for empty values
4. ❌ No caching mechanism
5. ❌ Can cause input lag with complex regex

**Performance Impact:**
- Typing 10 characters = 10 regex compilations
- Complex regex = 5-50ms per validation
- Result: Noticeable input lag

---

### Q9: Optimize the email validator:

**Difficulty:** Medium

**Optimized Code:**
```typescript
// ✅ Regex compiled once (static/const)
const EMAIL_REGEX = /^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,4}$/;

function optimizedEmailValidator(control: AbstractControl): ValidationErrors | null {
  // ✅ Early return for empty values
  if (!control.value) {
    return null;
  }
  
  // ✅ Use pre-compiled regex
  const valid = EMAIL_REGEX.test(control.value);
  return valid ? null : { email: true };
}

// ✅ OR use Angular's built-in validator
@Component({})
export class OptimizedFormComponent {
  form = new FormGroup({
    email: new FormControl('', [
      Validators.email,  // Built-in, optimized validator
      Validators.required
    ])
  });
}

// ✅ For expensive validators, use updateOn: 'blur'
export class BlurValidationComponent {
  form = new FormGroup({
    email: new FormControl('', {
      validators: [Validators.email, customExpensiveValidator],
      updateOn: 'blur'  // Only validate on blur, not every keystroke
    })
  });
}
```

**Performance Improvements:**
- ✅ Static regex (compiled once)
- ✅ Early return optimization
- ✅ Built-in validators when possible
- ✅ updateOn: 'blur' for expensive validators

**Metrics:**
- Before: 10ms per validation × 50 keystrokes = 500ms
- After: <1ms per validation × 50 keystrokes = 50ms
- **Improvement: 10x faster validation**

**Advanced: Cached Validator Factory**
```typescript
function createCachedValidator(validatorFn: Function) {
  const cache = new Map<any, ValidationErrors | null>();
  
  return (control: AbstractControl): ValidationErrors | null => {
    const value = control.value;
    
    if (cache.has(value)) {
      return cache.get(value);
    }
    
    const result = validatorFn(control);
    cache.set(value, result);
    return result;
  };
}

// Usage
const cachedEmailValidator = createCachedValidator(emailValidator);
```

---

## Async Validators

### Q10: What's the performance problem with this async validator?

**Difficulty:** Hard

**Bad Code:**
```typescript
function usernameAvailabilityValidator(http: HttpClient) {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    // ❌ API call on EVERY keystroke!
    return http.get<boolean>(`/api/check-username/${control.value}`).pipe(
      map(available => available ? null : { usernameTaken: true })
    );
  };
}

@Component({})
export class RegistrationComponent {
  form = new FormGroup({
    username: new FormControl('', {
      asyncValidators: usernameAvailabilityValidator(this.http)
    })
  });
}
```

**Issues:**
1. ❌ No debounce (API call per keystroke)
2. ❌ No request cancellation (race conditions)
3. ❌ No caching (duplicate checks)
4. ❌ No minimum length check
5. ❌ Expensive network overhead

**Performance Impact:**
- Typing "angular" = 7 API calls
- Backspacing = more API calls
- Total: 20-50+ API calls for one field
- Network cost + server load

---

### Q11: Optimize the async validator:

**Difficulty:** Hard

**Optimized Code:**
```typescript
function optimizedUsernameValidator(http: HttpClient) {
  // ✅ Cache for results
  const cache = new Map<string, boolean>();
  
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    const username = control.value;
    
    // ✅ Early return for empty/short values
    if (!username || username.length < 3) {
      return of(null);
    }
    
    // ✅ Check cache first
    if (cache.has(username)) {
      const available = cache.get(username);
      return of(available ? null : { usernameTaken: true });
    }
    
    // ✅ Debounce + cancellation via switchMap
    return of(username).pipe(
      debounceTime(500),  // Wait 500ms after typing stops
      switchMap(name =>   // Cancel previous requests
        http.get<boolean>(`/api/check-username/${name}`).pipe(
          map(available => {
            cache.set(name, available);  // ✅ Cache result
            return available ? null : { usernameTaken: true };
          }),
          catchError(() => of(null))  // ✅ Handle errors gracefully
        )
      )
    );
  };
}

// Usage with additional optimization
@Component({})
export class OptimizedRegistrationComponent {
  form = new FormGroup({
    username: new FormControl('', {
      validators: [Validators.required, Validators.minLength(3)],
      asyncValidators: optimizedUsernameValidator(this.http),
      updateOn: 'blur'  // ✅ Only validate on blur!
    })
  });
}
```

**Performance Improvements:**
- ✅ debounceTime(500): Reduces API calls by **90%**
- ✅ Caching: Eliminates duplicate checks
- ✅ switchMap: Cancels obsolete requests
- ✅ updateOn: 'blur': Validates only when needed
- ✅ Early returns: Skips validation for invalid input

**Metrics:**
- Before: 50+ API calls while typing
- After: 1-2 API calls per field
- **Improvement: 95% reduction in API calls**

**Advanced: Shared Cache Service**
```typescript
@Injectable()
export class ValidationCacheService {
  private cache = new Map<string, Map<any, any>>();
  
  get(validatorName: string, key: any): any {
    return this.cache.get(validatorName)?.get(key);
  }
  
  set(validatorName: string, key: any, value: any): void {
    if (!this.cache.has(validatorName)) {
      this.cache.set(validatorName, new Map());
    }
    this.cache.get(validatorName).set(key, value);
  }
  
  clear(validatorName?: string): void {
    if (validatorName) {
      this.cache.delete(validatorName);
    } else {
      this.cache.clear();
    }
  }
}
```

---

## Dynamic Forms

### Q12: What's inefficient in this dynamic form?

**Difficulty:** Hard

**Bad Code:**
```typescript
@Component({})
export class DynamicFormComponent {
  form = new FormGroup({
    fields: new FormArray([])
  });
  
  addField(name: string, type: string) {
    const formArray = this.form.get('fields') as FormArray;
    
    // ❌ Manual change detection
    formArray.push(new FormGroup({
      name: new FormControl(name),
      type: new FormControl(type),
      value: new FormControl('')
    }));
    
    this.cdr.detectChanges(); // ❌ Expensive!
  }
  
  addMultipleFields(count: number) {
    // ❌ Not batched - triggers CD for each addition
    for (let i = 0; i < count; i++) {
      this.addField(`field${i}`, 'text');
    }
  }
}
```

**Issues:**
1. ❌ Manual detectChanges is expensive
2. ❌ No batching for multiple additions
3. ❌ Triggers full CD tree for each operation
4. ❌ O(n) operation becomes O(n²)
5. ❌ Creates FormGroup on each call

**Performance Impact:**
- Adding 100 fields = 100 CD cycles
- Each CD checks entire component tree
- Result: Multi-second UI freeze

---

### Q13: Optimize dynamic form operations:

**Difficulty:** Medium

**Optimized Code:**
```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class OptimizedDynamicFormComponent {
  form = new FormGroup({
    fields: new FormArray([])
  });
  
  // ✅ Batch additions
  addMultipleFields(fields: Array<{name: string, type: string}>) {
    const formArray = this.form.get('fields') as FormArray;
    
    // ✅ Create all controls first
    const controls = fields.map(field =>
      new FormGroup({
        name: new FormControl(field.name),
        type: new FormControl(field.type),
        value: new FormControl('')
      })
    );
    
    // ✅ Add all at once - single CD cycle
    controls.forEach(control => formArray.push(control));
    
    // No manual detectChanges needed with OnPush + FormArray events
  }
  
  // ✅ Factory pattern for FormGroup creation
  private createFieldControl(name: string, type: string): FormGroup {
    return new FormGroup({
      name: new FormControl(name),
      type: new FormControl(type),
      value: new FormControl('')
    });
  }
  
  // ✅ Optimized single addition
  addField(name: string, type: string) {
    const formArray = this.form.get('fields') as FormArray;
    formArray.push(this.createFieldControl(name, type));
  }
  
  // ✅ Bulk remove (also batched)
  removeFields(indices: number[]) {
    const formArray = this.form.get('fields') as FormArray;
    
    // Sort in reverse to avoid index shifting
    indices.sort((a, b) => b - a).forEach(index => {
      formArray.removeAt(index);
    });
  }
}
```

**Performance Improvements:**
- ✅ Batched operations (single CD cycle)
- ✅ OnPush change detection
- ✅ Factory pattern for reusability
- ✅ No manual detectChanges
- ✅ Array methods for efficiency

**Metrics:**
- Before: 100 fields = 100 CD cycles = 2-5 seconds
- After: 100 fields = 1 CD cycle = 50-100ms
- **Improvement: 20-50x faster for bulk operations**

**Advanced: Lazy FormGroup Creation**
```typescript
class LazyFormArray extends FormArray {
  private controlFactory: () => AbstractControl;
  
  constructor(
    controls: AbstractControl[],
    controlFactory: () => AbstractControl,
    validatorOrOpts?: ValidatorFn | ValidatorFn[] | AbstractControlOptions | null,
    asyncValidator?: AsyncValidatorFn | AsyncValidatorFn[] | null
  ) {
    super(controls, validatorOrOpts, asyncValidator);
    this.controlFactory = controlFactory;
  }
  
  pushLazy(count: number) {
    const controls = Array.from({ length: count }, () => this.controlFactory());
    controls.forEach(control => this.push(control));
  }
}
```

---

## Form State Management

### Q14: Why is this statusChanges subscription inefficient?

**Difficulty:** Medium

**Bad Code:**
```typescript
export class FormComponent implements OnInit {
  form = new FormGroup({
    name: new FormControl(''),
    email: new FormControl('')
  });
  
  isValid = false;
  isInvalid = false;
  isPending = false;
  
  ngOnInit() {
    // ❌ Fires on every form change
    this.form.statusChanges.subscribe(status => {
      console.log('Status changed:', status);
      
      // ❌ Multiple property assignments
      this.isValid = status === 'VALID';
      this.isInvalid = status === 'INVALID';
      this.isPending = status === 'PENDING';
      
      // ❌ Triggers change detection for each assignment
    });
    
    // ❌ No unsubscribe - memory leak
  }
}
```

**Issues:**
1. ❌ Fires on every keystroke
2. ❌ No debounce or distinctUntilChanged
3. ❌ Multiple property assignments
4. ❌ Console.log in production
5. ❌ No unsubscribe (memory leak)
6. ❌ Triggers multiple CD cycles

---

### Q15: Optimize form status subscription:

**Difficulty:** Medium

**Optimized Code:**
```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <form [formGroup]="form">
      <input formControlName="name">
      <input formControlName="email">
    </form>
    
    <div *ngIf="isValid$ | async" class="success">
      Form is valid!
    </div>
    
    <button [disabled]="isInvalid$ | async">
      Submit
    </button>
  `
})
export class OptimizedFormComponent {
  form = new FormGroup({
    name: new FormControl('', Validators.required),
    email: new FormControl('', [Validators.required, Validators.email])
  });
  
  // ✅ Reactive streams
  status$ = this.form.statusChanges.pipe(
    startWith(this.form.status),
    distinctUntilChanged(),  // Only emit on actual changes
    shareReplay(1)           // Share subscription, cache last value
  );
  
  isValid$ = this.status$.pipe(
    map(status => status === 'VALID')
  );
  
  isInvalid$ = this.status$.pipe(
    map(status => status === 'INVALID')
  );
  
  isPending$ = this.status$.pipe(
    map(status => status === 'PENDING')
  );
  
  // ✅ No manual subscriptions = no memory leaks
  // ✅ Async pipe handles unsubscribe automatically
}
```

**Performance Improvements:**
- ✅ distinctUntilChanged: Prevents duplicate emissions
- ✅ shareReplay(1): Single subscription shared
- ✅ Async pipe: Auto-unsubscribe
- ✅ OnPush compatible
- ✅ No manual subscriptions

**Metrics:**
- Before: 50 status emissions, 3 properties × 50 = 150 operations
- After: ~10 actual changes, shared stream
- **Improvement: 70% fewer operations**

**Advanced: Form State Service**
```typescript
@Injectable()
export class FormStateService {
  createFormState(form: FormGroup) {
    const status$ = form.statusChanges.pipe(
      startWith(form.status),
      distinctUntilChanged(),
      shareReplay(1)
    );
    
    const value$ = form.valueChanges.pipe(
      startWith(form.value),
      shareReplay(1)
    );
    
    return {
      status$,
      value$,
      isValid$: status$.pipe(map(s => s === 'VALID')),
      isInvalid$: status$.pipe(map(s => s === 'INVALID')),
      isPending$: status$.pipe(map(s => s === 'PENDING')),
      isDirty$: new BehaviorSubject(form.dirty).pipe(
        switchMap(() => merge(
          form.valueChanges.pipe(map(() => form.dirty)),
          form.statusChanges.pipe(map(() => form.dirty))
        ))
      ),
      errors$: combineLatest([
        ...Object.keys(form.controls).map(key =>
          form.get(key).statusChanges.pipe(
            startWith(form.get(key).status),
            map(() => ({ [key]: form.get(key).errors }))
          )
        )
      ]).pipe(
        map(errors => Object.assign({}, ...errors))
      )
    };
  }
}
```

---

## Large Forms Optimization

### Q16: What's wrong with this large form approach?

**Difficulty:** Hard

**Bad Code:**
```typescript
@Component({})
export class LargeFormComponent {
  // ❌ 100 fields in one flat FormGroup
  form = new FormGroup({
    field1: new FormControl('', Validators.required),
    field2: new FormControl('', Validators.required),
    field3: new FormControl('', Validators.required),
    // ... 97 more fields
    field100: new FormControl('', Validators.required)
  });
  
  ngOnInit() {
    // ❌ All validations run on ANY change
    this.form.valueChanges.subscribe(value => {
      console.log('Form changed'); // Fires 100 times for 100 field changes
      this.saveToStorage(value);
    });
  }
}
```

**Issues:**
1. ❌ All 100 validators run on any field change
2. ❌ Heavy change detection overhead
3. ❌ Poor UX (input lag)
4. ❌ Difficult to maintain
5. ❌ No lazy loading
6. ❌ Memory overhead

**Performance Impact:**
- Each keystroke = 100 validator checks
- 1000 keystrokes = 100,000 validator calls
- Result: Sluggish input, frustrated users

---

### Q17: Optimize large form with grouping:

**Difficulty:** Hard

**Optimized Code:**
```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <mat-stepper>
      <!-- Step 1: Personal Info -->
      <mat-step [formGroup]="form.get('personal')">
        <ng-template matStepLabel>Personal Info</ng-template>
        <div [formGroup]="form