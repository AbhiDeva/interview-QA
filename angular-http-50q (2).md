[text](<angular-http-50q (1).md>)  private retryWithBackoff(config: RetryConfig) {
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
            }, { retryCount: 0, delay: 0, error: null }),
            delayWhen(acc => timer(acc.delay))
          )
        )
      );
  }
  
  // Pattern 6: Retry with timeout
  getDataWithTimeout(): Observable<any> {
    return this.http.get('/api/data').pipe(
      timeout(5000), // 5 second timeout
      retry(3),
      catchError(error => {
        if (error.name === 'TimeoutError') {
          console.error('Request timed out');
        }
        return throwError(() => error);
      })
    );
  }
  
  // Pattern 7: Retry with circuit breaker
  private failureCount = 0;
  private circuitOpen = false;
  
  getDataWithCircuitBreaker(): Observable<any> {
    if (this.circuitOpen) {
      return throwError(() => new Error('Circuit breaker is open'));
    }
    
    return this.http.get('/api/data').pipe(
      tap(() => {
        // Reset on success
        this.failureCount = 0;
      }),
      catchError(error => {
        this.failureCount++;
        
        // Open circuit after 5 failures
        if (this.failureCount >= 5) {
          this.circuitOpen = true;
          console.log('Circuit breaker opened');
          
          // Close circuit after 30 seconds
          setTimeout(() => {
            this.circuitOpen = false;
            this.failureCount = 0;
            console.log('Circuit breaker closed');
          }, 30000);
        }
        
        return throwError(() => error);
      }),
      retry(3)
    );
  }
}

interface RetryConfig {
  maxRetries: number;
  initialDelay: number;
  maxDelay: number;
  backoffFactor: number;
}
```

---

### Q9: How to implement request polling?

**Difficulty:** Medium

**Code:**
```typescript
import { interval, timer, switchMap, takeWhile, repeatWhen } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class PollingService {
  constructor(private http: HttpClient) {}
  
  // Pattern 1: Simple interval polling
  pollData(intervalMs: number = 5000): Observable<any> {
    return interval(intervalMs).pipe(
      startWith(0), // Immediate first call
      switchMap(() => this.http.get('/api/data'))
    );
  }
  
  // Pattern 2: Polling with start/stop control
  createPollableRequest(intervalMs: number = 5000): {
    data$: Observable<any>,
    stop: () => void
  } {
    const stopSubject = new Subject<void>();
    
    const data$ = interval(intervalMs).pipe(
      startWith(0),
      switchMap(() => this.http.get('/api/data')),
      takeUntil(stopSubject)
    );
    
    return {
      data$,
      stop: () => stopSubject.next()
    };
  }
  
  // Pattern 3: Conditional polling (stop on condition)
  pollUntilComplete(taskId: string): Observable<Task> {
    return interval(2000).pipe(
      startWith(0),
      switchMap(() => this.http.get<Task>(`/api/tasks/${taskId}`)),
      takeWhile(task => task.status !== 'completed', true), // true = inclusive
      tap(task => console.log('Task status:', task.status))
    );
  }
  
  // Pattern 4: Exponential backoff polling
  pollWithBackoff(taskId: string): Observable<Task> {
    let attempt = 0;
    
    return defer(() => this.http.get<Task>(`/api/tasks/${taskId}`)).pipe(
      repeatWhen(notifications =>
        notifications.pipe(
          delay(() => {
            attempt++;
            const delayMs = Math.min(1000 * Math.pow(2, attempt), 30000);
            console.log(`Next poll in ${delayMs}ms`);
            return timer(delayMs);
          })
        )
      ),
      takeWhile(task => task.status !== 'completed', true)
    );
  }
  
  // Pattern 5: Smart polling (adjust based on response)
  smartPoll(endpoint: string): Observable<any> {
    let pollInterval = 1000; // Start with 1 second
    
    return timer(0, pollInterval).pipe(
      switchMap(() => this.http.get<any>(endpoint)),
      tap(response => {
        // Adjust polling interval based on response
        if (response.isStale) {
          pollInterval = Math.max(pollInterval / 2, 500); // Faster
        } else {
          pollInterval = Math.min(pollInterval * 1.5, 30000); // Slower
        }
      })
    );
  }
  
  // Pattern 6: Polling with retry
  pollWithRetry(endpoint: string, intervalMs: number = 5000): Observable<any> {
    return interval(intervalMs).pipe(
      startWith(0),
      switchMap(() =>
        this.http.get(endpoint).pipe(
          retry(3),
          catchError(error => {
            console.error('Poll failed:', error);
            return of(null);
          })
        )
      ),
      filter(data => data !== null)
    );
  }
  
  // Pattern 7: Long polling (wait for server response)
  longPoll(endpoint: string): Observable<any> {
    return this.http.get(endpoint, {
      params: { timeout: '30000' } // Server holds connection
    }).pipe(
      repeatWhen(notifications =>
        notifications.pipe(
          delay(1000) // Small delay before next long poll
        )
      )
    );
  }
  
  // Pattern 8: Polling with window focus
  pollWhenFocused(endpoint: string, intervalMs: number = 5000): Observable<any> {
    const focused$ = merge(
      of(document.hasFocus()),
      fromEvent(window, 'focus').pipe(map(() => true)),
      fromEvent(window, 'blur').pipe(map(() => false))
    );
    
    return focused$.pipe(
      switchMap(focused =>
        focused
          ? interval(intervalMs).pipe(
              startWith(0),
              switchMap(() => this.http.get(endpoint))
            )
          : EMPTY
      )
    );
  }
}

// Component usage
@Component({
  selector: 'app-status-monitor',
  template: `
    <div *ngIf="task$ | async as task">
      <p>Status: {{ task.status }}</p>
      <p>Progress: {{ task.progress }}%</p>
    </div>
    
    <button (click)="startPolling()" [disabled]="polling">Start</button>
    <button (click)="stopPolling()" [disabled]="!polling">Stop</button>
  `
})
export class StatusMonitorComponent implements OnDestroy {
  task$: Observable<Task>;
  polling = false;
  private stopPolling$ = new Subject<void>();
  
  constructor(private pollingService: PollingService) {}
  
  startPolling() {
    this.polling = true;
    
    this.task$ = this.pollingService.pollUntilComplete('task-123').pipe(
      takeUntil(this.stopPolling$),
      finalize(() => {
        this.polling = false;
        console.log('Polling stopped');
      })
    );
  }
  
  stopPolling() {
    this.stopPolling$.next();
  }
  
  ngOnDestroy() {
    this.stopPolling$.next();
    this.stopPolling$.complete();
  }
}

interface Task {
  id: string;
  status: 'pending' | 'processing' | 'completed' | 'failed';
  progress: number;
}
```

---

### Q10: How to implement request debouncing and throttling?

**Difficulty:** Medium

**Code:**
```typescript
import { debounceTime, throttleTime, distinctUntilChanged, switchMap } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class DebounceThrottleService {
  constructor(private http: HttpClient) {}
  
  // Debounce: Wait for pause in emissions
  searchWithDebounce(query$: Observable<string>): Observable<any[]> {
    return query$.pipe(
      debounceTime(300), // Wait 300ms after last keystroke
      distinctUntilChanged(), // Only if value changed
      switchMap(query => 
        query ? this.http.get<any[]>(`/api/search?q=${query}`) : of([])
      )
    );
  }
  
  // Throttle: Limit emission rate
  autoSaveWithThrottle(changes$: Observable<any>): Observable<any> {
    return changes$.pipe(
      throttleTime(2000), // Maximum once per 2 seconds
      switchMap(data => this.http.post('/api/save', data))
    );
  }
  
  // Combination: Debounce + Throttle
  smartSearch(query$: Observable<string>): Observable<any[]> {
    return query$.pipe(
      debounceTime(300), // Wait for pause
      throttleTime(1000), // But at most once per second
      distinctUntilChanged(),
      switchMap(query => this.http.get<any[]>(`/api/search?q=${query}`))
    );
  }
}

// Search component with debounce
@Component({
  selector: 'app-search',
  template: `
    <input 
      type="text" 
      [formControl]="searchControl"
      placeholder="Search...">
    
    <div *ngIf="loading">Searching...</div>
    
    <div *ngFor="let result of results$ | async">
      {{ result.name }}
    </div>
  `
})
export class SearchComponent implements OnInit {
  searchControl = new FormControl('');
  results$: Observable<any[]>;
  loading = false;
  
  constructor(private http: HttpClient) {}
  
  ngOnInit() {
    this.results$ = this.searchControl.valueChanges.pipe(
      tap(() => this.loading = true),
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(query => 
        query.length >= 3
          ? this.http.get<any[]>(`/api/search?q=${query}`)
          : of([])
      ),
      tap(() => this.loading = false),
      catchError(error => {
        console.error('Search error:', error);
        this.loading = false;
        return of([]);
      })
    );
  }
}

// Auto-save with throttle
@Component({
  selector: 'app-editor',
  template: `
    <textarea 
      [formControl]="contentControl"
      placeholder="Start typing...">
    </textarea>
    
    <div *ngIf="saving">Saving...</div>
    <div *ngIf="saved">Saved!</div>
  `
})
export class EditorComponent implements OnInit, OnDestroy {
  contentControl = new FormControl('');
  saving = false;
  saved = false;
  
  private destroy$ = new Subject<void>();
  
  constructor(private http: HttpClient) {}
  
  ngOnInit() {
    this.contentControl.valueChanges.pipe(
      tap(() => {
        this.saving = true;
        this.saved = false;
      }),
      debounceTime(1000), // Wait 1s after typing stops
      distinctUntilChanged(),
      switchMap(content =>
        this.http.post('/api/save', { content }).pipe(
          tap(() => {
            this.saving = false;
            this.saved = true;
            setTimeout(() => this.saved = false, 2000);
          }),
          catchError(error => {
            this.saving = false;
            console.error('Save failed:', error);
            return of(null);
          })
        )
      ),
      takeUntil(this.destroy$)
    ).subscribe();
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

---

## HTTP Interceptors

### Q11: How to create and use HTTP interceptors?

**Difficulty:** Medium

**Code:**
```typescript
// Basic interceptor
import { HttpInterceptor, HttpRequest, HttpHandler, HttpEvent } from '@angular/common/http';

@Injectable()
export class LoggingInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    console.log('Request:', req.method, req.url);
    
    return next.handle(req).pipe(
      tap({
        next: (event) => {
          if (event.type === HttpEventType.Response) {
            console.log('Response:', event.status, event.body);
          }
        },
        error: (error) => {
          console.error('Error:', error);
        }
      })
    );
  }
}

// Auth token interceptor
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private authService: AuthService) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Get token
    const token = this.authService.getToken();
    
    // Clone request and add authorization header
    if (token) {
      req = req.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`
        }
      });
    }
    
    return next.handle(req);
  }
}

// Error handling interceptor
@Injectable()
export class ErrorInterceptor implements HttpInterceptor {
  constructor(
    private notificationService: NotificationService,
    private router: Router
  ) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      catchError((error: HttpErrorResponse) => {
        let errorMessage = '';
        
        if (error.error instanceof ErrorEvent) {
          // Client-side error
          errorMessage = `Client Error: ${error.error.message}`;
        } else {
          // Server-side error
          errorMessage = `Server Error: ${error.status} - ${error.message}`;
          
          // Handle specific status codes
          switch (error.status) {
            case 401:
              this.router.navigate(['/login']);
              break;
            case 403:
              this.notificationService.error('Access denied');
              break;
            case 404:
              this.notificationService.error('Resource not found');
              break;
            case 500:
              this.notificationService.error('Server error');
              break;
          }
        }
        
        console.error(errorMessage);
        return throwError(() => error);
      })
    );
  }
}

// Loading interceptor
@Injectable()
export class LoadingInterceptor implements HttpInterceptor {
  private requests = 0;
  
  constructor(private loadingService: LoadingService) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Skip loading for specific endpoints
    if (req.url.includes('/api/ping')) {
      return next.handle(req);
    }
    
    this.requests++;
    this.loadingService.show();
    
    return next.handle(req).pipe(
      finalize(() => {
        this.requests--;
        if (this.requests === 0) {
          this.loadingService.hide();
        }
      })
    );
  }
}

// Caching interceptor
@Injectable()
export class CacheInterceptor implements HttpInterceptor {
  private cache = new Map<string, HttpResponse<any>>();
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Only cache GET requests
    if (req.method !== 'GET') {
      return next.handle(req);
    }
    
    // Check cache
    const cachedResponse = this.cache.get(req.url);
    if (cachedResponse) {
      console.log('Returning cached response');
      return of(cachedResponse);
    }
    
    // Make request and cache response
    return next.handle(req).pipe(
      tap(event => {
        if (event instanceof HttpResponse) {
          this.cache.set(req.url, event);
          
          // Clear cache after 5 minutes
          setTimeout(() => this.cache.delete(req.url), 5 * 60 * 1000);
        }
      })
    );
  }
}

// Retry interceptor
@Injectable()
export class RetryInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      retry({
        count: 3,
        delay: (error, retryCount) => {
          // Only retry on 5xx errors
          if (error.status >= 500 && error.status < 600) {
            const delayMs = Math.pow(2, retryCount) * 1000;
            console.log(`Retry ${retryCount} after ${delayMs}ms`);
            return timer(delayMs);
          }
          // Don't retry on other errors
          return throwError(() => error);
        }
      })
    );
  }
}

// Timeout interceptor
@Injectable()
export class TimeoutInterceptor implements HttpInterceptor {
  private readonly DEFAULT_TIMEOUT = 30000; // 30 seconds
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const timeoutValue = req.headers.get('timeout') || this.DEFAULT_TIMEOUT;
    
    return next.handle(req).pipe(
      timeout(+timeoutValue),
      catchError(error => {
        if (error.name === 'TimeoutError') {
          console.error('Request timeout:', req.url);
        }
        return throwError(() => error);
      })
    );
  }
}

// Register interceptors (app.config.ts - Standalone)
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([
        loggingInterceptor,
        authInterceptor,
        errorInterceptor,
        loadingInterceptor
      ])
    )
  ]
};

// Register interceptors (app.module.ts - Module)
@NgModule({
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: LoggingInterceptor,
      multi: true
    },
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptor,
      multi: true
    },
    {
      provide: HTTP_INTERCEPTORS,
      useClass: ErrorInterceptor,
      multi: true
    }
  ]
})
export class AppModule {}
```

---

### Q12: How to create functional interceptors (Angular 15+)?

**Difficulty:** Medium

**Code:**
```typescript
import { HttpInterceptorFn } from '@angular/common/http';

// Logging interceptor
export const loggingInterceptor: HttpInterceptorFn = (req, next) => {
  console.log('Request:', req.method, req.url);
  
  return next(req).pipe(
    tap({
      next: (event) => {
        if (event.type === HttpEventType.Response) {
          console.log('Response:', event.status);
        }
      },
      error: (error) => console.error('Error:', error)
    })
  );
};

// Auth interceptor
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();
  
  if (token) {
    req = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });
  }
  
  return next(req);
};

// Error handling interceptor
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const router = inject(Router);
  const snackBar = inject(MatSnackBar);
  
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        router.navigate(['/login']);
      } else if (error.status === 403) {
        snackBar.open('Access denied', 'Close', { duration: 3000 });
      }
      
      return throwError(() => error);
    })
  );
};

// Retry with backoff interceptor
export const retryInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    retry({
      count: 3,
      delay: (error, retryCount) => {
        if (error.status >= 500) {
          return timer(Math.pow(2, retryCount) * 1000);
        }
        return throwError(() => error);
      }
    })
  );
};

// Cache interceptor
const cache = new Map<string, HttpResponse<any>>();

export const cacheInterceptor: HttpInterceptorFn = (req, next) => {
  if (req.method !== 'GET') {
    return next(req);
  }
  
  const cached = cache.get(req.url);
  if (cached) {
    return of(cached);
  }
  
  return next(req).pipe(
    tap(event => {
      if (event instanceof HttpResponse) {
        cache.set(req.url, event);
      }
    })
  );
};

// Token refresh interceptor
export const tokenRefreshInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401 && !req.url.includes('/refresh')) {
        // Token expired, try to refresh
        return authService.refreshToken().pipe(
          switchMap(newToken => {
            // Retry original request with new token
            const clonedReq = req.clone({
              setHeaders: {
                Authorization: `Bearer ${newToken}`
              }
            });
            return next(clonedReq);
          }),
          catchError(refreshError => {
            // Refresh failed, logout
            authService.logout();
            return throwError(() => refreshError);
          })
        );
      }
      
      return throwError(() => error);
    })
  );
};

// Base URL interceptor
export const baseUrlInterceptor: HttpInterceptorFn = (req, next) => {
  const baseUrl = 'https://api.example.com';
  
  // Only add base URL if request is relative
  if (!req.url.startsWith('http')) {
    req = req.clone({
      url: `${baseUrl}${req.url}`
    });
  }
  
  return next(req);
};

// Request timing interceptor
export const timingInterceptor: HttpInterceptorFn = (req, next) => {
  const started = Date.now();
  
  return next(req).pipe(
    tap({
      next: (event) => {
        if (event.type === HttpEventType.Response) {
          const elapsed = Date.now() - started;
          console.log(`Request to ${req.url} took ${elapsed}ms`);
        }
      }
    })
  );
};

// Conditional interceptor
export const conditionalInterceptor: HttpInterceptorFn = (req, next) => {
  // Skip interceptor for specific URLs
  if (req.url.includes('/public') || req.url.includes('/assets')) {
    return next(req);
  }
  
  // Apply custom logic
  req = req.clone({
    setHeaders: {
      'X-Custom-Header': 'custom-value'
    }
  });
  
  return next(req);
};

// Register in app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([
        loggingInterceptor,
        timingInterceptor,
        baseUrlInterceptor,
        authInterceptor,
        tokenRefreshInterceptor,
        retryInterceptor,
        errorInterceptor,
        cacheInterceptor
      ])
    )
  ]
};
```

---

### Q13: How to modify request and response in interceptors?

**Difficulty:** Hard

**Code:**
```typescript
// Request modification interceptor
@Injectable()
export class RequestModificationInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // 1. Add headers
    let modifiedReq = req.clone({
      setHeaders: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        'X-Request-ID': this.generateRequestId(),
        'X-Client-Version': '1.0.0'
      }
    });
    
    // 2. Add query parameters
    modifiedReq = modifiedReq.clone({
      setParams: {
        timestamp: Date.now().toString(),
        locale: 'en-US'
      }
    });
    
    // 3. Transform request body
    if (req.body && typeof req.body === 'object') {
      const transformedBody = this.transformRequest(req.body);
      modifiedReq = modifiedReq.clone({
        body: transformedBody
      });
    }
    
    // 4. Change URL
    if (!modifiedReq.url.startsWith('http')) {
      modifiedReq = modifiedReq.clone({
        url: `${environment.apiUrl}${modifiedReq.url}`
      });
    }
    
    return next.handle(modifiedReq);
  }
  
  private transformRequest(body: any): any {
    // Convert camelCase to snake_case
    return Object.keys(body).reduce((acc, key) => {
      const snakeKey = key.replace(/[A-Z]/g, letter => `_${letter.toLowerCase()}`);
      acc[snakeKey] = body[key];
      return acc;
    }, {});
  }
  
  private generateRequestId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}

// Response modification interceptor
@Injectable()
export class ResponseModificationInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      map(event => {
        if (event instanceof HttpResponse) {
          // 1. Transform response body
          if (event.body && typeof event.body === 'object') {
            const transformedBody = this.transformResponse(event.body);
            
            return event.clone({
              body: transformedBody
            });
          }
          
          // 2. Add custom properties to response
          if (event.body) {
            return event.clone({
              body: {
                ...event.body,
                _metadata: {
                  timestamp: Date.now(),
                  cached: false
                }
              }
            });
          }
        }
        
        return event;
      })
    );
  }
  
  private transformResponse(body: any): any {
    // Convert snake_case to camelCase
    if (Array.isArray(body)) {
      return body.map(item => this.transformResponse(item));
    }
    
    if (body && typeof body === 'object') {
      return Object.keys(body).reduce((acc, key) => {
        const camelKey = key.replace(/_([a-z])/g, (_, letter) => letter.toUpperCase());
        acc[camelKey] = this.transformResponse(body[key]);
        return acc;
      }, {});
    }
    
    return body;
  }
}

// Request/Response encryption interceptor
@Injectable()
export class EncryptionInterceptor implements HttpInterceptor {
  constructor(private encryptionService: EncryptionService) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    let modifiedReq = req;
    
    // Encrypt request body for sensitive endpoints
    if (req.url.includes('/sensitive') && req.body) {
      const encryptedBody = this.encryptionService.encrypt(JSON.stringify(req.body));
      modifiedReq = req.clone({
        body: { data: encryptedBody },
        setHeaders: {
          'X-Encrypted': 'true'
        }
      });
    }
    
    return next.handle(modifiedReq).pipe(
      map(event => {
        // Decrypt response if encrypted
        if (event instanceof HttpResponse && event.headers.get('X-Encrypted') === 'true') {
          const decryptedBody = this.encryptionService.decrypt(event.body.data);
          return event.clone({
            body: JSON.parse(decryptedBody)
          });
        }
        
        return event;
      })
    );
  }
}

// Response normalization interceptor
@Injectable()
export class NormalizationInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      map(event => {
        if (event instanceof HttpResponse && event.body) {
          // Normalize different API response formats
          let normalizedBody = event.body;
          
          // Format 1: { data: {...}, meta: {...} }
          if (event.body.data && event.body.meta) {
            normalizedBody = {
              ...event.body.data,
              _meta: event.body.meta
            };
          }
          
          // Format 2: { result: {...}, success: true }
          if (event.body.result && event.body.success !== undefined) {
            normalizedBody = event.body.result;
          }
          
          // Format 3: Unwrap single item arrays
          if (Array.isArray(event.body) && event.body.length === 1 && req.url.includes('/single')) {
            normalizedBody = event.body[0];
          }
          
          return event.clone({ body: normalizedBody });
        }
        
        return event;
      })
    );
  }
}
```

---

### Q14: How to handle multiple interceptors order?

**Difficulty:** Medium

**Code:**
```typescript
// Interceptor execution order matters!
// They execute in the order they are provided

// 1. BaseURL Interceptor (runs first - modifies URL)
@Injectable()
export class Base# Angular HTTP - 50 Interview Questions & Code Examples

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