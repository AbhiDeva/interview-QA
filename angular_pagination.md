# Angular Pagination Techniques - Complete Guide

A comprehensive guide to implementing different pagination patterns in Angular applications.

---

## Table of Contents

1. [Basic Client-Side Pagination](#1-basic-client-side-pagination)
2. [Server-Side Pagination](#2-server-side-pagination)
3. [Infinite Scroll Pagination](#3-infinite-scroll-pagination)
4. [Virtual Scroll Pagination](#4-virtual-scroll-pagination)
5. [Load More Button Pagination](#5-load-more-button-pagination)
6. [Cursor-Based Pagination](#6-cursor-based-pagination)
7. [Using Angular Material Paginator](#7-using-angular-material-paginator)
8. [NgRx State Management Pagination](#8-ngrx-state-management-pagination)
9. [Reusable Pagination Component](#9-reusable-pagination-component)
10. [Advanced Techniques](#10-advanced-techniques)

---

## 1. Basic Client-Side Pagination

### 1.1 Simple Client-Side Pagination

**Component:**

```typescript
import { Component, OnInit } from '@angular/core';

interface User {
  id: number;
  name: string;
  email: string;
  role: string;
}

@Component({
  selector: 'app-client-pagination',
  templateUrl: './client-pagination.component.html',
  styleUrls: ['./client-pagination.component.scss']
})
export class ClientPaginationComponent implements OnInit {
  // All data
  allUsers: User[] = [];
  
  // Paginated data
  paginatedUsers: User[] = [];
  
  // Pagination settings
  currentPage = 1;
  itemsPerPage = 10;
  totalPages = 0;
  
  ngOnInit() {
    this.loadAllData();
    this.updatePaginatedData();
  }
  
  loadAllData() {
    // Simulate loading all data at once
    this.allUsers = Array.from({ length: 100 }, (_, i) => ({
      id: i + 1,
      name: `User ${i + 1}`,
      email: `user${i + 1}@example.com`,
      role: i % 3 === 0 ? 'Admin' : 'User'
    }));
    
    this.totalPages = Math.ceil(this.allUsers.length / this.itemsPerPage);
  }
  
  updatePaginatedData() {
    const startIndex = (this.currentPage - 1) * this.itemsPerPage;
    const endIndex = startIndex + this.itemsPerPage;
    this.paginatedUsers = this.allUsers.slice(startIndex, endIndex);
  }
  
  goToPage(page: number) {
    if (page >= 1 && page <= this.totalPages) {
      this.currentPage = page;
      this.updatePaginatedData();
    }
  }
  
  nextPage() {
    this.goToPage(this.currentPage + 1);
  }
  
  previousPage() {
    this.goToPage(this.currentPage - 1);
  }
  
  changeItemsPerPage(itemsPerPage: number) {
    this.itemsPerPage = itemsPerPage;
    this.currentPage = 1;
    this.totalPages = Math.ceil(this.allUsers.length / this.itemsPerPage);
    this.updatePaginatedData();
  }
  
  get pageNumbers(): number[] {
    return Array.from({ length: this.totalPages }, (_, i) => i + 1);
  }
  
  get startIndex(): number {
    return (this.currentPage - 1) * this.itemsPerPage + 1;
  }
  
  get endIndex(): number {
    return Math.min(this.currentPage * this.itemsPerPage, this.allUsers.length);
  }
}
```

**Template:**

```html
<!-- client-pagination.component.html -->
<div class="pagination-container">
  <!-- Items per page selector -->
  <div class="items-per-page">
    <label>Items per page:</label>
    <select (change)="changeItemsPerPage(+$event.target.value)">
      <option value="10">10</option>
      <option value="25">25</option>
      <option value="50">50</option>
      <option value="100">100</option>
    </select>
  </div>

  <!-- Data table -->
  <table>
    <thead>
      <tr>
        <th>ID</th>
        <th>Name</th>
        <th>Email</th>
        <th>Role</th>
      </tr>
    </thead>
    <tbody>
      <tr *ngFor="let user of paginatedUsers">
        <td>{{ user.id }}</td>
        <td>{{ user.name }}</td>
        <td>{{ user.email }}</td>
        <td>{{ user.role }}</td>
      </tr>
    </tbody>
  </table>

  <!-- Pagination info -->
  <div class="pagination-info">
    Showing {{ startIndex }} to {{ endIndex }} of {{ allUsers.length }} entries
  </div>

  <!-- Pagination controls -->
  <div class="pagination-controls">
    <button 
      (click)="previousPage()" 
      [disabled]="currentPage === 1">
      Previous
    </button>
    
    <button 
      *ngFor="let page of pageNumbers" 
      (click)="goToPage(page)"
      [class.active]="currentPage === page">
      {{ page }}
    </button>
    
    <button 
      (click)="nextPage()" 
      [disabled]="currentPage === totalPages">
      Next
    </button>
  </div>
</div>
```

### 1.2 Client-Side Pagination with Pipe

**Pagination Pipe:**

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'paginate'
})
export class PaginatePipe implements PipeTransform {
  transform(items: any[], page: number, pageSize: number): any[] {
    if (!items || !items.length) return [];
    
    const startIndex = (page - 1) * pageSize;
    const endIndex = startIndex + pageSize;
    
    return items.slice(startIndex, endIndex);
  }
}
```

**Component using Pipe:**

```typescript
@Component({
  selector: 'app-pipe-pagination',
  template: `
    <div class="user-list">
      <div *ngFor="let user of users | paginate:currentPage:pageSize">
        {{ user.name }} - {{ user.email }}
      </div>
    </div>
    
    <div class="pagination">
      <button (click)="previousPage()">Previous</button>
      <span>Page {{ currentPage }} of {{ totalPages }}</span>
      <button (click)="nextPage()">Next</button>
    </div>
  `
})
export class PipePaginationComponent {
  users: User[] = [];
  currentPage = 1;
  pageSize = 10;
  
  get totalPages(): number {
    return Math.ceil(this.users.length / this.pageSize);
  }
  
  nextPage() {
    if (this.currentPage < this.totalPages) {
      this.currentPage++;
    }
  }
  
  previousPage() {
    if (this.currentPage > 1) {
      this.currentPage--;
    }
  }
}
```

---

## 2. Server-Side Pagination

### 2.1 Backend API Service

```typescript
import { Injectable } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';

export interface PaginatedResponse<T> {
  data: T[];
  page: number;
  pageSize: number;
  totalItems: number;
  totalPages: number;
}

@Injectable({
  providedIn: 'root'
})
export class UserApiService {
  private apiUrl = 'https://api.example.com/users';
  
  constructor(private http: HttpClient) {}
  
  getUsers(page: number, pageSize: number, sortBy?: string, sortOrder?: string): Observable<PaginatedResponse<User>> {
    let params = new HttpParams()
      .set('page', page.toString())
      .set('pageSize', pageSize.toString());
    
    if (sortBy) {
      params = params.set('sortBy', sortBy);
    }
    
    if (sortOrder) {
      params = params.set('sortOrder', sortOrder);
    }
    
    return this.http.get<PaginatedResponse<User>>(this.apiUrl, { params });
  }
  
  searchUsers(searchTerm: string, page: number, pageSize: number): Observable<PaginatedResponse<User>> {
    const params = new HttpParams()
      .set('search', searchTerm)
      .set('page', page.toString())
      .set('pageSize', pageSize.toString());
    
    return this.http.get<PaginatedResponse<User>>(this.apiUrl, { params });
  }
}
```

### 2.2 Server-Side Pagination Component

```typescript
import { Component, OnInit } from '@angular/core';
import { debounceTime, distinctUntilChanged, switchMap } from 'rxjs/operators';
import { Subject } from 'rxjs';

@Component({
  selector: 'app-server-pagination',
  templateUrl: './server-pagination.component.html'
})
export class ServerPaginationComponent implements OnInit {
  users: User[] = [];
  
  // Pagination state
  currentPage = 1;
  pageSize = 10;
  totalItems = 0;
  totalPages = 0;
  
  // Sorting
  sortBy = 'name';
  sortOrder: 'asc' | 'desc' = 'asc';
  
  // Search
  searchTerm = '';
  private searchSubject = new Subject<string>();
  
  // Loading state
  loading = false;
  error: string | null = null;
  
  constructor(private userApi: UserApiService) {}
  
  ngOnInit() {
    this.loadUsers();
    this.setupSearch();
  }
  
  loadUsers() {
    this.loading = true;
    this.error = null;
    
    this.userApi.getUsers(this.currentPage, this.pageSize, this.sortBy, this.sortOrder)
      .subscribe({
        next: (response) => {
          this.users = response.data;
          this.totalItems = response.totalItems;
          this.totalPages = response.totalPages;
          this.loading = false;
        },
        error: (error) => {
          this.error = 'Failed to load users';
          this.loading = false;
          console.error(error);
        }
      });
  }
  
  setupSearch() {
    this.searchSubject.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(term => {
        this.loading = true;
        return this.userApi.searchUsers(term, this.currentPage, this.pageSize);
      })
    ).subscribe({
      next: (response) => {
        this.users = response.data;
        this.totalItems = response.totalItems;
        this.totalPages = response.totalPages;
        this.loading = false;
      },
      error: (error) => {
        this.error = 'Search failed';
        this.loading = false;
      }
    });
  }
  
  onSearch(term: string) {
    this.searchTerm = term;
    this.currentPage = 1;
    this.searchSubject.next(term);
  }
  
  goToPage(page: number) {
    if (page >= 1 && page <= this.totalPages && page !== this.currentPage) {
      this.currentPage = page;
      this.loadUsers();
    }
  }
  
  changePageSize(size: number) {
    this.pageSize = size;
    this.currentPage = 1;
    this.loadUsers();
  }
  
  sortByColumn(column: string) {
    if (this.sortBy === column) {
      this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
    } else {
      this.sortBy = column;
      this.sortOrder = 'asc';
    }
    this.loadUsers();
  }
  
  get pageNumbers(): number[] {
    const maxVisible = 5;
    const pages: number[] = [];
    
    let startPage = Math.max(1, this.currentPage - Math.floor(maxVisible / 2));
    let endPage = Math.min(this.totalPages, startPage + maxVisible - 1);
    
    if (endPage - startPage < maxVisible - 1) {
      startPage = Math.max(1, endPage - maxVisible + 1);
    }
    
    for (let i = startPage; i <= endPage; i++) {
      pages.push(i);
    }
    
    return pages;
  }
}
```

**Template:**

```html
<!-- server-pagination.component.html -->
<div class="server-pagination-container">
  <!-- Search -->
  <div class="search-box">
    <input 
      type="text" 
      placeholder="Search users..."
      [ngModel]="searchTerm"
      (ngModelChange)="onSearch($event)">
  </div>

  <!-- Loading indicator -->
  <div *ngIf="loading" class="loading">
    Loading...
  </div>

  <!-- Error message -->
  <div *ngIf="error" class="error">
    {{ error }}
  </div>

  <!-- Data table -->
  <table *ngIf="!loading && !error">
    <thead>
      <tr>
        <th (click)="sortByColumn('id')">
          ID
          <span *ngIf="sortBy === 'id'">
            {{ sortOrder === 'asc' ? '↑' : '↓' }}
          </span>
        </th>
        <th (click)="sortByColumn('name')">
          Name
          <span *ngIf="sortBy === 'name'">
            {{ sortOrder === 'asc' ? '↑' : '↓' }}
          </span>
        </th>
        <th (click)="sortByColumn('email')">
          Email
          <span *ngIf="sortBy === 'email'">
            {{ sortOrder === 'asc' ? '↑' : '↓' }}
          </span>
        </th>
      </tr>
    </thead>
    <tbody>
      <tr *ngFor="let user of users">
        <td>{{ user.id }}</td>
        <td>{{ user.name }}</td>
        <td>{{ user.email }}</td>
      </tr>
    </tbody>
  </table>

  <!-- Pagination controls -->
  <div class="pagination-controls" *ngIf="totalPages > 0">
    <div class="page-size-selector">
      <label>Show:</label>
      <select [ngModel]="pageSize" (ngModelChange)="changePageSize(+$event)">
        <option value="10">10</option>
        <option value="25">25</option>
        <option value="50">50</option>
        <option value="100">100</option>
      </select>
    </div>

    <div class="page-navigation">
      <button 
        (click)="goToPage(1)" 
        [disabled]="currentPage === 1">
        First
      </button>
      
      <button 
        (click)="goToPage(currentPage - 1)" 
        [disabled]="currentPage === 1">
        Previous
      </button>
      
      <button 
        *ngFor="let page of pageNumbers"
        (click)="goToPage(page)"
        [class.active]="currentPage === page">
        {{ page }}
      </button>
      
      <button 
        (click)="goToPage(currentPage + 1)" 
        [disabled]="currentPage === totalPages">
        Next
      </button>
      
      <button 
        (click)="goToPage(totalPages)" 
        [disabled]="currentPage === totalPages">
        Last
      </button>
    </div>

    <div class="pagination-info">
      Page {{ currentPage }} of {{ totalPages }} ({{ totalItems }} total items)
    </div>
  </div>
</div>
```

---

## 3. Infinite Scroll Pagination

### 3.1 Using Intersection Observer

```typescript
import { Component, OnInit, HostListener, ElementRef, ViewChild } from '@angular/core';

@Component({
  selector: 'app-infinite-scroll',
  templateUrl: './infinite-scroll.component.html'
})
export class InfiniteScrollComponent implements OnInit {
  items: any[] = [];
  currentPage = 1;
  pageSize = 20;
  loading = false;
  hasMore = true;
  
  @ViewChild('sentinel') sentinel!: ElementRef;
  private observer!: IntersectionObserver;
  
  constructor(private dataService: DataService) {}
  
  ngOnInit() {
    this.loadMore();
    this.setupIntersectionObserver();
  }
  
  ngAfterViewInit() {
    if (this.sentinel) {
      this.observer.observe(this.sentinel.nativeElement);
    }
  }
  
  setupIntersectionObserver() {
    const options = {
      root: null,
      rootMargin: '100px',
      threshold: 0.1
    };
    
    this.observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting && !this.loading && this.hasMore) {
          this.loadMore();
        }
      });
    }, options);
  }
  
  loadMore() {
    if (this.loading || !this.hasMore) return;
    
    this.loading = true;
    
    this.dataService.getItems(this.currentPage, this.pageSize)
      .subscribe({
        next: (response) => {
          this.items = [...this.items, ...response.data];
          this.hasMore = response.data.length === this.pageSize;
          this.currentPage++;
          this.loading = false;
        },
        error: (error) => {
          console.error('Failed to load items', error);
          this.loading = false;
        }
      });
  }
  
  ngOnDestroy() {
    if (this.observer) {
      this.observer.disconnect();
    }
  }
}
```

**Template:**

```html
<!-- infinite-scroll.component.html -->
<div class="infinite-scroll-container">
  <div class="items-grid">
    <div class="item-card" *ngFor="let item of items">
      <h3>{{ item.title }}</h3>
      <p>{{ item.description }}</p>
    </div>
  </div>

  <!-- Sentinel element for intersection observer -->
  <div #sentinel class="sentinel"></div>

  <!-- Loading indicator -->
  <div *ngIf="loading" class="loading">
    <div class="spinner"></div>
    <p>Loading more items...</p>
  </div>

  <!-- End of list message -->
  <div *ngIf="!hasMore && !loading" class="end-message">
    No more items to load
  </div>
</div>
```

### 3.2 Using Scroll Event

```typescript
@Component({
  selector: 'app-scroll-pagination',
  templateUrl: './scroll-pagination.component.html'
})
export class ScrollPaginationComponent implements OnInit {
  items: any[] = [];
  currentPage = 1;
  pageSize = 20;
  loading = false;
  hasMore = true;
  
  @HostListener('window:scroll', ['$event'])
  onScroll() {
    const scrollPosition = window.pageYOffset + window.innerHeight;
    const pageHeight = document.documentElement.scrollHeight;
    
    // Load more when user is 200px from bottom
    if (scrollPosition >= pageHeight - 200 && !this.loading && this.hasMore) {
      this.loadMore();
    }
  }
  
  ngOnInit() {
    this.loadMore();
  }
  
  loadMore() {
    this.loading = true;
    
    this.dataService.getItems(this.currentPage, this.pageSize)
      .subscribe({
        next: (response) => {
          this.items = [...this.items, ...response.data];
          this.hasMore = response.data.length === this.pageSize;
          this.currentPage++;
          this.loading = false;
        },
        error: () => {
          this.loading = false;
        }
      });
  }
}
```

---

## 4. Virtual Scroll Pagination

### 4.1 Using Angular CDK Virtual Scroll

**Install CDK:**
```bash
npm install @angular/cdk
```

**Module:**

```typescript
import { ScrollingModule } from '@angular/cdk/scrolling';

@NgModule({
  imports: [
    ScrollingModule
  ]
})
export class AppModule { }
```

**Component:**

```typescript
import { Component, OnInit } from '@angular/core';
import { CdkVirtualScrollViewport } from '@angular/cdk/scrolling';

@Component({
  selector: 'app-virtual-scroll',
  templateUrl: './virtual-scroll.component.html',
  styleUrls: ['./virtual-scroll.component.scss']
})
export class VirtualScrollComponent implements OnInit {
  items: any[] = [];
  loading = false;
  
  ngOnInit() {
    // Load initial large dataset
    this.loadItems();
  }
  
  loadItems() {
    this.loading = true;
    
    // Simulate loading large dataset
    this.items = Array.from({ length: 10000 }, (_, i) => ({
      id: i + 1,
      title: `Item ${i + 1}`,
      description: `Description for item ${i + 1}`
    }));
    
    this.loading = false;
  }
  
  trackByFn(index: number, item: any) {
    return item.id;
  }
}
```

**Template:**

```html
<!-- virtual-scroll.component.html -->
<div class="virtual-scroll-container">
  <cdk-virtual-scroll-viewport 
    itemSize="80" 
    class="viewport">
    
    <div 
      *cdkVirtualFor="let item of items; trackBy: trackByFn" 
      class="item">
      <h3>{{ item.title }}</h3>
      <p>{{ item.description }}</p>
    </div>
    
  </cdk-virtual-scroll-viewport>
</div>
```

**Styles:**

```scss
// virtual-scroll.component.scss
.virtual-scroll-container {
  height: 100vh;
  
  .viewport {
    height: 600px;
    border: 1px solid #ccc;
    
    .item {
      height: 80px;
      padding: 16px;
      border-bottom: 1px solid #eee;
      
      h3 {
        margin: 0 0 8px 0;
        font-size: 16px;
      }
      
      p {
        margin: 0;
        color: #666;
        font-size: 14px;
      }
    }
  }
}
```

### 4.2 Virtual Scroll with Dynamic Loading

```typescript
@Component({
  selector: 'app-virtual-dynamic',
  template: `
    <cdk-virtual-scroll-viewport 
      itemSize="80" 
      (scrolledIndexChange)="onScrollIndexChange($event)"
      class="viewport">
      
      <div 
        *cdkVirtualFor="let item of items; trackBy: trackByFn" 
        class="item">
        {{ item.title }}
      </div>
      
      <div *ngIf="loading" class="loading">Loading...</div>
    </cdk-virtual-scroll-viewport>
  `
})
export class VirtualDynamicComponent {
  items: any[] = [];
  loading = false;
  currentPage = 1;
  pageSize = 50;
  
  ngOnInit() {
    this.loadMore();
  }
  
  onScrollIndexChange(index: number) {
    const threshold = this.items.length - 10;
    
    if (index >= threshold && !this.loading) {
      this.loadMore();
    }
  }
  
  loadMore() {
    this.loading = true;
    
    this.dataService.getItems(this.currentPage, this.pageSize)
      .subscribe(response => {
        this.items = [...this.items, ...response.data];
        this.currentPage++;
        this.loading = false;
      });
  }
  
  trackByFn(index: number, item: any) {
    return item.id;
  }
}
```

---

## 5. Load More Button Pagination

```typescript
@Component({
  selector: 'app-load-more',
  templateUrl: './load-more.component.html'
})
export class LoadMoreComponent implements OnInit {
  items: any[] = [];
  currentPage = 1;
  pageSize = 10;
  loading = false;
  hasMore = true;
  totalItems = 0;
  
  ngOnInit() {
    this.loadItems();
  }
  
  loadItems() {
    this.loading = true;
    
    this.dataService.getItems(this.currentPage, this.pageSize)
      .subscribe({
        next: (response) => {
          this.items = [...this.items, ...response.data];
          this.totalItems = response.totalItems;
          this.hasMore = this.items.length < this.totalItems;
          this.currentPage++;
          this.loading = false;
        },
        error: () => {
          this.loading = false;
        }
      });
  }
  
  loadMore() {
    if (!this.loading && this.hasMore) {
      this.loadItems();
    }
  }
  
  reset() {
    this.items = [];
    this.currentPage = 1;
    this.hasMore = true;
    this.loadItems();
  }
}
```

**Template:**

```html
<!-- load-more.component.html -->
<div class="load-more-container">
  <div class="items-list">
    <div class="item" *ngFor="let item of items">
      <h3>{{ item.title }}</h3>
      <p>{{ item.description }}</p>
    </div>
  </div>

  <!-- Load more button -->
  <div class="load-more-section">
    <div class="items-info">
      Showing {{ items.length }} of {{ totalItems }} items
    </div>
    
    <button 
      *ngIf="hasMore"
      (click)="loadMore()"
      [disabled]="loading"
      class="load-more-btn">
      {{ loading ? 'Loading...' : 'Load More' }}
    </button>
    
    <div *ngIf="!hasMore" class="all-loaded">
      All items loaded
    </div>
  </div>

  <!-- Reset button -->
  <button (click)="reset()" class="reset-btn">
    Reset
  </button>
</div>
```

---

## 6. Cursor-Based Pagination

### 6.1 Cursor Pagination Service

```typescript
export interface CursorPaginatedResponse<T> {
  data: T[];
  nextCursor: string | null;
  prevCursor: string | null;
  hasMore: boolean;
}

@Injectable({
  providedIn: 'root'
})
export class CursorPaginationService {
  private apiUrl = 'https://api.example.com/items';
  
  constructor(private http: HttpClient) {}
  
  getItems(cursor: string | null = null, limit: number = 20): Observable<CursorPaginatedResponse<any>> {
    let params = new HttpParams().set('limit', limit.toString());
    
    if (cursor) {
      params = params.set('cursor', cursor);
    }
    
    return this.http.get<CursorPaginatedResponse<any>>(this.apiUrl, { params });
  }
}
```

### 6.2 Cursor Pagination Component

```typescript
@Component({
  selector: 'app-cursor-pagination',
  templateUrl: './cursor-pagination.component.html'
})
export class CursorPaginationComponent implements OnInit {
  items: any[] = [];
  nextCursor: string | null = null;
  prevCursor: string | null = null;
  hasMore = false;
  loading = false;
  
  // History for back navigation
  private cursorHistory: (string | null)[] = [null];
  private currentHistoryIndex = 0;
  
  constructor(private cursorService: CursorPaginationService) {}
  
  ngOnInit() {
    this.loadItems();
  }
  
  loadItems(cursor: string | null = null) {
    this.loading = true;
    
    this.cursorService.getItems(cursor, 20)
      .subscribe({
        next: (response) => {
          this.items = response.data;
          this.nextCursor = response.nextCursor;
          this.prevCursor = response.prevCursor;
          this.hasMore = response.hasMore;
          this.loading = false;
        },
        error: () => {
          this.loading = false;
        }
      });
  }
  
  next() {
    if (this.nextCursor && !this.loading) {
      this.currentHistoryIndex++;
      
      if (this.currentHistoryIndex >= this.cursorHistory.length) {
        this.cursorHistory.push(this.nextCursor);
      }
      
      this.loadItems(this.nextCursor);
    }
  }
  
  previous() {
    if (this.currentHistoryIndex > 0 && !this.loading) {
      this.currentHistoryIndex--;
      const cursor = this.cursorHistory[this.currentHistoryIndex];
      this.loadItems(cursor);
    }
  }
  
  get canGoBack(): boolean {
    return this.currentHistoryIndex > 0;
  }
  
  get canGoForward(): boolean {
    return this.hasMore && this.nextCursor !== null;
  }
}
```

**Template:**

```html
<!-- cursor-pagination.component.html -->
<div class="cursor-pagination-container">
  <div class="items-grid">
    <div *ngFor="let item of items" class="item-card">
      <h3>{{ item.title }}</h3>
      <p>{{ item.description }}</p>
    </div>
  </div>

  <div *ngIf="loading" class="loading">
    Loading...
  </div>

  <div class="cursor-controls">
    <button 
      (click)="previous()" 
      [disabled]="!canGoBack || loading">
      ← Previous
    </button>
    
    <button 
      (click)="next()" 
      [disabled]="!canGoForward || loading">
      Next →
    </button>
  </div>
</div>
```

---

## 7. Using Angular Material Paginator

### 7.1 Installation

```bash
npm install @angular/material @angular/cdk
```

### 7.2 Material Paginator Setup

**Module:**

```typescript
import { MatPaginatorModule } from '@angular/material/paginator';
import { MatTableModule } from '@angular/material/table';

@NgModule({
  imports: [
    MatPaginatorModule,
    MatTableModule
  ]
})
export class AppModule { }
```

**Component:**

```typescript
import { Component, OnInit, ViewChild } from '@angular/core';
import { MatPaginator, PageEvent } from '@angular/material/paginator';
import { MatTableDataSource } from '@angular/material/table';

@Component({
  selector: 'app-material-pagination',
  templateUrl: './material-pagination.component.html',
  styleUrls: ['./material-pagination.component.scss']
})
export class MaterialPaginationComponent implements OnInit {
  displayedColumns: string[] = ['id', 'name', 'email', 'role'];
  dataSource!: MatTableDataSource<User>;
  
  @ViewChild(MatPaginator) paginator!: MatPaginator;
  
  // Pagination settings
  totalItems = 0;
  pageSize = 10;
  pageSizeOptions = [5, 10, 25, 50, 100];
  currentPage = 0;
  
  constructor(private userService: UserService) {}
  
  ngOnInit() {
    this.loadUsers();
  }
  
  ngAfterViewInit() {
    if (this.dataSource) {
      this.dataSource.paginator = this.paginator;
    }
  }
  
  loadUsers() {
    this.userService.getAllUsers().subscribe(users => {
      this.dataSource = new MatTableDataSource(users);
      this.dataSource.paginator = this.paginator;
      this.totalItems = users.length;
    });
  }
  
  onPageChange(event: PageEvent) {
    console.log('Page event:', event);
    this.currentPage = event.pageIndex;
    this.pageSize = event.pageSize;
    
    // For server-side pagination:
    // this.loadUsersFromServer(event.pageIndex + 1, event.pageSize);
  }
}
```

**Template:**

```html
<!-- material-pagination.component.html -->
<div class="material-pagination-container">
  <table mat-table [dataSource]="dataSource" class="mat-elevation-z8">
    
    <!-- ID Column -->
    <ng-container matColumnDef="id">
      <th mat-header-cell *matHeaderCellDef>ID</th>
      <td mat-cell *matCellDef="let user">{{ user.id }}</td>
    </ng-container>
    
    <!-- Name Column -->
    <ng-container matColumnDef="name">
      <th mat-header-cell *matHeaderCellDef>Name</th>
      <td mat-cell *matCellDef="let user">{{ user.name }}</td>
    </ng-container>
    
    <!-- Email Column -->
    <ng-container matColumnDef="email">
      <th mat-header-cell *matHeaderCellDef>Email</th>
      <td mat-cell *matCellDef="let user">{{ user.email }}</td>
    </ng-container>
    
    <!-- Role Column -->
    <ng-container matColumnDef="role">
      <th mat-header-cell *matHeaderCellDef>Role</th>
      <td mat-cell *matCellDef="let user">{{ user.role }}</td>
    </ng-container>
    
    <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
    <tr mat-row *matRowDef="let row; columns: displayedColumns;"></tr>
  </table>
  
  <mat-paginator 
    [length]="totalItems"
    [pageSize]="pageSize"
    [pageSizeOptions]="pageSizeOptions"
    (page)="onPageChange($event)"
    showFirstLastButtons>
  </mat-paginator>
</div>
```

### 7.3 Material Paginator with Server-Side Data

```typescript
@Component({
  selector: 'app-server-material-pagination',
  templateUrl: './server-material-pagination.component.html'
})
export class ServerMaterialPaginationComponent implements OnInit {
  displayedColumns: string[] = ['id', 'name', 'email', 'role'];
  dataSource = new MatTableDataSource<User>([]);
  
  @ViewChild(MatPaginator) paginator!: MatPaginator;
  
  totalItems = 0;
  pageSize = 10;
  pageSizeOptions = [5, 10, 25, 50];
  loading = false;
  
  constructor(private userApi: UserApiService) {}
  
  ngOnInit() {
    this.loadPage(0, this.pageSize);
  }
  
  loadPage(pageIndex: number, pageSize: number) {
    this.loading = true;
    
    // API expects 1-based page numbers
    const page = pageIndex + 1;
    
    this.userApi.getUsers(page, pageSize).subscribe({
      next: (response) => {
        this.dataSource.data = response.data;
        this.totalItems = response.totalItems;
        this.loading = false;
      },
      error: () => {
        this.loading = false;
      }
    });
  }
  
  onPageChange(event: PageEvent) {
    this.loadPage(event.pageIndex, event.pageSize);
  }
}
```

---

## 8. NgRx State Management Pagination

### 8.1 Actions

```typescript
// pagination.actions.ts
import { createAction, props } from '@ngrx/store';

export const loadPage = createAction(
  '[Pagination] Load Page',
  props<{ page: number; pageSize: number }>()
);

export const loadPageSuccess = createAction(
  '[Pagination] Load Page Success',
  props<{ data: any[]; totalItems: number; page: number }>()
);

export const loadPageFailure = createAction(
  '[Pagination] Load Page Failure',
  props<{ error: any }>()
);

export const setPageSize = createAction(
  '[Pagination] Set Page Size',
  props<{ pageSize: number }>()
);

export const nextPage = createAction('[Pagination] Next Page');
export const previousPage = createAction('[Pagination] Previous Page');
export const goToPage = createAction(
  '[Pagination] Go To Page',
  props<{ page: number }>()
);
```

### 8.2 Reducer

```typescript
// pagination.reducer.ts
import { createReducer, on } from '@ngrx/store';
import * as PaginationActions from './pagination.actions';

export interface PaginationState {
  data: any[];
  currentPage: number;
  pageSize: number;
  totalItems: number;
  totalPages: number;
  loading: boolean;
  error: any;
}

export const initialState: PaginationState = {
  data: [],
  currentPage: 1,
  pageSize: 10,
  totalItems: 0,
  totalPages: 0,
  loading: false,
  error: null
};

export const paginationReducer = createReducer(
  initialState,
  
  on(PaginationActions.loadPage, (state) => ({
    ...state,
    loading: true,
    error: null
  })),
  
  on(PaginationActions.loadPageSuccess, (state, { data, totalItems, page }) => ({
    ...state,
    data,
    totalItems,
    currentPage: page,
    totalPages: Math.ceil(totalItems / state.pageSize),
    loading: false
  })),
  
  on(PaginationActions.loadPageFailure, (state, { error }) => ({
    ...state,
    loading: false,
    error
  })),
  
  on(PaginationActions.setPageSize, (state, { pageSize }) => ({
    ...state,
    pageSize,
    currentPage: 1,
    totalPages: Math.ceil(state.totalItems / pageSize)
  })),
  
  on(PaginationActions.nextPage, (state) => ({
    ...state,
    currentPage: Math.min(state.currentPage + 1, state.totalPages)
  })),
  
  on(PaginationActions.previousPage, (state) => ({
    ...state,
    currentPage: Math.max(state.currentPage - 1, 1)
  })),
  
  on(PaginationActions.goToPage, (state, { page }) => ({
    ...state,
    currentPage: Math.max(1, Math.min(page, state.totalPages))
  }))
);
```

### 8.3 Effects

```typescript
// pagination.effects.ts
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { Store } from '@ngrx/store';
import { of } from 'rxjs';
import { map, catchError, switchMap, withLatestFrom } from 'rxjs/operators';
import * as PaginationActions from './pagination.actions';
import { selectCurrentPage, selectPageSize } from './pagination.selectors';

@Injectable()
export class PaginationEffects {
  
  loadPage$ = createEffect(() =>
    this.actions$.pipe(
      ofType(PaginationActions.loadPage),
      switchMap(({ page, pageSize }) =>
        this.dataService.getItems(page, pageSize).pipe(
          map(response =>
            PaginationActions.loadPageSuccess({
              data: response.data,
              totalItems: response.totalItems,
              page
            })
          ),
          catchError(error =>
            of(PaginationActions.loadPageFailure({ error }))
          )
        )
      )
    )
  );
  
  pageNavigation$ = createEffect(() =>
    this.actions$.pipe(
      ofType(
        PaginationActions.nextPage,
        PaginationActions.previousPage,
        PaginationActions.goToPage,
        PaginationActions.setPageSize
      ),
      withLatestFrom(
        this.store.select(selectCurrentPage),
        this.store.select(selectPageSize)
      ),
      map(([action, page, pageSize]) =>
        PaginationActions.loadPage({ page, pageSize })
      )
    )
  );
  
  constructor(
    private actions$: Actions,
    private store: Store,
    private dataService: DataService
  ) {}
}
```

### 8.4 Selectors

```typescript
// pagination.selectors.ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { PaginationState } from './pagination.reducer';

export const selectPaginationState = createFeatureSelector<PaginationState>('pagination');

export const selectData = createSelector(
  selectPaginationState,
  (state) => state.data
);

export const selectCurrentPage = createSelector(
  selectPaginationState,
  (state) => state.currentPage
);

export const selectPageSize = createSelector(
  selectPaginationState,
  (state) => state.pageSize
);

export const selectTotalItems = createSelector(
  selectPaginationState,
  (state) => state.totalItems
);

export const selectTotalPages = createSelector(
  selectPaginationState,
  (state) => state.totalPages
);

export const selectLoading = createSelector(
  selectPaginationState,
  (state) => state.loading
);

export const selectError = createSelector(
  selectPaginationState,
  (state) => state.error
);

export const selectPaginationInfo = createSelector(
  selectCurrentPage,
  selectPageSize,
  selectTotalItems,
  selectTotalPages,
  (currentPage, pageSize, totalItems, totalPages) => ({
    currentPage,
    pageSize,
    totalItems,
    totalPages,
    startIndex: (currentPage - 1) * pageSize + 1,
    endIndex: Math.min(currentPage * pageSize, totalItems)
  })
);
```

### 8.5 Component with NgRx

```typescript
@Component({
  selector: 'app-ngrx-pagination',
  templateUrl: './ngrx-pagination.component.html'
})
export class NgrxPaginationComponent implements OnInit {
  data$ = this.store.select(selectData);
  paginationInfo$ = this.store.select(selectPaginationInfo);
  loading$ = this.store.select(selectLoading);
  error$ = this.store.select(selectError);
  
  constructor(private store: Store) {}
  
  ngOnInit() {
    this.store.dispatch(PaginationActions.loadPage({ page: 1, pageSize: 10 }));
  }
  
  onPageChange(page: number) {
    this.store.dispatch(PaginationActions.goToPage({ page }));
  }
  
  onPageSizeChange(pageSize: number) {
    this.store.dispatch(PaginationActions.setPageSize({ pageSize }));
  }
  
  nextPage() {
    this.store.dispatch(PaginationActions.nextPage());
  }
  
  previousPage() {
    this.store.dispatch(PaginationActions.previousPage());
  }
}
```

**Template:**

```html
<!-- ngrx-pagination.component.html -->
<div class="ngrx-pagination-container">
  <div *ngIf="loading$ | async" class="loading">Loading...</div>
  
  <div *ngIf="error$ | async as error" class="error">
    {{ error }}
  </div>
  
  <div class="data-list">
    <div *ngFor="let item of data$ | async" class="item">
      {{ item.name }}
    </div>
  </div>
  
  <div *ngIf="paginationInfo$ | async as info" class="pagination-controls">
    <div class="info">
      Showing {{ info.startIndex }} to {{ info.endIndex }} of {{ info.totalItems }}
    </div>
    
    <button 
      (click)="previousPage()" 
      [disabled]="info.currentPage === 1">
      Previous
    </button>
    
    <span>Page {{ info.currentPage }} of {{ info.totalPages }}</span>
    
    <button 
      (click)="nextPage()" 
      [disabled]="info.currentPage === info.totalPages">
      Next
    </button>
  </div>
</div>
```

---

## 9. Reusable Pagination Component

### 9.1 Reusable Pagination Component

```typescript
import { Component, EventEmitter, Input, Output } from '@angular/core';

@Component({
  selector: 'app-pagination',
  templateUrl: './pagination.component.html',
  styleUrls: ['./pagination.component.scss']
})
export class PaginationComponent {
  @Input() currentPage = 1;
  @Input() totalPages = 1;
  @Input() totalItems = 0;
  @Input() pageSize = 10;
  @Input() maxVisiblePages = 5;
  @Input() showFirstLast = true;
  @Input() showPageSize = true;
  @Input() pageSizeOptions = [10, 25, 50, 100];
  
  @Output() pageChange = new EventEmitter<number>();
  @Output() pageSizeChange = new EventEmitter<number>();
  
  get visiblePages(): number[] {
    const pages: number[] = [];
    let startPage = Math.max(1, this.currentPage - Math.floor(this.maxVisiblePages / 2));
    let endPage = Math.min(this.totalPages, startPage + this.maxVisiblePages - 1);
    
    if (endPage - startPage < this.maxVisiblePages - 1) {
      startPage = Math.max(1, endPage - this.maxVisiblePages + 1);
    }
    
    for (let i = startPage; i <= endPage; i++) {
      pages.push(i);
    }
    
    return pages;
  }
  
  get startIndex(): number {
    return (this.currentPage - 1) * this.pageSize + 1;
  }
  
  get endIndex(): number {
    return Math.min(this.currentPage * this.pageSize, this.totalItems);
  }
  
  goToPage(page: number) {
    if (page >= 1 && page <= this.totalPages && page !== this.currentPage) {
      this.pageChange.emit(page);
    }
  }
  
  onPageSizeChange(size: number) {
    this.pageSizeChange.emit(size);
  }
  
  first() {
    this.goToPage(1);
  }
  
  last() {
    this.goToPage(this.totalPages);
  }
  
  next() {
    this.goToPage(this.currentPage + 1);
  }
  
  previous() {
    this.goToPage(this.currentPage - 1);
  }
}
```

**Template:**

```html
<!-- pagination.component.html -->
<div class="pagination-wrapper">
  <div class="pagination-info">
    Showing {{ startIndex }} to {{ endIndex }} of {{ totalItems }} entries
  </div>
  
  <div class="pagination-controls">
    <!-- First button -->
    <button 
      *ngIf="showFirstLast"
      (click)="first()"
      [disabled]="currentPage === 1"
      class="pagination-btn">
      First
    </button>
    
    <!-- Previous button -->
    <button 
      (click)="previous()"
      [disabled]="currentPage === 1"
      class="pagination-btn">
      ‹ Prev
    </button>
    
    <!-- Page numbers -->
    <button 
      *ngFor="let page of visiblePages"
      (click)="goToPage(page)"
      [class.active]="currentPage === page"
      class="pagination-btn page-number">
      {{ page }}
    </button>
    
    <!-- Next button -->
    <button 
      (click)="next()"
      [disabled]="currentPage === totalPages"
      class="pagination-btn">
      Next ›
    </button>
    
    <!-- Last button -->
    <button 
      *ngIf="showFirstLast"
      (click)="last()"
      [disabled]="currentPage === totalPages"
      class="pagination-btn">
      Last
    </button>
  </div>
  
  <!-- Page size selector -->
  <div *ngIf="showPageSize" class="page-size-selector">
    <label>Show:</label>
    <select 
      [ngModel]="pageSize"
      (ngModelChange)="onPageSizeChange($event)">
      <option *ngFor="let option of pageSizeOptions" [value]="option">
        {{ option }}
      </option>
    </select>
    <span>per page</span>
  </div>
</div>
```

### 9.2 Using the Reusable Component

```typescript
@Component({
  selector: 'app-user-list',
  template: `
    <div class="user-list">
      <div *ngFor="let user of paginatedUsers">
        {{ user.name }}
      </div>
      
      <app-pagination
        [currentPage]="currentPage"
        [totalPages]="totalPages"
        [totalItems]="totalItems"
        [pageSize]="pageSize"
        (pageChange)="onPageChange($event)"
        (pageSizeChange)="onPageSizeChange($event)">
      </app-pagination>
    </div>
  `
})
export class UserListComponent {
  users: User[] = [];
  paginatedUsers: User[] = [];
  currentPage = 1;
  pageSize = 10;
  totalItems = 0;
  totalPages = 0;
  
  ngOnInit() {
    this.loadUsers();
  }
  
  loadUsers() {
    this.userService.getUsers().subscribe(users => {
      this.users = users;
      this.totalItems = users.length;
      this.totalPages = Math.ceil(this.totalItems / this.pageSize);
      this.updatePaginatedData();
    });
  }
  
  updatePaginatedData() {
    const start = (this.currentPage - 1) * this.pageSize;
    const end = start + this.pageSize;
    this.paginatedUsers = this.users.slice(start, end);
  }
  
  onPageChange(page: number) {
    this.currentPage = page;
    this.updatePaginatedData();
  }
  
  onPageSizeChange(size: number) {
    this.pageSize = size;
    this.currentPage = 1;
    this.totalPages = Math.ceil(this.totalItems / this.pageSize);
    this.updatePaginatedData();
  }
}
```

---

## 10. Advanced Techniques

### 10.1 Pagination with Filtering and Sorting

```typescript
@Component({
  selector: 'app-advanced-pagination',
  templateUrl: './advanced-pagination.component.html'
})
export class AdvancedPaginationComponent implements OnInit {
  allData: any[] = [];
  filteredData: any[] = [];
  paginatedData: any[] = [];
  
  currentPage = 1;
  pageSize = 10;
  totalPages = 0;
  
  searchTerm = '';
  sortColumn = 'name';
  sortDirection: 'asc' | 'desc' = 'asc';
  
  ngOnInit() {
    this.loadData();
  }
  
  loadData() {
    this.dataService.getAllData().subscribe(data => {
      this.allData = data;
      this.applyFiltersAndSort();
    });
  }
  
  applyFiltersAndSort() {
    // Filter
    if (this.searchTerm) {
      this.filteredData = this.allData.filter(item =>
        item.name.toLowerCase().includes(this.searchTerm.toLowerCase()) ||
        item.email.toLowerCase().includes(this.searchTerm.toLowerCase())
      );
    } else {
      this.filteredData = [...this.allData];
    }
    
    // Sort
    this.filteredData.sort((a, b) => {
      const aVal = a[this.sortColumn];
      const bVal = b[this.sortColumn];
      
      let comparison = 0;
      if (aVal > bVal) comparison = 1;
      if (aVal < bVal) comparison = -1;
      
      return this.sortDirection === 'asc' ? comparison : -comparison;
    });
    
    // Update pagination
    this.totalPages = Math.ceil(this.filteredData.length / this.pageSize);
    this.currentPage = 1;
    this.updatePaginatedData();
  }
  
  updatePaginatedData() {
    const start = (this.currentPage - 1) * this.pageSize;
    const end = start + this.pageSize;
    this.paginatedData = this.filteredData.slice(start, end);
  }
  
  onSearch(term: string) {
    this.searchTerm = term;
    this.applyFiltersAndSort();
  }
  
  onSort(column: string) {
    if (this.sortColumn === column) {
      this.sortDirection = this.sortDirection === 'asc' ? 'desc' : 'asc';
    } else {
      this.sortColumn = column;
      this.sortDirection = 'asc';
    }
    this.applyFiltersAndSort();
  }
}
```

### 10.2 Pagination with URL Parameters

```typescript
import { ActivatedRoute, Router } from '@angular/router';

@Component({
  selector: 'app-url-pagination',
  templateUrl: './url-pagination.component.html'
})
export class UrlPaginationComponent implements OnInit {
  items: any[] = [];
  currentPage = 1;
  pageSize = 10;
  
  constructor(
    private route: ActivatedRoute,
    private router: Router,
    private dataService: DataService
  ) {}
  
  ngOnInit() {
    // Read pagination from URL
    this.route.queryParams.subscribe(params => {
      this.currentPage = +params['page'] || 1;
      this.pageSize = +params['pageSize'] || 10;
      this.loadData();
    });
  }
  
  loadData() {
    this.dataService.getItems(this.currentPage, this.pageSize)
      .subscribe(response => {
        this.items = response.data;
      });
  }
  
  goToPage(page: number) {
    // Update URL
    this.router.navigate([], {
      relativeTo: this.route,
      queryParams: {
        page,
        pageSize: this.pageSize
      },
      queryParamsHandling: 'merge'
    });
  }
  
  changePageSize(size: number) {
    this.router.navigate([], {
      relativeTo: this.route,
      queryParams: {
        page: 1,
        pageSize: size
      },
      queryParamsHandling: 'merge'
    });
  }
}
```

### 10.3 Pagination with Caching

```typescript
@Injectable({
  providedIn: 'root'
})
export class CachedPaginationService {
  private cache = new Map<string, any>();
  private cacheExpiry = 5 * 60 * 1000; // 5 minutes
  
  constructor(private http: HttpClient) {}
  
  getItems(page: number, pageSize: number): Observable<any> {
    const cacheKey = `${page}-${pageSize}`;
    const cached = this.cache.get(cacheKey);
    
    // Check if cache exists and is not expired
    if (cached && Date.now() - cached.timestamp < this.cacheExpiry) {
      console.log('Using cached data');
      return of(cached.data);
    }
    
    // Fetch fresh data
    return this.http.get(`/api/items?page=${page}&pageSize=${pageSize}`)
      .pipe(
        tap(data => {
          // Store in cache
          this.cache.set(cacheKey, {
            data,
            timestamp: Date.now()
          });
        })
      );
  }
  
  clearCache() {
    this.cache.clear();
  }
  
  invalidatePage(page: number, pageSize: number) {
    const cacheKey = `${page}-${pageSize}`;
    this.cache.delete(cacheKey);
  }
}
```

---

## Summary

### Choosing the Right Pagination Strategy

| Strategy | Best For | Pros | Cons |
|----------|----------|------|------|
| **Client-Side** | Small datasets (<1000 items) | Simple, fast navigation | Loads all data upfront |
| **Server-Side** | Large datasets | Efficient, scalable | More complex, requires API |
| **Infinite Scroll** | Social feeds, product catalogs | Great UX, mobile-friendly | Hard to reach footer |
| **Virtual Scroll** | Very large lists (10k+ items) | Excellent performance | Limited UI flexibility |
| **Load More** | Image galleries, news feeds | Simple, clear | Can get cluttered |
| **Cursor-Based** | Real-time data, distributed systems | Consistent, handles updates well | Can't jump to specific page |
| **Material Paginator** | Admin panels, data tables | Professional, accessible | Requires Material dependency |

### Performance Tips

1. **Use trackBy** in *ngFor for better performance
2. **Implement virtual scrolling** for large lists
3. **Cache API responses** when appropriate
4. **Debounce search inputs** to reduce API calls
5. **Use OnPush change detection** for pagination components
6. **Lazy load images** in paginated lists
7. **Implement skeleton loaders** for better UX

Choose the pagination strategy that best fits your use case, data size, and user experience requirements!