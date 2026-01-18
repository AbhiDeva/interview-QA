# 35 Angular Interview Questions and Coding Exercises on Routing

## Conceptual Questions (1-20)

### Basic Level (1-10)

**1. What is Angular Router and why is it needed?**

Angular Router is a powerful navigation library that enables navigation from one view to another as users perform application tasks. It interprets browser URLs as instructions to navigate to a client-generated view and can pass optional parameters to the supporting view component.

**2. What is the difference between RouterModule.forRoot() and RouterModule.forChild()?**

`forRoot()` is used in the root AppModule to configure the router at the application level and should only be called once. `forChild()` is used in feature modules to add additional routes without re-instantiating the router service.

**3. Explain the role of `<router-outlet>` directive.**

The `<router-outlet>` is a placeholder directive that Angular dynamically fills based on the current router state. It acts as a component placeholder where routed components are rendered.

**4. What are route parameters and how do you access them?**

Route parameters are dynamic values in the URL path (e.g., `/user/:id`). They can be accessed using `ActivatedRoute` service through either `snapshot.paramMap` for one-time access or `paramMap` observable for reactive updates.

**5. What is the difference between paramMap and queryParamMap?**

`paramMap` contains route parameters defined in the route path (e.g., `/user/:id`), while `queryParamMap` contains optional parameters passed in the URL query string (e.g., `/users?page=1&sort=name`).

**6. What are Route Guards in Angular?**

Route Guards are interfaces that allow you to control navigation to and from routes. They include CanActivate, CanActivateChild, CanDeactivate, Resolve, and CanLoad guards.

**7. What is lazy loading in Angular routing?**

Lazy loading is a design pattern that loads feature modules only when they're needed, rather than loading all modules at application startup. This improves initial load time and performance.

**8. What is the purpose of the pathMatch property in route configuration?**

`pathMatch` determines how the router matches a URL to a route path. It has two values: 'full' (entire URL must match) and 'prefix' (URL must start with the path).

**9. How do you implement programmatic navigation in Angular?**

Programmatic navigation is achieved using the `Router` service's `navigate()` or `navigateByUrl()` methods. For example: `this.router.navigate(['/home'])`.

**10. What is a wildcard route and when would you use it?**

A wildcard route uses `**` as the path and matches any URL. It's typically used for 404 pages and should be placed last in the route configuration.

### Intermediate Level (11-20)

**11. Explain child routes and nested routing in Angular.**

Child routes allow you to create hierarchical route structures where components can have their own router-outlet for nested views. They're defined using the `children` property in route configuration.

**12. What is route resolver and when would you use it?**

A Resolver is a service that pre-fetches data before a route is activated, ensuring the component has all necessary data before rendering. It implements the `Resolve` interface.

**13. How does the Location strategy work in Angular?**

Angular supports two location strategies: `PathLocationStrategy` (default, uses HTML5 pushState) and `HashLocationStrategy` (uses URL hash fragment). They determine how URLs are formatted.

**14. What is the difference between navigate() and navigateByUrl()?**

`navigate()` accepts an array of commands and supports relative navigation, while `navigateByUrl()` accepts a complete URL string and always performs absolute navigation.

**15. Explain route data and how to use it.**

Route data is static information attached to a route using the `data` property. It can be accessed via `ActivatedRoute` and is useful for passing configuration or metadata to components.

**16. What are auxiliary routes in Angular?**

Auxiliary routes (named outlets) allow you to display multiple views simultaneously. They're defined with a name and accessed using the `outlet` property in route configuration.

**17. How do you handle route errors and failed navigation?**

Route errors can be handled by subscribing to router events, implementing guards that return observables/promises, or using error handling in resolvers.

**18. What is the purpose of RouteReuseStrategy?**

RouteReuseStrategy allows you to customize route component reuse behavior, determining when components should be destroyed and recreated versus reused during navigation.

**19. Explain the router events lifecycle.**

Router emits events during navigation: NavigationStart, RoutesRecognized, RouteConfigLoadStart, RouteConfigLoadEnd, NavigationEnd, NavigationCancel, NavigationError.

**20. How do you preserve query parameters during navigation?**

Use the `queryParamsHandling` option with values 'merge' or 'preserve' in navigation extras to maintain query parameters across routes.

---

## Coding Exercises (21-35)

### 21. Configure basic routing with multiple components

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { AboutComponent } from './about/about.component';
import { ContactComponent } from './contact/contact.component';

const routes: Routes = [
  { path: '', redirectTo: '/home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: 'contact', component: ContactComponent },
  { path: '**', redirectTo: '/home' }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

### 22. Implement route parameters and access them

```typescript
// app-routing.module.ts
const routes: Routes = [
  { path: 'user/:id', component: UserDetailComponent },
  { path: 'product/:id/:category', component: ProductComponent }
];

// user-detail.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-user-detail',
  template: `<h2>User ID: {{ userId }}</h2>`
})
export class UserDetailComponent implements OnInit {
  userId: string;

  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    // Snapshot approach
    this.userId = this.route.snapshot.paramMap.get('id');
    
    // Observable approach (reactive)
    this.route.paramMap.subscribe(params => {
      this.userId = params.get('id');
    });
  }
}
```

### 23. Implement query parameters

```typescript
// product-list.component.ts
import { Component } from '@angular/core';
import { Router, ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-product-list',
  template: `
    <button (click)="filterProducts()">Filter Products</button>
    <p>Category: {{ category }}, Page: {{ page }}</p>
  `
})
export class ProductListComponent implements OnInit {
  category: string;
  page: number;

  constructor(
    private router: Router,
    private route: ActivatedRoute
  ) {}

  ngOnInit() {
    this.route.queryParamMap.subscribe(params => {
      this.category = params.get('category');
      this.page = +params.get('page');
    });
  }

  filterProducts() {
    this.router.navigate(['/products'], {
      queryParams: { category: 'electronics', page: 1 }
    });
  }
}
```

### 24. Create a CanActivate guard

```typescript
// auth.guard.ts
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot, Router } from '@angular/router';
import { AuthService } from './auth.service';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): boolean {
    if (this.authService.isLoggedIn()) {
      return true;
    }
    
    this.router.navigate(['/login'], {
      queryParams: { returnUrl: state.url }
    });
    return false;
  }
}

// app-routing.module.ts
const routes: Routes = [
  {
    path: 'dashboard',
    component: DashboardComponent,
    canActivate: [AuthGuard]
  }
];
```

### 25. Implement lazy loading

```typescript
// app-routing.module.ts
const routes: Routes = [
  { path: 'home', component: HomeComponent },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module')
      .then(m => m.AdminModule)
  },
  {
    path: 'products',
    loadChildren: () => import('./products/products.module')
      .then(m => m.ProductsModule)
  }
];

// admin-routing.module.ts (in admin folder)
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { AdminDashboardComponent } from './admin-dashboard/admin-dashboard.component';

const routes: Routes = [
  { path: '', component: AdminDashboardComponent },
  { path: 'users', component: AdminUsersComponent }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class AdminRoutingModule { }
```

### 26. Create nested/child routes

```typescript
// app-routing.module.ts
const routes: Routes = [
  {
    path: 'products',
    component: ProductsComponent,
    children: [
      { path: '', component: ProductListComponent },
      { path: ':id', component: ProductDetailComponent },
      { path: ':id/edit', component: ProductEditComponent }
    ]
  }
];

// products.component.ts
@Component({
  selector: 'app-products',
  template: `
    <h1>Products</h1>
    <nav>
      <a routerLink="./list">List</a>
    </nav>
    <router-outlet></router-outlet>
  `
})
export class ProductsComponent { }
```

### 27. Implement CanDeactivate guard

```typescript
// can-deactivate.guard.ts
export interface CanComponentDeactivate {
  canDeactivate: () => boolean | Promise<boolean>;
}

@Injectable({
  providedIn: 'root'
})
export class CanDeactivateGuard implements CanDeactivate<CanComponentDeactivate> {
  canDeactivate(
    component: CanComponentDeactivate
  ): boolean | Promise<boolean> {
    return component.canDeactivate ? component.canDeactivate() : true;
  }
}

// form.component.ts
@Component({
  selector: 'app-form',
  template: `<form><!-- form fields --></form>`
})
export class FormComponent implements CanComponentDeactivate {
  hasUnsavedChanges = false;

  canDeactivate(): boolean {
    if (this.hasUnsavedChanges) {
      return confirm('You have unsaved changes. Do you want to leave?');
    }
    return true;
  }
}

// routing config
{
  path: 'form',
  component: FormComponent,
  canDeactivate: [CanDeactivateGuard]
}
```

### 28. Create a route resolver

```typescript
// user.resolver.ts
import { Injectable } from '@angular/core';
import { Resolve, ActivatedRouteSnapshot } from '@angular/router';
import { Observable } from 'rxjs';
import { UserService } from './user.service';
import { User } from './user.model';

@Injectable({
  providedIn: 'root'
})
export class UserResolver implements Resolve<User> {
  constructor(private userService: UserService) {}

  resolve(route: ActivatedRouteSnapshot): Observable<User> {
    const userId = route.paramMap.get('id');
    return this.userService.getUser(userId);
  }
}

// app-routing.module.ts
const routes: Routes = [
  {
    path: 'user/:id',
    component: UserComponent,
    resolve: { user: UserResolver }
  }
];

// user.component.ts
ngOnInit() {
  this.route.data.subscribe(data => {
    this.user = data['user'];
  });
}
```

### 29. Programmatic navigation with extras

```typescript
import { Component } from '@angular/core';
import { Router, NavigationExtras } from '@angular/router';

@Component({
  selector: 'app-navigation-example',
  template: `
    <button (click)="navigateToUser()">Go to User</button>
    <button (click)="navigateWithQuery()">Search Products</button>
  `
})
export class NavigationExampleComponent {
  constructor(private router: Router) {}

  navigateToUser() {
    // Simple navigation
    this.router.navigate(['/user', 123]);
  }

  navigateWithQuery() {
    const navigationExtras: NavigationExtras = {
      queryParams: { category: 'books', sort: 'price' },
      fragment: 'results'
    };
    this.router.navigate(['/products'], navigationExtras);
  }

  navigateRelative() {
    // Relative navigation
    this.router.navigate(['../sibling'], { relativeTo: this.route });
  }
}
```

### 30. Implement route data and access it

```typescript
// app-routing.module.ts
const routes: Routes = [
  {
    path: 'admin',
    component: AdminComponent,
    data: { title: 'Admin Panel', roles: ['admin', 'superuser'] }
  },
  {
    path: 'user',
    component: UserComponent,
    data: { title: 'User Profile', breadcrumb: 'Profile' }
  }
];

// admin.component.ts
@Component({
  selector: 'app-admin',
  template: `<h1>{{ pageTitle }}</h1>`
})
export class AdminComponent implements OnInit {
  pageTitle: string;
  allowedRoles: string[];

  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    this.route.data.subscribe(data => {
      this.pageTitle = data['title'];
      this.allowedRoles = data['roles'];
    });
  }
}
```

### 31. Create auxiliary/named routes

```typescript
// app-routing.module.ts
const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { path: 'chat', component: ChatComponent, outlet: 'sidebar' },
  { path: 'help', component: HelpComponent, outlet: 'popup' }
];

// app.component.html
<div class="main-content">
  <router-outlet></router-outlet>
</div>
<div class="sidebar">
  <router-outlet name="sidebar"></router-outlet>
</div>
<div class="popup">
  <router-outlet name="popup"></router-outlet>
</div>

// navigation.component.ts
openChat() {
  this.router.navigate([
    { outlets: { sidebar: ['chat'] } }
  ]);
}

openHelp() {
  this.router.navigate([
    { outlets: { primary: 'home', popup: 'help' } }
  ]);
}

closePopup() {
  this.router.navigate([{ outlets: { popup: null } }]);
}
```

### 32. Listen to router events

```typescript
import { Component, OnInit } from '@angular/core';
import { Router, NavigationStart, NavigationEnd, NavigationError } from '@angular/router';
import { filter } from 'rxjs/operators';

@Component({
  selector: 'app-root',
  template: `
    <div *ngIf="loading" class="spinner">Loading...</div>
    <router-outlet></router-outlet>
  `
})
export class AppComponent implements OnInit {
  loading = false;

  constructor(private router: Router) {}

  ngOnInit() {
    this.router.events.subscribe(event => {
      if (event instanceof NavigationStart) {
        this.loading = true;
        console.log('Navigation started:', event.url);
      }
      
      if (event instanceof NavigationEnd) {
        this.loading = false;
        console.log('Navigation ended:', event.url);
      }
      
      if (event instanceof NavigationError) {
        this.loading = false;
        console.error('Navigation error:', event.error);
      }
    });

    // Filter for specific events
    this.router.events.pipe(
      filter(event => event instanceof NavigationEnd)
    ).subscribe((event: NavigationEnd) => {
      // Track page views, update analytics, etc.
      console.log('Page loaded:', event.urlAfterRedirects);
    });
  }
}
```

### 33. Implement CanActivateChild guard

```typescript
// admin-guard.service.ts
import { Injectable } from '@angular/core';
import { CanActivateChild, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import { AuthService } from './auth.service';

@Injectable({
  providedIn: 'root'
})
export class AdminGuard implements CanActivateChild {
  constructor(private authService: AuthService) {}

  canActivateChild(
    childRoute: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): boolean {
    return this.authService.hasAdminRole();
  }
}

// app-routing.module.ts
const routes: Routes = [
  {
    path: 'admin',
    component: AdminComponent,
    canActivateChild: [AdminGuard],
    children: [
      { path: 'users', component: UsersComponent },
      { path: 'settings', component: SettingsComponent },
      { path: 'reports', component: ReportsComponent }
    ]
  }
];
```

### 34. Preserve and merge query parameters

```typescript
@Component({
  selector: 'app-product-filter',
  template: `
    <button (click)="updateCategory()">Update Category</button>
    <button (click)="addSort()">Add Sort</button>
  `
})
export class ProductFilterComponent {
  constructor(private router: Router, private route: ActivatedRoute) {}

  updateCategory() {
    // Replace all query params
    this.router.navigate(['/products'], {
      queryParams: { category: 'electronics' }
    });
  }

  addSort() {
    // Merge with existing query params
    this.router.navigate(['/products'], {
      queryParams: { sort: 'price' },
      queryParamsHandling: 'merge'
    });
  }

  preserveParams() {
    // Preserve existing query params
    this.router.navigate(['/products/detail'], {
      queryParamsHandling: 'preserve'
    });
  }

  navigateWithFragment() {
    this.router.navigate(['/products'], {
      queryParams: { page: 2 },
      queryParamsHandling: 'merge',
      fragment: 'top',
      preserveFragment: false
    });
  }
}
```

### 35. Implement CanLoad guard for lazy-loaded modules

```typescript
// can-load.guard.ts
import { Injectable } from '@angular/core';
import { CanLoad, Route, UrlSegment, Router } from '@angular/core';
import { AuthService } from './auth.service';

@Injectable({
  providedIn: 'root'
})
export class CanLoadGuard implements CanLoad {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}

  canLoad(route: Route, segments: UrlSegment[]): boolean {
    if (this.authService.isAuthenticated() && 
        this.authService.hasPermission(route.data?.permission)) {
      return true;
    }
    
    this.router.navigate(['/unauthorized']);
    return false;
  }
}

// app-routing.module.ts
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module')
      .then(m => m.AdminModule),
    canLoad: [CanLoadGuard],
    data: { permission: 'admin' }
  },
  {
    path: 'premium',
    loadChildren: () => import('./premium/premium.module')
      .then(m => m.PremiumModule),
    canLoad: [CanLoadGuard],
    data: { permission: 'premium-user' }
  }
];
```

---

## Summary

These questions and coding exercises cover the complete spectrum of Angular routing:

- **Basic Concepts**: Router setup, outlets, parameters
- **Navigation**: Programmatic navigation, query parameters, fragments
- **Guards**: CanActivate, CanDeactivate, CanActivateChild, CanLoad
- **Advanced Features**: Lazy loading, resolvers, auxiliary routes, route events
- **Best Practices**: Route data, parameter handling, navigation extras

Practice these to master Angular routing for interviews and real-world applications!