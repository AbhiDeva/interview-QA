# Angular Low-Level Design (LLD)

This document contains **50 Low-Level Design concepts for Angular**, each with **intent, design notes, and small code examples**. These are useful for **interviews, real-world projects, performance tuning, and scalable UI architecture**.

---

## 1. Component Responsibility Segregation
**Intent:** One component = one responsibility.

```ts
@Component({ selector: 'app-user-name' })
export class UserNameComponent {
  @Input() name!: string;
}
```

---

## 2. Smart vs Dumb Components
**Intent:** Separate business logic from UI.

```ts
// Smart
@Component({...})
export class UserContainer {
  user$ = this.store.select(selectUser);
}
```

```ts
// Dumb
@Component({...})
export class UserView {
  @Input() user!: User;
}
```

---

## 3. Standalone Components
**Intent:** Reduce module complexity.

```ts
@Component({
  standalone: true,
  imports: [CommonModule],
  template: `Hello {{name}}`
})
export class HelloComponent {
  name = 'Angular';
}
```

---

## 4. Feature-Based Folder Structure
**Intent:** Scalability and maintainability.

```
user/
 ├── user.component.ts
 ├── user.service.ts
 ├── user.routes.ts
```

---

## 5. Service as Single Source of Truth

```ts
@Injectable({ providedIn: 'root' })
export class UserService {
  private user$ = new BehaviorSubject<User | null>(null);
}
```

---

## 6. Dependency Injection Tokens

```ts
export const API_URL = new InjectionToken<string>('API_URL');
```

---

## 7. OnPush Change Detection
**Intent:** Performance optimization.

```ts
@Component({ changeDetection: ChangeDetectionStrategy.OnPush })
```

---

## 8. Async Pipe Over Manual Subscribe

```html
<div *ngIf="user$ | async as user">{{user.name}}</div>
```

---

## 9. Facade Pattern

```ts
@Injectable()
export class UserFacade {
  user$ = this.store.select(selectUser);
}
```

---

## 10. Container-Presenter Communication via Inputs/Outputs

```ts
@Output() save = new EventEmitter<User>();
```

---

## 11. Centralized Error Handling

```ts
@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  handleError(err: any) { console.error(err); }
}
```

---

## 12. Http Interceptor – Auth Token

```ts
intercept(req, next) {
  return next.handle(req.clone({ setHeaders: { Authorization: 'Bearer token' }}));
}
```

---

## 13. Http Interceptor – Loader

```ts
finalize(() => this.loader.hide());
```

---

## 14. Route Resolver

```ts
resolve() { return this.api.getUser(); }
```

---

## 15. Route Guard (Auth)

```ts
canActivate() { return this.auth.isLoggedIn(); }
```

---

## 16. Lazy Loading Feature Modules

```ts
loadChildren: () => import('./user/user.routes')
```

---

## 17. Signals for Local State

```ts
count = signal(0);
```

---

## 18. Computed Signals

```ts
total = computed(() => this.count() * 2);
```

---

## 19. Effect for Side Effects

```ts
effect(() => console.log(this.count()));
```

---

## 20. RxJS shareReplay

```ts
this.users$ = this.http.get('/users').pipe(shareReplay(1));
```

---

## 21. Avoid Nested Subscriptions

```ts
switchMap(id => this.api.getUser(id))
```

---

## 22. takeUntil for Cleanup

```ts
this.destroy$ = new Subject<void>();
```

---

## 23. ViewModel Pattern

```ts
vm$ = combineLatest({ user: this.user$, loading: this.loading$ });
```

---

## 24. Reusable UI Components

```ts
@Component({ selector: 'ui-button' })
```

---

## 25. Structural Directive

```ts
@Directive({ selector: '[ifRole]' })
```

---

## 26. Attribute Directive

```ts
@HostBinding('class.active') isActive = true;
```

---

## 27. TrackBy in ngFor

```html
<li *ngFor="let item of items; trackBy: trackId"></li>
```

---

## 28. Memoized Selectors (NgRx)

```ts
createSelector(selectState, s => s.user)
```

---

## 29. State Normalization

```ts
entities: Record<string, User>
```

---

## 30. Immutable State Updates

```ts
return { ...state, user };
```

---

## 31. Environment Configuration

```ts
environment.apiUrl
```

---

## 32. Shared Module Anti-Pattern (Avoid)

**Use standalone imports instead**

---

## 33. Pipe for Transformation

```ts
@Pipe({ name: 'capitalize' })
```

---

## 34. Pure Pipes for Performance

```ts
@Pipe({ pure: true })
```

---

## 35. Dynamic Component Loading

```ts
viewContainerRef.createComponent(MyComp);
```

---

## 36. Content Projection

```html
<ng-content></ng-content>
```

---

## 37. Multi-Slot Projection

```html
<ng-content select="[header]"></ng-content>
```

---

## 38. Custom Form Control (ControlValueAccessor)

```ts
writeValue(val: any) {}
```

---

## 39. Reactive Forms over Template Forms

```ts
this.form = new FormGroup({ name: new FormControl('') });
```

---

## 40. Form Validation at Control Level

```ts
Validators.required
```

---

## 41. Debounced Input Handling

```ts
valueChanges.pipe(debounceTime(300))
```

---

## 42. Optimized Change Detection with Signals

```ts
readonly user = signal<User | null>(null);
```

---

## 43. Error Boundary Component

```html
<app-error *ngIf="hasError"></app-error>
```

---

## 44. Feature Toggle via Config

```ts
if (config.enableNewUI) {}
```

---

## 45. API DTO vs UI Model Separation

```ts
interface UserDto { first_name: string }
```

---

## 46. Mapper Functions

```ts
mapDtoToUser(dto: UserDto): User {}
```

---

## 47. Constants Instead of Magic Strings

```ts
export const ROLE_ADMIN = 'ADMIN';
```

---

## 48. Logging Service Abstraction

```ts
this.logger.info('User loaded');
```

---

## 49. Feature-Level Routing

```ts
export const routes: Routes = [...];
```

---

## 50. Testable Design via Dependency Injection

```ts
constructor(private api: UserApi) {}
```

---

## ✅ Usage
- Interview preparation
- Enterprise Angular apps
- Performance & scalability design
- LLD documentation reference

---

## 🏢 Real Enterprise Examples (Grid, Auth, Dashboard)

Below are **practical enterprise-grade examples** mapping LLD concepts to **real systems**.

---

### 🧩 Enterprise Data Grid (Assets / Users / Orders)

**Use cases:** Large datasets, hierarchy, selection, performance

- **Smart vs Dumb Components**  
  `GridContainerComponent` fetches data & manages state  
  `GridTableComponent` renders rows

- **OnPush + trackBy**  
  Prevents re-rendering 1000+ rows

```html
<tr *ngFor="let row of rows; trackBy: trackById"></tr>
```

- **Facade Pattern**  
  GridFacade exposes `rows$`, `selectedIds$`

- **Signals for Selection State**  
  Optimized checkbox selection logic

```ts
selectedIds = signal<Set<string>>(new Set());
```

- **Virtual Scroll + Lazy Loading**  
  Load rows page-by-page

- **DTO → UI Mapper**  
  Backend grid DTO → UI row model

---

### 🔐 Authentication & Authorization System

**Use cases:** Login, role-based access, secure APIs

- **Auth Service (Single Source of Truth)**

```ts
@Injectable({ providedIn: 'root' })
export class AuthService {
  user$ = signal<User | null>(null);
}
```

- **HTTP Interceptor (Token Injection)**

```ts
req.clone({ setHeaders: { Authorization: `Bearer ${token}` } });
```

- **Route Guard (RBAC)**

```ts
canActivate() {
  return this.auth.hasRole('ADMIN');
}
```

- **Feature Toggle**  
  Enable/disable new auth flows per environment

- **Central Error Handling**  
  Auto logout on 401 / session expiry

---

### 📊 Enterprise Dashboard (KPIs, Charts, Widgets)

**Use cases:** Real-time data, performance, modular UI

- **Widget-Based Architecture**  
  Each widget = standalone component

- **Dynamic Component Loading**

```ts
viewContainerRef.createComponent(RevenueWidget);
```

- **ViewModel Pattern**  
  Combine API + loading + error state

```ts
vm$ = combineLatest({ data, loading, error });
```

- **Signals for Real-Time Updates**

```ts
kpiValue = signal(0);
```

- **Pure Pipes**  
  Format currency, percentage efficiently

- **Lazy Loaded Dashboard Routes**  
  Faster app startup

---

### 🧠 Why This Matters in Enterprises

- Handles **large data volumes**
- Prevents **unnecessary change detection**
- Improves **security & maintainability**
- Enables **parallel team development**
- Makes apps **testable & scalable**

---

If you want next:
- 🔹 **Architecture diagrams** for Grid/Auth/Dashboard
- 🔹 **Interview questions per enterprise example**
- 🔹 **Angular 17 Signals-only enterprise patterns**
- 🔹 **PDF / printable LLD handbook**

Tell me 👍

