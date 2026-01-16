# 100 Angular Interview Questions with Code Examples

## Basics & Components

### 1. What is a Component? Create a basic component.

**Answer:**
```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-hello',
  template: `<h1>Hello, {{name}}!</h1>`,
  styles: [`h1 { color: blue; }`]
})
export class HelloComponent {
  name = 'Angular';
}
```

---

### 2. How do you use Input and Output decorators?

**Answer:**
```typescript
// Child Component
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-child',
  template: `
    <p>{{message}}</p>
    <button (click)="sendMessage()">Send to Parent</button>
  `
})
export class ChildComponent {
  @Input() message: string;
  @Output() messageEvent = new EventEmitter<string>();

  sendMessage() {
    this.messageEvent.emit('Hello from child!');
  }
}

// Parent Component
@Component({
  selector: 'app-parent',
  template: `
    <app-child 
      [message]="parentMessage" 
      (messageEvent)="receiveMessage($event)">
    </app-child>
    <p>{{childMessage}}</p>
  `
})
export class ParentComponent {
  parentMessage = 'Hello Child';
  childMessage = '';

  receiveMessage(msg: string) {
    this.childMessage = msg;
  }
}
```

---

### 3. Explain Component Lifecycle Hooks.

**Answer:**
```typescript
import { Component, OnInit, OnDestroy, OnChanges, 
         AfterViewInit, SimpleChanges } from '@angular/core';

@Component({
  selector: 'app-lifecycle',
  template: `<p>Lifecycle Demo</p>`
})
export class LifecycleComponent implements OnInit, OnDestroy, 
                                           OnChanges, AfterViewInit {
  
  ngOnChanges(changes: SimpleChanges) {
    console.log('1. OnChanges - when input properties change');
  }

  ngOnInit() {
    console.log('2. OnInit - component initialized');
  }

  ngAfterViewInit() {
    console.log('3. AfterViewInit - view initialized');
  }

  ngOnDestroy() {
    console.log('4. OnDestroy - cleanup before destruction');
  }
}
```

---

### 4. What is ViewChild and ContentChild?

**Answer:**
```typescript
import { Component, ViewChild, ContentChild, ElementRef, 
         AfterViewInit, AfterContentInit } from '@angular/core';

@Component({
  selector: 'app-child-item',
  template: `<p>Child Item</p>`
})
export class ChildItemComponent {}

@Component({
  selector: 'app-parent',
  template: `
    <div #localRef>Local Reference</div>
    <app-child-item></app-child-item>
    <ng-content></ng-content>
  `
})
export class ParentComponent implements AfterViewInit, AfterContentInit {
  @ViewChild('localRef') localRef: ElementRef;
  @ViewChild(ChildItemComponent) childItem: ChildItemComponent;
  @ContentChild('contentRef') contentRef: ElementRef;

  ngAfterViewInit() {
    console.log(this.localRef.nativeElement.textContent);
  }

  ngAfterContentInit() {
    console.log('Content projected');
  }
}
```

---

### 5. How to create a standalone component?

**Answer:**
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-standalone',
  standalone: true,
  imports: [CommonModule],
  template: `
    <h2>Standalone Component</h2>
    <p *ngIf="isVisible">Visible content</p>
  `
})
export class StandaloneComponent {
  isVisible = true;
}
```

---

## Data Binding & Directives

### 6. Demonstrate all types of data binding.

**Answer:**
```typescript
@Component({
  selector: 'app-binding',
  template: `
    <!-- Interpolation -->
    <p>{{title}}</p>
    
    <!-- Property Binding -->
    <img [src]="imageUrl" [alt]="imageAlt">
    
    <!-- Event Binding -->
    <button (click)="handleClick()">Click Me</button>
    
    <!-- Two-way Binding -->
    <input [(ngModel)]="username">
    <p>Hello, {{username}}</p>
    
    <!-- Attribute Binding -->
    <button [attr.aria-label]="buttonLabel">Action</button>
    
    <!-- Class Binding -->
    <div [class.active]="isActive">Class Binding</div>
    
    <!-- Style Binding -->
    <p [style.color]="textColor">Styled Text</p>
  `
})
export class BindingComponent {
  title = 'Data Binding';
  imageUrl = 'assets/image.jpg';
  imageAlt = 'Description';
  username = '';
  buttonLabel = 'Action Button';
  isActive = true;
  textColor = 'blue';

  handleClick() {
    console.log('Button clicked!');
  }
}
```

---

### 7. Create custom structural directive.

**Answer:**
```typescript
import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';

@Directive({
  selector: '[appUnless]'
})
export class UnlessDirective {
  @Input() set appUnless(condition: boolean) {
    if (!condition) {
      this.viewContainer.createEmbeddedView(this.templateRef);
    } else {
      this.viewContainer.clear();
    }
  }

  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef
  ) {}
}

// Usage
@Component({
  template: `
    <p *appUnless="isHidden">This is visible when isHidden is false</p>
  `
})
export class AppComponent {
  isHidden = false;
}
```

---

### 8. Create custom attribute directive.

**Answer:**
```typescript
import { Directive, ElementRef, HostListener, Input } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  @Input() appHighlight = 'yellow';
  @Input() defaultColor = 'transparent';

  constructor(private el: ElementRef) {}

  @HostListener('mouseenter') onMouseEnter() {
    this.highlight(this.appHighlight);
  }

  @HostListener('mouseleave') onMouseLeave() {
    this.highlight(this.defaultColor);
  }

  private highlight(color: string) {
    this.el.nativeElement.style.backgroundColor = color;
  }
}

// Usage
@Component({
  template: `
    <p appHighlight="lightblue">Hover over me!</p>
  `
})
export class AppComponent {}
```

---

### 9. Explain ngFor with trackBy.

**Answer:**
```typescript
@Component({
  selector: 'app-list',
  template: `
    <ul>
      <li *ngFor="let item of items; trackBy: trackByFn; let i = index">
        {{i + 1}}. {{item.name}}
      </li>
    </ul>
    <button (click)="addItem()">Add Item</button>
  `
})
export class ListComponent {
  items = [
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' },
    { id: 3, name: 'Item 3' }
  ];

  trackByFn(index: number, item: any) {
    return item.id; // Track by id for better performance
  }

  addItem() {
    const newId = this.items.length + 1;
    this.items.push({ id: newId, name: `Item ${newId}` });
  }
}
```

---

### 10. How to use ngSwitch?

**Answer:**
```typescript
@Component({
  selector: 'app-switch',
  template: `
    <select [(ngModel)]="selectedOption">
      <option value="option1">Option 1</option>
      <option value="option2">Option 2</option>
      <option value="option3">Option 3</option>
    </select>

    <div [ngSwitch]="selectedOption">
      <p *ngSwitchCase="'option1'">You selected Option 1</p>
      <p *ngSwitchCase="'option2'">You selected Option 2</p>
      <p *ngSwitchCase="'option3'">You selected Option 3</p>
      <p *ngSwitchDefault>Please select an option</p>
    </div>
  `
})
export class SwitchComponent {
  selectedOption = '';
}
```

---

## Services & Dependency Injection

### 11. Create a basic service.

**Answer:**
```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class DataService {
  private data: string[] = [];

  addData(item: string) {
    this.data.push(item);
  }

  getData() {
    return this.data;
  }

  clearData() {
    this.data = [];
  }
}

// Component usage
@Component({
  selector: 'app-data',
  template: `
    <input #input>
    <button (click)="add(input.value)">Add</button>
    <ul>
      <li *ngFor="let item of items">{{item}}</li>
    </ul>
  `
})
export class DataComponent {
  items: string[] = [];

  constructor(private dataService: DataService) {
    this.items = this.dataService.getData();
  }

  add(value: string) {
    this.dataService.addData(value);
  }
}
```

---

### 12. Explain providedIn and different injection scopes.

**Answer:**
```typescript
// Root level - Singleton across app
@Injectable({
  providedIn: 'root'
})
export class RootService {
  counter = 0;
}

// Module level
@Injectable()
export class ModuleService {
  data: any[] = [];
}

@NgModule({
  providers: [ModuleService]
})
export class FeatureModule {}

// Component level - New instance per component
@Component({
  selector: 'app-local',
  providers: [LocalService],
  template: `<p>{{service.id}}</p>`
})
export class LocalComponent {
  constructor(public service: LocalService) {}
}

@Injectable()
export class LocalService {
  id = Math.random();
}
```

---

### 13. How to use InjectionToken?

**Answer:**
```typescript
import { InjectionToken } from '@angular/core';

export const API_URL = new InjectionToken<string>('api.url');
export const APP_CONFIG = new InjectionToken<AppConfig>('app.config');

export interface AppConfig {
  apiUrl: string;
  production: boolean;
}

// In module
@NgModule({
  providers: [
    { provide: API_URL, useValue: 'https://api.example.com' },
    { 
      provide: APP_CONFIG, 
      useValue: { 
        apiUrl: 'https://api.example.com', 
        production: false 
      } 
    }
  ]
})
export class AppModule {}

// In component/service
@Component({
  selector: 'app-root',
  template: `<p>API: {{apiUrl}}</p>`
})
export class AppComponent {
  constructor(
    @Inject(API_URL) public apiUrl: string,
    @Inject(APP_CONFIG) private config: AppConfig
  ) {
    console.log(config.production);
  }
}
```

---

### 14. Implement service communication between components.

**Answer:**
```typescript
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class MessageService {
  private messageSubject = new BehaviorSubject<string>('');
  public message$: Observable<string> = this.messageSubject.asObservable();

  sendMessage(message: string) {
    this.messageSubject.next(message);
  }
}

// Component 1
@Component({
  selector: 'app-sender',
  template: `
    <input #msg>
    <button (click)="send(msg.value)">Send</button>
  `
})
export class SenderComponent {
  constructor(private messageService: MessageService) {}

  send(message: string) {
    this.messageService.sendMessage(message);
  }
}

// Component 2
@Component({
  selector: 'app-receiver',
  template: `<p>Received: {{message}}</p>`
})
export class ReceiverComponent implements OnInit {
  message = '';

  constructor(private messageService: MessageService) {}

  ngOnInit() {
    this.messageService.message$.subscribe(msg => {
      this.message = msg;
    });
  }
}
```

---

### 15. How to create a factory provider?

**Answer:**
```typescript
import { Injectable, InjectionToken } from '@angular/core';

export const USE_FAKE_SERVICE = new InjectionToken<boolean>('use.fake.service');

export interface UserService {
  getUsers(): string[];
}

@Injectable()
export class RealUserService implements UserService {
  getUsers() {
    return ['Real User 1', 'Real User 2'];
  }
}

@Injectable()
export class FakeUserService implements UserService {
  getUsers() {
    return ['Fake User 1', 'Fake User 2'];
  }
}

export function userServiceFactory(useFake: boolean): UserService {
  return useFake ? new FakeUserService() : new RealUserService();
}

// In module
@NgModule({
  providers: [
    { provide: USE_FAKE_SERVICE, useValue: false },
    {
      provide: UserService,
      useFactory: userServiceFactory,
      deps: [USE_FAKE_SERVICE]
    }
  ]
})
export class AppModule {}
```

---

## RxJS & Observables

### 16. Create an Observable from scratch.

**Answer:**
```typescript
import { Observable } from 'rxjs';

@Component({
  selector: 'app-observable',
  template: `<p>Value: {{value}}</p>`
})
export class ObservableComponent implements OnInit {
  value = 0;

  ngOnInit() {
    const observable = new Observable<number>(subscriber => {
      let count = 0;
      const interval = setInterval(() => {
        subscriber.next(count++);
        if (count > 5) {
          subscriber.complete();
          clearInterval(interval);
        }
      }, 1000);

      // Cleanup function
      return () => {
        clearInterval(interval);
      };
    });

    observable.subscribe({
      next: (value) => this.value = value,
      complete: () => console.log('Complete!')
    });
  }
}
```

---

### 17. Demonstrate common RxJS operators.

**Answer:**
```typescript
import { Component, OnInit } from '@angular/core';
import { of, interval } from 'rxjs';
import { map, filter, take, debounceTime, distinctUntilChanged, 
         switchMap, mergeMap, catchError } from 'rxjs/operators';

@Component({
  selector: 'app-rxjs',
  template: `<div>Check console for results</div>`
})
export class RxjsComponent implements OnInit {
  ngOnInit() {
    // Map
    of(1, 2, 3, 4, 5)
      .pipe(map(x => x * 2))
      .subscribe(console.log); // 2, 4, 6, 8, 10

    // Filter
    of(1, 2, 3, 4, 5)
      .pipe(filter(x => x % 2 === 0))
      .subscribe(console.log); // 2, 4

    // Take
    interval(1000)
      .pipe(take(3))
      .subscribe(console.log); // 0, 1, 2

    // Chaining operators
    of(1, 2, 3, 4, 5)
      .pipe(
        filter(x => x > 2),
        map(x => x * x),
        take(2)
      )
      .subscribe(console.log); // 9, 16
  }
}
```

---

### 18. Implement search with debounce.

**Answer:**
```typescript
import { Component, OnInit } from '@angular/core';
import { FormControl } from '@angular/forms';
import { debounceTime, distinctUntilChanged, switchMap } from 'rxjs/operators';
import { of } from 'rxjs';

@Component({
  selector: 'app-search',
  template: `
    <input [formControl]="searchControl" placeholder="Search...">
    <ul>
      <li *ngFor="let result of results">{{result}}</li>
    </ul>
  `
})
export class SearchComponent implements OnInit {
  searchControl = new FormControl('');
  results: string[] = [];

  ngOnInit() {
    this.searchControl.valueChanges
      .pipe(
        debounceTime(300),
        distinctUntilChanged(),
        switchMap(term => this.search(term))
      )
      .subscribe(results => {
        this.results = results;
      });
  }

  search(term: string) {
    // Simulate API call
    const mockData = ['Apple', 'Banana', 'Orange', 'Mango'];
    return of(mockData.filter(item => 
      item.toLowerCase().includes(term.toLowerCase())
    ));
  }
}
```

---

### 19. Explain Subject, BehaviorSubject, and ReplaySubject.

**Answer:**
```typescript
import { Subject, BehaviorSubject, ReplaySubject } from 'rxjs';

@Component({
  selector: 'app-subjects',
  template: `<div>Check console</div>`
})
export class SubjectsComponent implements OnInit {
  ngOnInit() {
    // Subject - no initial value, no replay
    const subject = new Subject<number>();
    subject.subscribe(val => console.log('Sub A:', val));
    subject.next(1);
    subject.subscribe(val => console.log('Sub B:', val)); // Won't get 1
    subject.next(2); // Both get 2

    // BehaviorSubject - has initial value, replays last value
    const behaviorSubject = new BehaviorSubject<number>(0);
    behaviorSubject.subscribe(val => console.log('Behavior A:', val)); // Gets 0
    behaviorSubject.next(1);
    behaviorSubject.subscribe(val => console.log('Behavior B:', val)); // Gets 1
    behaviorSubject.next(2); // Both get 2

    // ReplaySubject - replays specified number of values
    const replaySubject = new ReplaySubject<number>(2); // Replay last 2
    replaySubject.next(1);
    replaySubject.next(2);
    replaySubject.next(3);
    replaySubject.subscribe(val => console.log('Replay:', val)); // Gets 2, 3
  }
}
```

---

### 20. How to handle errors in Observables?

**Answer:**
```typescript
import { Component, OnInit } from '@angular/core';
import { throwError, of } from 'rxjs';
import { catchError, retry, finalize } from 'rxjs/operators';

@Component({
  selector: 'app-error-handling',
  template: `
    <p>{{message}}</p>
    <p *ngIf="loading">Loading...</p>
  `
})
export class ErrorHandlingComponent implements OnInit {
  message = '';
  loading = false;

  ngOnInit() {
    this.loading = true;

    this.makeRequest()
      .pipe(
        retry(3), // Retry 3 times
        catchError(error => {
          console.error('Error:', error);
          return of('Default value'); // Return fallback
        }),
        finalize(() => this.loading = false)
      )
      .subscribe(
        data => this.message = data,
        error => this.message = 'Error occurred'
      );
  }

  makeRequest() {
    // Simulate API call that might fail
    return Math.random() > 0.5 
      ? of('Success!') 
      : throwError('API Error');
  }
}
```

---

## Forms

### 21. Create a template-driven form.

**Answer:**
```typescript
import { Component } from '@angular/core';
import { NgForm } from '@angular/forms';

@Component({
  selector: 'app-template-form',
  template: `
    <form #userForm="ngForm" (ngSubmit)="onSubmit(userForm)">
      <div>
        <label>Name:</label>
        <input 
          name="name" 
          ngModel 
          required 
          #name="ngModel">
        <span *ngIf="name.invalid && name.touched">Name is required</span>
      </div>

      <div>
        <label>Email:</label>
        <input 
          name="email" 
          ngModel 
          required 
          email 
          #email="ngModel">
        <span *ngIf="email.invalid && email.touched">
          Valid email is required
        </span>
      </div>

      <button [disabled]="userForm.invalid">Submit</button>
    </form>

    <pre>{{submittedData | json}}</pre>
  `
})
export class TemplateFormComponent {
  submittedData: any;

  onSubmit(form: NgForm) {
    if (form.valid) {
      this.submittedData = form.value;
      form.reset();
    }
  }
}
```

---

### 22. Create a reactive form.

**Answer:**
```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-reactive-form',
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <div>
        <label>Name:</label>
        <input formControlName="name">
        <span *ngIf="name.invalid && name.touched">
          Name is required (min 3 chars)
        </span>
      </div>

      <div>
        <label>Email:</label>
        <input formControlName="email">
        <span *ngIf="email.invalid && email.touched">
          Valid email is required
        </span>
      </div>

      <div>
        <label>Age:</label>
        <input formControlName="age" type="number">
        <span *ngIf="age.invalid && age.touched">
          Age must be between 18 and 100
        </span>
      </div>

      <button [disabled]="userForm.invalid">Submit</button>
    </form>

    <pre>{{submittedData | json}}</pre>
  `
})
export class ReactiveFormComponent implements OnInit {
  userForm: FormGroup;
  submittedData: any;

  constructor(private fb: FormBuilder) {}

  ngOnInit() {
    this.userForm = this.fb.group({
      name: ['', [Validators.required, Validators.minLength(3)]],
      email: ['', [Validators.required, Validators.email]],
      age: ['', [Validators.required, Validators.min(18), Validators.max(100)]]
    });
  }

  get name() { return this.userForm.get('name'); }
  get email() { return this.userForm.get('email'); }
  get age() { return this.userForm.get('age'); }

  onSubmit() {
    if (this.userForm.valid) {
      this.submittedData = this.userForm.value;
      this.userForm.reset();
    }
  }
}
```

---

### 23. Create a custom validator.

**Answer:**
```typescript
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

// Custom Validator Function
export function forbiddenNameValidator(forbiddenName: string): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const forbidden = control.value?.toLowerCase() === forbiddenName.toLowerCase();
    return forbidden ? { forbiddenName: { value: control.value } } : null;
  };
}

// Async Validator (e.g., check username availability)
export function usernameAsyncValidator(service: any): ValidatorFn {
  return (control: AbstractControl): Promise<ValidationErrors | null> => {
    return new Promise((resolve) => {
      setTimeout(() => {
        const taken = control.value === 'admin';
        resolve(taken ? { usernameTaken: true } : null);
      }, 1000);
    });
  };
}

// Usage
@Component({
  selector: 'app-custom-validation',
  template: `
    <form [formGroup]="form">
      <input formControlName="username">
      <span *ngIf="username.hasError('forbiddenName')">
        This name is forbidden
      </span>
      <span *ngIf="username.hasError('usernameTaken')">
        Username is already taken
      </span>
      <span *ngIf="username.pending">Checking...</span>
    </form>
  `
})
export class CustomValidationComponent implements OnInit {
  form: FormGroup;

  constructor(private fb: FormBuilder) {}

  ngOnInit() {
    this.form = this.fb.group({
      username: ['', 
        [Validators.required, forbiddenNameValidator('admin')],
        [usernameAsyncValidator(null)]
      ]
    });
  }

  get username() { return this.form.get('username'); }
}
```

---

### 24. Create a FormArray for dynamic form fields.

**Answer:**
```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, FormArray, Validators } from '@angular/forms';

@Component({
  selector: 'app-form-array',
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <div formArrayName="skills">
        <div *ngFor="let skill of skills.controls; let i = index" 
             [formGroupName]="i">
          <input formControlName="name" placeholder="Skill name">
          <input formControlName="years" type="number" placeholder="Years">
          <button type="button" (click)="removeSkill(i)">Remove</button>
        </div>
      </div>
      
      <button type="button" (click)="addSkill()">Add Skill</button>
      <button type="submit" [disabled]="form.invalid">Submit</button>
    </form>

    <pre>{{form.value | json}}</pre>
  `
})
export class FormArrayComponent implements OnInit {
  form: FormGroup;

  constructor(private fb: FormBuilder) {}

  ngOnInit() {
    this.form = this.fb.group({
      skills: this.fb.array([])
    });

    // Add initial skill
    this.addSkill();
  }

  get skills() {
    return this.form.get('skills') as FormArray;
  }

  createSkill(): FormGroup {
    return this.fb.group({
      name: ['', Validators.required],
      years: ['', [Validators.required, Validators.min(0)]]
    });
  }

  addSkill() {
    this.skills.push(this.createSkill());
  }

  removeSkill(index: number) {
    this.skills.removeAt(index);
  }

  onSubmit() {
    console.log(this.form.value);
  }
}
```

---

### 25. Implement value changes and status changes.

**Answer:**
```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup } from '@angular/forms';

@Component({
  selector: 'app-form-changes',
  template: `
    <form [formGroup]="form">
      <input formControlName="search" placeholder="Search">
      <p>Status: {{formStatus}}</p>
      <p>Search value: {{searchValue}}</p>
    </form>
  `
})
export class FormChangesComponent implements OnInit {
  form: FormGroup;
  formStatus = '';
  searchValue = '';

  constructor(private fb: FormBuilder) {}

  ngOnInit() {
    this.form = this.fb.group({
      search: ['']
    });

    // Listen to value changes
    this.form.get('search').valueChanges
      .pipe(debounceTime(300))
      .subscribe(value => {
        this.searchValue = value;
        console.log('Search value changed:', value);
      });

    // Listen to status changes
    this.form.statusChanges.subscribe(status => {
      this.formStatus = status;
      console.log('Form status:', status);
    });

    // Listen to entire form value changes
    this.form.valueChanges.subscribe(value => {
      console.log('Form value:', value);
    });
  }
}
```

---

## HTTP & API Calls

### 26. Make HTTP GET request.

**Answer:**
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class CrudService {
  private apiUrl = 'https://api.example.com/items';

  constructor(private http: HttpClient) {}

  // POST
  createItem(item: any): Observable<any> {
    return this.http.post(this.apiUrl, item);
  }

  // PUT
  updateItem(id: number, item: any): Observable<any> {
    return this.http.put(`${this.apiUrl}/${id}`, item);
  }

  // PATCH
  patchItem(id: number, item: Partial<any>): Observable<any> {
    return this.http.patch(`${this.apiUrl}/${id}`, item);
  }

  // DELETE
  deleteItem(id: number): Observable<any> {
    return this.http.delete(`${this.apiUrl}/${id}`);
  }
}

// Component usage
@Component({
  selector: 'app-crud',
  template: `
    <button (click)="create()">Create</button>
    <button (click)="update()">Update</button>
    <button (click)="delete()">Delete</button>
  `
})
export class CrudComponent {
  constructor(private crudService: CrudService) {}

  create() {
    const newItem = { name: 'New Item', description: 'Description' };
    this.crudService.createItem(newItem).subscribe(
      response => console.log('Created:', response),
      error => console.error('Error:', error)
    );
  }

  update() {
    const updatedItem = { name: 'Updated Item' };
    this.crudService.updateItem(1, updatedItem).subscribe(
      response => console.log('Updated:', response)
    );
  }

  delete() {
    this.crudService.deleteItem(1).subscribe(
      response => console.log('Deleted:', response)
    );
  }
}
```

---

### 28. Create HTTP Interceptor for authentication.

**Answer:**
```typescript
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpRequest, HttpHandler, 
         HttpEvent, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Get token from storage
    const token = localStorage.getItem('token');

    // Clone request and add authorization header
    if (token) {
      req = req.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`
        }
      });
    }

    return next.handle(req).pipe(
      catchError((error: HttpErrorResponse) => {
        if (error.status === 401) {
          // Handle unauthorized error
          console.log('Unauthorized - redirect to login');
        }
        return throwError(error);
      })
    );
  }
}

// Register in module
@NgModule({
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptor,
      multi: true
    }
  ]
})
export class AppModule {}
```

---

### 29. Create logging interceptor.

**Answer:**
```typescript
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpRequest, HttpHandler, 
         HttpEvent, HttpResponse } from '@angular/common/http';
import { Observable } from 'rxjs';
import { tap, finalize } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements HttpInterceptor {
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const started = Date.now();
    let status: string;

    return next.handle(req).pipe(
      tap(
        event => {
          if (event instanceof HttpResponse) {
            status = 'succeeded';
          }
        },
        error => status = 'failed'
      ),
      finalize(() => {
        const elapsed = Date.now() - started;
        console.log(`${req.method} ${req.urlWithParams} ${status} in ${elapsed}ms`);
      })
    );
  }
}
```

---

### 30. Handle HTTP errors globally.

**Answer:**
```typescript
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpRequest, HttpHandler, 
         HttpEvent, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError, retry } from 'rxjs/operators';

@Injectable()
export class ErrorInterceptor implements HttpInterceptor {
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      retry(2), // Retry failed request up to 2 times
      catchError((error: HttpErrorResponse) => {
        let errorMessage = '';

        if (error.error instanceof ErrorEvent) {
          // Client-side error
          errorMessage = `Client Error: ${error.error.message}`;
        } else {
          // Server-side error
          errorMessage = `Server Error: ${error.status} - ${error.message}`;
        }

        // Log to console or send to logging service
        console.error(errorMessage);

        // Show user-friendly message
        this.showErrorToUser(error.status);

        return throwError(errorMessage);
      })
    );
  }

  private showErrorToUser(status: number) {
    switch (status) {
      case 400:
        alert('Bad Request');
        break;
      case 401:
        alert('Unauthorized - Please login');
        break;
      case 404:
        alert('Resource not found');
        break;
      case 500:
        alert('Server error - Please try again later');
        break;
      default:
        alert('An error occurred');
    }
  }
}
```

---

### 31. Use HttpParams for query parameters.

**Answer:**
```typescript
import { HttpClient, HttpParams } from '@angular/common/http';
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class SearchService {
  private apiUrl = 'https://api.example.com/search';

  constructor(private http: HttpClient) {}

  search(query: string, page: number, limit: number) {
    // Method 1: Using HttpParams
    let params = new HttpParams()
      .set('q', query)
      .set('page', page.toString())
      .set('limit', limit.toString());

    return this.http.get(this.apiUrl, { params });
  }

  searchWithObject(filters: any) {
    // Method 2: Using object
    const params = {
      search: filters.search || '',
      category: filters.category || '',
      sort: filters.sort || 'asc'
    };

    return this.http.get(this.apiUrl, { params });
  }

  searchWithBuilder(term: string, options: any) {
    // Method 3: Building params conditionally
    let params = new HttpParams().set('q', term);

    if (options.category) {
      params = params.set('category', options.category);
    }

    if (options.minPrice) {
      params = params.set('minPrice', options.minPrice);
    }

    if (options.maxPrice) {
      params = params.set('maxPrice', options.maxPrice);
    }

    return this.http.get(this.apiUrl, { params });
  }
}
```

---

## Routing & Navigation

### 32. Set up basic routing.

**Answer:**
```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: '', redirectTo: '/home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: 'contact', component: ContactComponent },
  { path: '**', component: NotFoundComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}

// App Component
@Component({
  selector: 'app-root',
  template: `
    <nav>
      <a routerLink="/home" routerLinkActive="active">Home</a>
      <a routerLink="/about" routerLinkActive="active">About</a>
      <a routerLink="/contact" routerLinkActive="active">Contact</a>
    </nav>
    <router-outlet></router-outlet>
  `,
  styles: [`
    .active { font-weight: bold; color: blue; }
  `]
})
export class AppComponent {}
```

---

### 33. Implement route parameters and query params.

**Answer:**
```typescript
// Routes with parameters
const routes: Routes = [
  { path: 'user/:id', component: UserDetailComponent },
  { path: 'product/:id/:slug', component: ProductComponent }
];

// Reading route params
@Component({
  selector: 'app-user-detail',
  template: `<p>User ID: {{userId}}</p>`
})
export class UserDetailComponent implements OnInit {
  userId: string;

  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    // Method 1: Snapshot (non-reactive)
    this.userId = this.route.snapshot.paramMap.get('id');

    // Method 2: Observable (reactive)
    this.route.paramMap.subscribe(params => {
      this.userId = params.get('id');
    });
  }
}

// Navigating with params
@Component({
  selector: 'app-user-list',
  template: `
    <button (click)="viewUser(123)">View User</button>
    <button (click)="searchProducts()">Search</button>
  `
})
export class UserListComponent {
  constructor(private router: Router) {}

  viewUser(id: number) {
    // Navigate with params
    this.router.navigate(['/user', id]);
  }

  searchProducts() {
    // Navigate with query params
    this.router.navigate(['/products'], {
      queryParams: { 
        search: 'laptop', 
        category: 'electronics',
        page: 1 
      }
    });
  }
}

// Reading query params
@Component({
  selector: 'app-products',
  template: `<p>Search: {{searchTerm}}</p>`
})
export class ProductsComponent implements OnInit {
  searchTerm: string;

  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    this.route.queryParamMap.subscribe(params => {
      this.searchTerm = params.get('search');
    });
  }
}
```

---

### 34. Create child routes and nested routing.

**Answer:**
```typescript
const routes: Routes = [
  {
    path: 'admin',
    component: AdminComponent,
    children: [
      { path: '', redirectTo: 'dashboard', pathMatch: 'full' },
      { path: 'dashboard', component: DashboardComponent },
      { path: 'users', component: UsersComponent },
      { path: 'settings', component: SettingsComponent }
    ]
  }
];

// Admin Component with nested router-outlet
@Component({
  selector: 'app-admin',
  template: `
    <div class="admin-layout">
      <nav>
        <a routerLink="dashboard" routerLinkActive="active">Dashboard</a>
        <a routerLink="users" routerLinkActive="active">Users</a>
        <a routerLink="settings" routerLinkActive="active">Settings</a>
      </nav>
      <div class="content">
        <router-outlet></router-outlet>
      </div>
    </div>
  `
})
export class AdminComponent {}
```

---

### 35. Implement route guards.

**Answer:**
```typescript
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot, 
         Router, UrlTree } from '@angular/router';
import { Observable } from 'rxjs';

// Auth Guard
@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
  constructor(private router: Router) {}

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): boolean | UrlTree {
    const isAuthenticated = !!localStorage.getItem('token');

    if (isAuthenticated) {
      return true;
    }

    // Redirect to login
    return this.router.createUrlTree(['/login'], {
      queryParams: { returnUrl: state.url }
    });
  }
}

// CanDeactivate Guard
export interface CanComponentDeactivate {
  canDeactivate: () => Observable<boolean> | Promise<boolean> | boolean;
}

@Injectable({
  providedIn: 'root'
})
export class CanDeactivateGuard implements CanDeactivate<CanComponentDeactivate> {
  canDeactivate(
    component: CanComponentDeactivate
  ): Observable<boolean> | Promise<boolean> | boolean {
    return component.canDeactivate ? component.canDeactivate() : true;
  }
}

// Usage in routes
const routes: Routes = [
  { 
    path: 'admin', 
    component: AdminComponent,
    canActivate: [AuthGuard]
  },
  {
    path: 'edit',
    component: EditComponent,
    canDeactivate: [CanDeactivateGuard]
  }
];

// Component with canDeactivate
@Component({
  selector: 'app-edit',
  template: `<form>...</form>`
})
export class EditComponent implements CanComponentDeactivate {
  hasUnsavedChanges = true;

  canDeactivate(): boolean {
    if (this.hasUnsavedChanges) {
      return confirm('You have unsaved changes. Do you really want to leave?');
    }
    return true;
  }
}
```

---

### 36. Implement lazy loading.

**Answer:**
```typescript
// App routing module
const routes: Routes = [
  { path: '', component: HomeComponent },
  { 
    path: 'admin', 
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule),
    canActivate: [AuthGuard]
  },
  {
    path: 'products',
    loadChildren: () => import('./products/products.module').then(m => m.ProductsModule)
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}

// Feature module (admin.module.ts)
const adminRoutes: Routes = [
  {
    path: '',
    component: AdminLayoutComponent,
    children: [
      { path: 'dashboard', component: DashboardComponent },
      { path: 'users', component: UsersComponent }
    ]
  }
];

@NgModule({
  declarations: [AdminLayoutComponent, DashboardComponent, UsersComponent],
  imports: [
    CommonModule,
    RouterModule.forChild(adminRoutes)
  ]
})
export class AdminModule {}
```

---

### 37. Implement preloading strategies.

**Answer:**
```typescript
import { PreloadAllModules, PreloadingStrategy, Route } from '@angular/router';
import { Observable, of, timer } from 'rxjs';
import { flatMap } from 'rxjs/operators';

// Use built-in PreloadAllModules
@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      preloadingStrategy: PreloadAllModules
    })
  ]
})
export class AppRoutingModule {}

// Custom preloading strategy
@Injectable({
  providedIn: 'root'
})
export class CustomPreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    // Only preload routes with data: { preload: true }
    if (route.data && route.data['preload']) {
      console.log('Preloading:', route.path);
      return load();
    }
    return of(null);
  }
}

// Network-aware preloading
@Injectable({
  providedIn: 'root'
})
export class NetworkAwarePreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    const connection = (navigator as any).connection;
    
    if (connection) {
      // Only preload on fast connections
      if (connection.effectiveType === '4g') {
        return timer(3000).pipe(flatMap(() => load()));
      }
    }
    
    return of(null);
  }
}

// Usage
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule),
    data: { preload: true }
  }
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      preloadingStrategy: CustomPreloadingStrategy
    })
  ]
})
export class AppRoutingModule {}
```

---

### 38. Programmatic navigation with Router.

**Answer:**
```typescript
import { Router, NavigationExtras } from '@angular/router';

@Component({
  selector: 'app-navigation',
  template: `
    <button (click)="navigateSimple()">Simple Navigate</button>
    <button (click)="navigateWithParams()">With Params</button>
    <button (click)="navigateWithQuery()">With Query</button>
    <button (click)="navigateRelative()">Relative Navigate</button>
    <button (click)="goBack()">Go Back</button>
  `
})
export class NavigationComponent {
  constructor(
    private router: Router,
    private route: ActivatedRoute
  ) {}

  navigateSimple() {
    this.router.navigate(['/products']);
  }

  navigateWithParams() {
    this.router.navigate(['/product', 123]);
  }

  navigateWithQuery() {
    const navigationExtras: NavigationExtras = {
      queryParams: { page: 1, sort: 'name' },
      fragment: 'section1'
    };
    this.router.navigate(['/products'], navigationExtras);
  }

  navigateRelative() {
    // Navigate relative to current route
    this.router.navigate(['../sibling'], { relativeTo: this.route });
  }

  goBack() {
    window.history.back();
  }

  navigateWithState() {
    this.router.navigate(['/details'], {
      state: { data: { id: 1, name: 'Test' } }
    });
  }
}

// Reading navigation state
@Component({
  selector: 'app-details',
  template: `<p>{{data | json}}</p>`
})
export class DetailsComponent {
  data: any;

  constructor(private router: Router) {
    const navigation = this.router.getCurrentNavigation();
    this.data = navigation?.extras?.state?.['data'];
  }
}
```

---

## Pipes

### 39. Create a custom pipe.

**Answer:**
```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'exponential'
})
export class ExponentialPipe implements PipeTransform {
  transform(value: number, exponent: number = 1): number {
    return Math.pow(value, exponent);
  }
}

// Usage
@Component({
  template: `
    <p>{{2 | exponential}}</p>        <!-- 2 -->
    <p>{{2 | exponential:3}}</p>      <!-- 8 -->
    <p>{{5 | exponential:2}}</p>      <!-- 25 -->
  `
})
export class AppComponent {}
```

---

### 40. Create a filter pipe.

**Answer:**
```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'filter',
  pure: false // Make it impure to detect changes in arrays
})
export class FilterPipe implements PipeTransform {
  transform(items: any[], searchText: string, property: string): any[] {
    if (!items || !searchText) {
      return items;
    }

    searchText = searchText.toLowerCase();

    return items.filter(item => {
      if (property) {
        return item[property].toLowerCase().includes(searchText);
      }
      return JSON.stringify(item).toLowerCase().includes(searchText);
    });
  }
}

// Usage
@Component({
  template: `
    <input [(ngModel)]="searchTerm" placeholder="Search">
    <ul>
      <li *ngFor="let user of users | filter:searchTerm:'name'">
        {{user.name}}
      </li>
    </ul>
  `
})
export class UserListComponent {
  searchTerm = '';
  users = [
    { name: 'John', email: 'john@example.com' },
    { name: 'Jane', email: 'jane@example.com' },
    { name: 'Bob', email: 'bob@example.com' }
  ];
}
```

---

### 41. Create an async pipe example with observables.

**Answer:**
```typescript
import { Component } from '@angular/core';
import { Observable, interval } from 'rxjs';
import { map, take } from 'rxjs/operators';

@Component({
  selector: 'app-async-pipe',
  template: `
    <!-- Async pipe with Observable -->
    <p>Time: {{time$ | async}}</p>

    <!-- Async pipe with Promise -->
    <p>Message: {{message$ | async}}</p>

    <!-- Multiple subscriptions (not recommended) -->
    <p>Count: {{count$ | async}}</p>
    <p>Double: {{(count$ | async) * 2}}</p>

    <!-- Better: Use as to avoid multiple subscriptions -->
    <ng-container *ngIf="count$ | async as count">
      <p>Count: {{count}}</p>
      <p>Double: {{count * 2}}</p>
      <p>Triple: {{count * 3}}</p>
    </ng-container>

    <!-- With loading and error states -->
    <div *ngIf="users$ | async as users; else loading">
      <div *ngFor="let user of users">{{user.name}}</div>
    </div>
    <ng-template #loading>Loading...</ng-template>
  `
})
export class AsyncPipeComponent {
  time$: Observable<Date>;
  message$: Promise<string>;
  count$: Observable<number>;
  users$: Observable<any[]>;

  constructor() {
    this.time$ = interval(1000).pipe(
      map(() => new Date())
    );

    this.message$ = new Promise(resolve => {
      setTimeout(() => resolve('Hello from Promise!'), 2000);
    });

    this.count$ = interval(1000).pipe(take(10));

    this.users$ = of([
      { name: 'User 1' },
      { name: 'User 2' }
    ]);
  }
}
```

---

### 42. Create a sort pipe.

**Answer:**
```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'sort',
  pure: false
})
export class SortPipe implements PipeTransform {
  transform(array: any[], field: string, order: 'asc' | 'desc' = 'asc'): any[] {
    if (!array || !field) {
      return array;
    }

    const sorted = [...array].sort((a, b) => {
      const aValue = a[field];
      const bValue = b[field];

      if (aValue < bValue) {
        return order === 'asc' ? -1 : 1;
      }
      if (aValue > bValue) {
        return order === 'asc' ? 1 : -1;
      }
      return 0;
    });

    return sorted;
  }
}

// Usage
@Component({
  template: `
    <button (click)="toggleSort()">
      Sort by Name ({{sortOrder}})
    </button>
    <ul>
      <li *ngFor="let item of items | sort:'name':sortOrder">
        {{item.name}} - {{item.price}}
      </li>
    </ul>
  `
})
export class SortComponent {
  sortOrder: 'asc' | 'desc' = 'asc';
  
  items = [
    { name: 'Product C', price: 30 },
    { name: 'Product A', price: 10 },
    { name: 'Product B', price: 20 }
  ];

  toggleSort() {
    this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
  }
}
```

---

### 43. Create a safe HTML pipe.

**Answer:**
```typescript
import { Pipe, PipeTransform } from '@angular/core';
import { DomSanitizer, SafeHtml } from '@angular/platform-browser';

@Pipe({
  name: 'safeHtml'
})
export class SafeHtmlPipe implements PipeTransform {
  constructor(private sanitizer: DomSanitizer) {}

  transform(value: string): SafeHtml {
    return this.sanitizer.bypassSecurityTrustHtml(value);
  }
}

// Usage
@Component({
  template: `
    <div [innerHTML]="htmlContent | safeHtml"></div>
  `
})
export class SafeHtmlComponent {
  htmlContent = '<h1 style="color: red;">This is HTML content</h1><p>With <strong>formatting</strong></p>';
}
```

---

## State Management

### 44. Implement a simple state service.

**Answer:**
```typescript
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';

export interface AppState {
  user: any;
  cart: any[];
  loading: boolean;
}

@Injectable({
  providedIn: 'root'
})
export class StateService {
  private initialState: AppState = {
    user: null,
    cart: [],
    loading: false
  };

  private state$ = new BehaviorSubject<AppState>(this.initialState);

  // Selectors
  get state(): AppState {
    return this.state$.value;
  }

  getState$(): Observable<AppState> {
    return this.state$.asObservable();
  }

  getUser$(): Observable<any> {
    return this.state$.pipe(
      map(state => state.user)
    );
  }

  getCart$(): Observable<any[]> {
    return this.state$.pipe(
      map(state => state.cart)
    );
  }

  // Actions
  setUser(user: any) {
    this.setState({ user });
  }

  addToCart(item: any) {
    const cart = [...this.state.cart, item];
    this.setState({ cart });
  }

  removeFromCart(itemId: number) {
    const cart = this.state.cart.filter(item => item.id !== itemId);
    this.setState({ cart });
  }

  setLoading(loading: boolean) {
    this.setState({ loading });
  }

  private setState(partialState: Partial<AppState>) {
    this.state$.next({
      ...this.state,
      ...partialState
    });
  }

  reset() {
    this.state$.next(this.initialState);
  }
}

// Component usage
@Component({
  selector: 'app-cart',
  template: `
    <div *ngIf="cart$ | async as cart">
      <p>Items: {{cart.length}}</p>
      <div *ngFor="let item of cart">
        {{item.name}}
        <button (click)="remove(item.id)">Remove</button>
      </div>
    </div>
  `
})
export class CartComponent implements OnInit {
  cart$: Observable<any[]>;

  constructor(private stateService: StateService) {}

  ngOnInit() {
    this.cart$ = this.stateService.getCart$();
  }

  remove(id: number) {
    this.stateService.removeFromCart(id);
  }
}
```

---

### 45. Implement NgRx basic setup.

**Answer:**
```typescript
// actions/counter.actions.ts
import { createAction, props } from '@ngrx/store';

export const increment = createAction('[Counter] Increment');
export const decrement = createAction('[Counter] Decrement');
export const reset = createAction('[Counter] Reset');
export const set = createAction(
  '[Counter] Set',
  props<{ value: number }>()
);

// reducers/counter.reducer.ts
import { createReducer, on } from '@ngrx/store';
import * as CounterActions from '../actions/counter.actions';

export interface CounterState {
  count: number;
}

export const initialState: CounterState = {
  count: 0
};

export const counterReducer = createReducer(
  initialState,
  on(CounterActions.increment, state => ({ count: state.count + 1 })),
  on(CounterActions.decrement, state => ({ count: state.count - 1 })),
  on(CounterActions.reset, state => ({ count: 0 })),
  on(CounterActions.set, (state, { value }) => ({ count: value }))
);

// selectors/counter.selectors.ts
import { createSelector, createFeatureSelector } from '@ngrx/store';
import { CounterState } from '../reducers/counter.reducer';

export const selectCounterState = createFeatureSelector<CounterState>('counter');

export const selectCount = createSelector(
  selectCounterState,
  (state: CounterState) => state.count
);

export const selectCountDoubled = createSelector(
  selectCount,
  (count: number) => count * 2
);

// app.module.ts
import { StoreModule } from '@ngrx/store';
import { counterReducer } from './store/reducers/counter.reducer';

@NgModule({
  imports: [
    StoreModule.forRoot({ counter: counterReducer })
  ]
})
export class AppModule {}

// Component
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import * as CounterActions from './store/actions/counter.actions';
import { selectCount } from './store/selectors/counter.selectors';

@Component({
  selector: 'app-counter',
  template: `
    <p>Count: {{count$ | async}}</p>
    <button (click)="increment()">+</button>
    <button (click)="decrement()">-</button>
    <button (click)="reset()">Reset</button>
  `
})
export class CounterComponent {
  count$ '@angular/common/http';
import { Observable } from 'rxjs';

export interface User {
  id: number;
  name: string;
  email: string;
}

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private apiUrl = 'https://api.example.com/users';

  constructor(private http: HttpClient) {}

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }

  getUserById(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }
}

// Component
@Component({
  selector: 'app-users',
  template: `
    <ul>
      <li *ngFor="let user of users">{{user.name}}</li>
    </ul>
  `
})
export class UsersComponent implements OnInit {
  users: User[] = [];

  constructor(private userService: UserService) {}

  ngOnInit() {
    this.userService.getUsers().subscribe(
      data => this.users = data,
      error => console.error('Error:', error)
    );
  }
}
```

---

### 27. Make HTTP POST, PUT, DELETE requests.

**Answer:**
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from