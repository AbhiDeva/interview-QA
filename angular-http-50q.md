# Angular HTTP - 50 Interview Questions & Code Examples

## Table of Contents
1. [HTTP Basics (Q1-Q10)](#http-basics)
2. [HTTP Interceptors (Q11-Q15)](#http-interceptors)
3. [Error Handling (Q16-Q20)](#error-handling)
4. [Request/Response Transformation (Q21-Q25)](#requestresponse-transformation)
5. [Authentication & Headers (Q26-Q30)](#authentication--headers)
6. [Observable Patterns (Q31-Q35)](#observable-patterns)
7. [Performance & Caching (Q36-Q40)](#performance--caching)
8. [File Upload/Download (Q41-Q45)](#file-uploaddownload)
9. [Advanced Patterns (Q46-Q50)](#advanced-patterns)

---

## HTTP Basics

### Q1: How to set up HttpClient in Angular?

**Difficulty:** Easy

**Code:**
```typescript
// app.config.ts (Standalone - Angular 14+)
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient, withInterceptors } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([/* interceptors */])
    )
  ]
};

// app.module.ts (Module-based)
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  imports: [
    BrowserModule,
    HttpClientModule  // Import HttpClientModule
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent]
})
export class AppModule {}

// Basic usage in service
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class UserService {
  private apiUrl = 'https://api.example.com/users';
  
  constructor(private http: HttpClient) {}
  
  // GET request
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }
  
  // GET by ID
  getUser(id: string): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }
  
  // POST request
  createUser(user: User): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }
  
  // PUT request
  updateUser(id: string, user: User): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user);
  }
  
  // PATCH request
  patchUser(id: string, updates: Partial<User>): Observable<User> {
    return this.http.patch<User>(`${this.apiUrl}/${id}`, updates);
  }
  
  // DELETE request
  deleteUser(id: string): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}

interface User {
  id: string;
  name: string;
  email: string;
}
```

---

### Q2: How to handle HTTP requests in components?

**Difficulty:** Easy

**Code:**
```typescript
// ❌ BAD: Subscribe in component without cleanup
@Component({
  selector: 'app-users',
  template: `
    <div *ngFor="let user of users">{{ user.name }}</div>
  `
})
export class BadUsersComponent implements OnInit {
  users: User[] = [];
  
  constructor(private userService: UserService) {}
  
  ngOnInit() {
    // ❌ Memory leak - no unsubscribe
    this.userService.getUsers().subscribe(users => {
      this.users = users;
    });
  }
}

// ✅ GOOD: Use async pipe (auto unsubscribe)
@Component({
  selector: 'app-users',
  template: `
    <div *ngIf="users$ | async as users; else loading">
      <div *ngFor="let user of users">{{ user.name }}</div>
    </div>
    
    <ng-template #loading>
      <p>Loading...</p>
    </ng-template>
  `
})
export class GoodUsersComponent implements OnInit {
  users$: Observable<User[]>;
  
  constructor(private userService: UserService) {}
  
  ngOnInit() {
    // ✅ No memory leak with async pipe
    this.users$ = this.userService.getUsers();
  }
}

// ✅ ALTERNATIVE: Manual subscription with cleanup
@Component({
  selector: 'app-users-manual',
  template: `
    <div *ngFor="let user of users">{{ user.name }}</div>
    <div *ngIf="loading">Loading...</div>
    <div *ngIf="error">{{ error }}</div>
  `
})
export class ManualUsersComponent implements OnInit, OnDestroy {
  users: User[] = [];
  loading = false;
  error: string = null;
  
  private destroy$ = new Subject<void>();
  
  constructor(private userService: UserService) {}
  
  ngOnInit() {
    this.loading = true;
    
    this.userService.getUsers().pipe(
      takeUntil(this.destroy$),
      finalize(() => this.loading = false)
    ).subscribe({
      next: (users) => this.users = users,
      error: (err) => this.error = err.message
    });
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// ✅ BEST: Reactive state management
@Component({
  selector: 'app-users-reactive',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div *ngIf="state$ | async as state">
      <div *ngIf="state.loading">Loading...</div>
      
      <div *ngIf="state.error" class="error">
        {{ state.error }}
      </div>
      
      <div *ngFor="let user of state.users">
        {{ user.name }}
      </div>
    </div>
    
    <button (click)="refresh()">Refresh</button>
  `
})
export class ReactiveUsersComponent implements OnInit {
  private refreshSubject = new Subject<void>();
  
  state$ = this.refreshSubject.pipe(
    startWith(null),
    switchMap(() => 
      this.userService.getUsers().pipe(
        map(users => ({ loading: false, users, error: null })),
        startWith({ loading: true, users: [], error: null }),
        catchError(error => of({ loading: false, users: [], error: error.message }))
      )
    )
  );
  
  constructor(private userService: UserService) {}
  
  ngOnInit() {
    this.refresh();
  }
  
  refresh() {
    this.refreshSubject.next();
  }
}
```

---

### Q3: How to pass query parameters and headers?

**Difficulty:** Easy

**Code:**
```typescript
import { HttpClient, HttpParams, HttpHeaders } from '@angular/common/http';

@Injectable({ providedIn: 'root' })
export class ApiService {
  private apiUrl = 'https://api.example.com';
  
  constructor(private http: HttpClient) {}
  
  // Query parameters - Method 1: HttpParams
  getUsersWithParams(page: number, size: number, sort: string): Observable<User[]> {
    const params = new HttpParams()
      .set('page', page.toString())
      .set('size', size.toString())
      .set('sort', sort);
    
    return this.http.get<User[]>(`${this.apiUrl}/users`, { params });
  }
  
  // Query parameters - Method 2: Object
  getUsersWithObject(filters: any): Observable<User[]> {
    return this.http.get<User[]>(`${this.apiUrl}/users`, {
      params: {
        page: filters.page,
        size: filters.size,
        sort: filters.sort
      }
    });
  }
  
  // Query parameters - Method 3: fromObject
  searchUsers(criteria: SearchCriteria): Observable<User[]> {
    const params = new HttpParams({ fromObject: criteria as any });
    return this.http.get<User[]>(`${this.apiUrl}/users/search`, { params });
  }
  
  // Custom headers
  getUserWithHeaders(): Observable<User> {
    const headers = new HttpHeaders()
      .set('Authorization', 'Bearer token123')
      .set('Content-Type', 'application/json')
      .set('X-Custom-Header', 'custom-value');
    
    return this.http.get<User>(`${this.apiUrl}/user`, { headers });
  }
  
  // Combined: params + headers
  searchWithAuth(query: string, token: string): Observable<any[]> {
    const params = new HttpParams()
      .set('q', query)
      .set('limit', '10');
    
    const headers = new HttpHeaders()
      .set('Authorization', `Bearer ${token}`);
    
    return this.http.get<any[]>(`${this.apiUrl}/search`, { params, headers });
  }
  
  // URL with template literals
  getUserByIdAndRole(userId: string, role: string): Observable<User> {
    return this.http.get<User>(
      `${this.apiUrl}/users/${userId}/role/${role}`
    );
  }
  
  // Dynamic query building
  buildDynamicQuery(filters: Record<string, any>): Observable<any[]> {
    let params = new HttpParams();
    
    Object.keys(filters).forEach(key => {
      if (filters[key] !== null && filters[key] !== undefined) {
        params = params.set(key, filters[key].toString());
      }
    });
    
    return this.http.get<any[]>(`${this.apiUrl}/data`, { params });
  }
  
  // Array parameters
  getUsersByIds(ids: string[]): Observable<User[]> {
    let params = new HttpParams();
    ids.forEach(id => {
      params = params.append('id', id);
    });
    
    // Results in: /users?id=1&id=2&id=3
    return this.http.get<User[]>(`${this.apiUrl}/users`, { params });
  }
}

interface SearchCriteria {
  name?: string;
  email?: string;
  status?: string;
  page: number;
  size: number;
}
```

---

### Q4: How to handle different response types?

**Difficulty:** Medium

**Code:**
```typescript
@Injectable({ providedIn: 'root' })
export class ResponseTypeService {
  private apiUrl = 'https://api.example.com';
  
  constructor(private http: HttpClient) {}
  
  // 1. JSON response (default)
  getJsonData(): Observable<any> {
    return this.http.get(`${this.apiUrl}/data`);
  }
  
  // 2. Text response
  getTextData(): Observable<string> {
    return this.http.get(`${this.apiUrl}/text`, {
      responseType: 'text'
    });
  }
  
  // 3. Blob response (for files)
  downloadFile(): Observable<Blob> {
    return this.http.get(`${this.apiUrl}/file`, {
      responseType: 'blob'
    });
  }
  
  // 4. ArrayBuffer response
  getBinaryData(): Observable<ArrayBuffer> {
    return this.http.get(`${this.apiUrl}/binary`, {
      responseType: 'arraybuffer'
    });
  }
  
  // 5. Full HTTP response (with headers, status, etc.)
  getFullResponse(): Observable<HttpResponse<User>> {
    return this.http.get<User>(`${this.apiUrl}/user`, {
      observe: 'response'
    });
  }
  
  // 6. HTTP events (for progress tracking)
  uploadWithProgress(file: File): Observable<HttpEvent<any>> {
    const formData = new FormData();
    formData.append('file', file);
    
    return this.http.post(`${this.apiUrl}/upload`, formData, {
      reportProgress: true,
      observe: 'events'
    });
  }
  
  // 7. Custom response handling
  getUserWithMetadata(): Observable<{ user: User; headers: HttpHeaders }> {
    return this.http.get<User>(`${this.apiUrl}/user`, {
      observe: 'response'
    }).pipe(
      map(response => ({
        user: response.body,
        headers: response.headers
      }))
    );
  }
}

// Usage in component
@Component({})
export class ResponseComponent {
  constructor(private service: ResponseTypeService) {}
  
  // Handle full response
  loadUser() {
    this.service.getFullResponse().subscribe(response => {
      console.log('Status:', response.status);
      console.log('Headers:', response.headers.get('Content-Type'));
      console.log('Body:', response.body);
    });
  }
  
  // Handle upload progress
  uploadFile(file: File) {
    this.service.uploadWithProgress(file).subscribe(event => {
      if (event.type === HttpEventType.UploadProgress) {
        const progress = Math.round(100 * event.loaded / event.total);
        console.log(`Upload: ${progress}%`);
      } else if (event.type === HttpEventType.Response) {
        console.log('Upload complete:', event.body);
      }
    });
  }
  
  // Download file
  downloadFile() {
    this.service.downloadFile().subscribe(blob => {
      const url = window.URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = 'file.pdf';
      link.click();
      window.URL.revokeObjectURL(url);
    });
  }
}
```

---

### Q5: How to cancel HTTP requests?

**Difficulty:** Medium

**Code:**
```typescript
import { Subject, takeUntil, switchMap } from 'rxjs';

// Method 1: Using takeUntil
@Component({
  selector: 'app-search',
  template: `
    <input (input)="search($event.target.value)">
    <button (click)="cancelSearch()">Cancel</button>
    
    <div *ngFor="let result of results">{{ result.name }}</div>
  `
})
export class SearchComponent implements OnDestroy {
  results: any[] = [];
  private cancelSearch$ = new Subject<void>();
  private destroy$ = new Subject<void>();
  
  constructor(private searchService: SearchService) {}
  
  search(query: string) {
    // Cancel previous request
    this.cancelSearch$.next();
    
    this.searchService.search(query).pipe(
      takeUntil(this.cancelSearch$),
      takeUntil(this.destroy$)
    ).subscribe(results => {
      this.results = results;
    });
  }
  
  cancelSearch() {
    this.cancelSearch$.next();
    this.results = [];
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// Method 2: Using switchMap (auto-cancellation)
@Component({
  selector: 'app-autocomplete',
  template: `
    <input [formControl]="searchControl">
    <div *ngFor="let result of results$ | async">{{ result.name }}</div>
  `
})
export class AutocompleteComponent implements OnInit {
  searchControl = new FormControl('');
  results$: Observable<any[]>;
  
  constructor(private searchService: SearchService) {}
  
  ngOnInit() {
    this.results$ = this.searchControl.valueChanges.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(query => 
        query ? this.searchService.search(query) : of([])
      )
    );
  }
}

// Method 3: AbortController (manual cancellation)
@Injectable({ providedIn: 'root' })
export class CancellableService {
  private abortControllers = new Map<string, AbortController>();
  
  constructor(private http: HttpClient) {}
  
  getData(key: string): Observable<any> {
    // Cancel previous request with same key
    this.cancel(key);
    
    // Create new AbortController
    const controller = new AbortController();
    this.abortControllers.set(key, controller);
    
    return this.http.get('/api/data', {
      context: new HttpContext().set(ABORT_SIGNAL, controller.signal)
    }).pipe(
      finalize(() => this.abortControllers.delete(key))
    );
  }
  
  cancel(key: string) {
    const controller = this.abortControllers.get(key);
    if (controller) {
      controller.abort();
      this.abortControllers.delete(key);
    }
  }
  
  cancelAll() {
    this.abortControllers.forEach(controller => controller.abort());
    this.abortControllers.clear();
  }
}

// Method 4: Unsubscribe pattern
@Component({})
export class UnsubscribeComponent implements OnDestroy {
  private subscription: Subscription;
  
  constructor(private dataService: DataService) {}
  
  loadData() {
    // Cancel previous request
    if (this.subscription) {
      this.subscription.unsubscribe();
    }
    
    // New request
    this.subscription = this.dataService.getData().subscribe(data => {
      console.log('Data loaded:', data);
    });
  }
  
  ngOnDestroy() {
    if (this.subscription) {
      this.subscription.unsubscribe();
    }
  }
}

// Method 5: Route guard cancellation
@Injectable()
export class DataResolver implements Resolve<any> {
  constructor(private dataService: DataService) {}
  
  resolve(route: ActivatedRouteSnapshot): Observable<any> {
    // Automatically cancelled when navigating away
    return this.dataService.getData();
  }
}
```

---

### Q6: How to make sequential HTTP requests?

**Difficulty:** Medium

**Code:**
```typescript
import { concatMap, mergeMap, switchMap, forkJoin, combineLatest } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class SequentialRequestsService {
  constructor(private http: HttpClient) {}
  
  // Pattern 1: concatMap - Sequential execution, maintains order
  createUserWithProfile(user: User, profile: Profile): Observable<any> {
    return this.http.post<User>('/api/users', user).pipe(
      concatMap(createdUser => 
        this.http.post('/api/profiles', {
          ...profile,
          userId: createdUser.id
        })
      )
    );
  }
  
  // Pattern 2: Multiple sequential requests
  loadUserDataSequentially(userId: string): Observable<UserData> {
    return this.http.get<User>(`/api/users/${userId}`).pipe(
      concatMap(user => 
        this.http.get<Profile>(`/api/profiles/${user.profileId}`).pipe(
          map(profile => ({ user, profile }))
        )
      ),
      concatMap(({ user, profile }) =>
        this.http.get<Order[]>(`/api/orders?userId=${user.id}`).pipe(
          map(orders => ({ user, profile, orders }))
        )
      )
    );
  }
  
  // Pattern 3: Chain with error handling
  processOrderSequentially(order: Order): Observable<ProcessResult> {
    return this.http.post<Order>('/api/orders', order).pipe(
      concatMap(createdOrder =>
        this.http.post(`/api/payments`, {
          orderId: createdOrder.id,
          amount: createdOrder.total
        }).pipe(
          catchError(error => {
            // Rollback order if payment fails
            return this.http.delete(`/api/orders/${createdOrder.id}`).pipe(
              switchMap(() => throwError(() => error))
            );
          })
        )
      ),
      concatMap(payment =>
        this.http.post('/api/notifications', {
          type: 'order_confirmation',
          orderId: payment.orderId
        }).pipe(
          map(() => ({ success: true, payment }))
        )
      )
    );
  }
  
  // Pattern 4: Array of sequential requests
  processMultipleSequentially(items: string[]): Observable<any[]> {
    return from(items).pipe(
      concatMap(item => this.http.post('/api/process', { item })),
      toArray() // Collect all results into array
    );
  }
  
  // Pattern 5: Conditional sequential requests
  getUserWithOptionalData(userId: string, includeOrders: boolean): Observable<any> {
    return this.http.get<User>(`/api/users/${userId}`).pipe(
      concatMap(user => {
        if (includeOrders) {
          return this.http.get<Order[]>(`/api/orders?userId=${user.id}`).pipe(
            map(orders => ({ user, orders }))
          );
        }
        return of({ user, orders: [] });
      })
    );
  }
  
  // Pattern 6: Waterfall pattern with accumulation
  processWaterfall(data: any): Observable<any> {
    return this.http.post('/api/step1', data).pipe(
      concatMap(step1Result =>
        this.http.post('/api/step2', {
          ...data,
          step1: step1Result
        }).pipe(
          concatMap(step2Result =>
            this.http.post('/api/step3', {
              ...data,
              step1: step1Result,
              step2: step2Result
            })
          )
        )
      )
    );
  }
}
```

---

### Q7: How to make parallel HTTP requests?

**Difficulty:** Medium

**Code:**
```typescript
import { forkJoin, combineLatest, merge, zip } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class ParallelRequestsService {
  constructor(private http: HttpClient) {}
  
  // Pattern 1: forkJoin - Wait for all to complete
  loadDashboardData(): Observable<DashboardData> {
    return forkJoin({
      users: this.http.get<User[]>('/api/users'),
      orders: this.http.get<Order[]>('/api/orders'),
      products: this.http.get<Product[]>('/api/products'),
      stats: this.http.get<Stats>('/api/stats')
    });
  }
  
  // Pattern 2: forkJoin with array
  loadMultipleUsers(ids: string[]): Observable<User[]> {
    const requests = ids.map(id => 
      this.http.get<User>(`/api/users/${id}`)
    );
    
    return forkJoin(requests);
  }
  
  // Pattern 3: combineLatest - Emit on any change
  loadLiveData(): Observable<[User[], Order[], number]> {
    return combineLatest([
      this.http.get<User[]>('/api/users'),
      this.http.get<Order[]>('/api/orders'),
      this.http.get<number>('/api/count')
    ]);
  }
  
  // Pattern 4: forkJoin with error handling
  loadDataWithFallback(): Observable<any> {
    return forkJoin({
      critical: this.http.get('/api/critical'),
      optional: this.http.get('/api/optional').pipe(
        catchError(error => of(null)) // Continue if optional fails
      ),
      cached: this.http.get('/api/cached').pipe(
        catchError(error => of({ cached: true }))
      )
    });
  }
  
  // Pattern 5: Partial results with merge
  loadDataWithMerge(): Observable<any> {
    return merge(
      this.http.get<User[]>('/api/users').pipe(
        map(users => ({ type: 'users', data: users }))
      ),
      this.http.get<Order[]>('/api/orders').pipe(
        map(orders => ({ type: 'orders', data: orders }))
      ),
      this.http.get<Product[]>('/api/products').pipe(
        map(products => ({ type: 'products', data: products }))
      )
    );
  }
  
  // Pattern 6: Race condition - first to complete wins
  loadFromMultipleSources(userId: string): Observable<User> {
    return race(
      this.http.get<User>(`https://api1.example.com/users/${userId}`),
      this.http.get<User>(`https://api2.example.com/users/${userId}`),
      this.http.get<User>(`https://api3.example.com/users/${userId}`)
    ).pipe(
      timeout(5000),
      catchError(error => throwError(() => new Error('All sources failed')))
    );
  }
  
  // Pattern 7: Batched parallel requests
  loadInBatches(ids: string[], batchSize: number): Observable<User[]> {
    const batches = [];
    for (let i = 0; i < ids.length; i += batchSize) {
      const batch = ids.slice(i, i + batchSize);
      batches.push(batch);
    }
    
    return from(batches).pipe(
      concatMap(batch => 
        forkJoin(
          batch.map(id => this.http.get<User>(`/api/users/${id}`))
        )
      ),
      scan((acc, batch) => [...acc, ...batch], [])
    );
  }
  
  // Pattern 8: Parallel with progress tracking
  loadMultipleWithProgress(ids: string[]): Observable<LoadProgress> {
    let completed = 0;
    const total = ids.length;
    
    const requests = ids.map(id =>
      this.http.get<User>(`/api/users/${id}`).pipe(
        tap(() => {
          completed++;
          console.log(`Progress: ${completed}/${total}`);
        })
      )
    );
    
    return forkJoin(requests).pipe(
      map(users => ({
        completed: true,
        progress: 100,
        data: users
      }))
    );
  }
}

// Usage in component
@Component({})
export class DashboardComponent implements OnInit {
  constructor(private parallelService: ParallelRequestsService) {}
  
  ngOnInit() {
    // Wait for all requests
    this.parallelService.loadDashboardData().subscribe(data => {
      console.log('All data loaded:', data);
      // Access: data.users, data.orders, data.products, data.stats
    });
    
    // Handle partial results as they arrive
    this.parallelService.loadDataWithMerge().subscribe(result => {
      console.log(`${result.type} loaded:`, result.data);
      // Handle each data type as it arrives
    });
  }
}

interface LoadProgress {
  completed: boolean;
  progress: number;
  data: any[];
}
```

---

### Q8: How to implement request retry logic?

**Difficulty:** Medium

**Code:**
```typescript
import { retry, retryWhen, delay, scan, tap, catchError } from 'rxjs/operators';

@Injectable({ providedIn: 'root' })
export class RetryService {
  constructor(private http: HttpClient) {}
  
  // Pattern 1: Simple retry - retry N times
  getData(): Observable<any> {
    return this.http.get('/api/data').pipe(
      retry(3), // Retry 3 times
      catchError(error => {
        console.error('Failed after 3 retries:', error);
        return throwError(() => error);
      })
    );
  }
  
  // Pattern 2: Retry with delay
  getDataWithDelay(): Observable<any> {
    return this.http.get('/api/data').pipe(
      retryWhen(errors =>
        errors.pipe(
          delay(1000), // Wait 1 second between retries
          take(3), // Maximum 3 retries
          tap(() => console.log('Retrying...')),
          concatMap((error, index) => {
            if (index >= 2) {
              return throwError(() => error);
            }
            return of(error);
          })
        )
      )
    );
  }
  
  // Pattern 3: Exponential backoff
  getDataWithBackoff(): Observable<any> {
    return this.http.get('/api/data').pipe(
      retryWhen(errors =>
        errors.pipe(
          scan((retryCount, error) => {
            if (retryCount >= 5) {
              throw error;
            }
            return retryCount + 1;
          }, 0),
          tap(retryCount => {
            const waitTime = Math.pow(2, retryCount) * 1000;
            console.log(`Retry ${retryCount}, waiting ${waitTime}ms`);
          }),
          delayWhen(retryCount => timer(Math.pow(2, retryCount) * 1000))
        )
      )
    );
  }
  
  // Pattern 4: Conditional retry (only on specific errors)
  getDataWithConditionalRetry(): Observable<any> {
    return this.http.get('/api/data').pipe(
      retryWhen(errors =>
        errors.pipe(
          scan((retryCount, error) => {
            // Only retry on 500 errors
            if (error.status >= 500 && retryCount < 3) {
              return retryCount + 1;
            }
            // Don't retry on 4xx errors
            throw error;
          }, 0),
          delay(1000)
        )
      )
    );
  }
  
  // Pattern 5: Custom retry strategy
  getDataWithCustomRetry(): Observable<any> {
    return this.http.get('/api/data').pipe(
      this.retryWithBackoff({
        maxRetries: 3,
        initialDelay: 1000,
        maxDelay: 10000,
        backoffFactor: 2
      })
    );
  }
  
  // Custom retry operator
  private retryWithBackoff(config: RetryConfig) {
    return <T>(source: Observable<T>) =>
      source.pipe(
        retryWhen(errors =>
          errors.pipe(
            scan((acc, error) => {
              const retryAttempt = acc.retryCount + 1;
              
              if (retryAttempt > config.maxRetries) {
                throw error;
              }
              
              const delay = Math.min(
                config.initialDelay * Math.pow(config.backoffFactor, acc.retryCount),
                config.maxDelay
              );
              
              console.log(`Retry attempt ${retryAttempt} after ${delay}ms`);
              
              return {
                retryCount: retryAttempt,
                delay,
                error
              };
            }, { retryCount: