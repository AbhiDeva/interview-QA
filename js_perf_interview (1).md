# JavaScript Performance Interview Questions & Answers

## Question 1: Debouncing vs Throttling

**Question:** Implement both debouncing and throttling functions. Explain when to use each.

**Answer:**

Debouncing delays execution until after a wait period of inactivity:

```javascript
function debounce(fn, delay) {
  let timer;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

Throttling ensures function executes at most once per time period:

```javascript
function throttle(fn, limit) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}
```

**Usage:**
- **Debounce**: Search input, form validation (wait for user to stop typing)
- **Throttle**: Scroll events, resize events (execute at regular intervals)

```javascript
const searchHandler = debounce((query) => console.log(query), 300);
const scrollHandler = throttle(() => console.log('scrolled'), 100);
```

---

## Question 2: Memory Leak Prevention

**Question:** Identify and fix the memory leak in this code.

**Answer:**

❌ **BAD - Memory leak from event listener:**

```javascript
function badExample() {
  const btn = document.getElementById('btn');
  const data = new Array(1000000).fill('leak');
  btn.addEventListener('click', () => {
    console.log(data.length);
  });
  // Event listener keeps reference to 'data' forever
}
```

✅ **GOOD - Proper cleanup:**

```javascript
function goodExample() {
  const btn = document.getElementById('btn');
  const data = new Array(1000000).fill('no leak');
  
  const handler = () => console.log(data.length);
  btn.addEventListener('click', handler);
  
  // Return cleanup function
  return () => btn.removeEventListener('click', handler);
}
```

**Key Points:**
- Always remove event listeners when they're no longer needed
- Return cleanup functions from setup code
- Use WeakMap/WeakSet for object references that shouldn't prevent garbage collection

---

## Question 3: Efficient Array Operations

**Question:** Optimize this code that filters and maps an array multiple times.

**Answer:**

❌ **BAD - Multiple iterations:**

```javascript
function inefficientArrayOps(arr) {
  return arr
    .filter(x => x > 0)
    .map(x => x * 2)
    .filter(x => x < 100);
  // Three separate passes through the array
}
```

✅ **GOOD - Single iteration:**

```javascript
function efficientArrayOps(arr) {
  return arr.reduce((acc, x) => {
    if (x > 0) {
      const doubled = x * 2;
      if (doubled < 100) acc.push(doubled);
    }
    return acc;
  }, []);
  // Single pass through the array
}
```

**Performance:** ~40% improvement on large arrays by reducing iterations from 3 to 1.

---

## Question 4: DOM Manipulation Optimization

**Question:** How can you minimize reflows when adding multiple elements to the DOM?

**Answer:**

❌ **BAD - Multiple reflows:**

```javascript
function slowDOMUpdate(items) {
  const list = document.getElementById('list');
  items.forEach(item => {
    const li = document.createElement('li');
    li.textContent = item;
    list.appendChild(li); // Reflow on each append!
  });
}
```

✅ **GOOD - Single reflow:**

```javascript
function fastDOMUpdate(items) {
  const list = document.getElementById('list');
  const fragment = document.createDocumentFragment();
  
  items.forEach(item => {
    const li = document.createElement('li');
    li.textContent = item;
    fragment.appendChild(li);
  });
  
  list.appendChild(fragment); // Single reflow
}
```

**Key Concepts:**
- Reflows are expensive (browser recalculates layout)
- DocumentFragment is an in-memory container
- Batch DOM updates whenever possible

---

## Question 5: Memoization for Expensive Calculations

**Question:** Implement a memoization function to cache expensive computations.

**Answer:**

```javascript
function memoize(fn) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

**Usage with Fibonacci:**

```javascript
const fibonacci = memoize((n) => {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(40)); // First call: ~1 second
console.log(fibonacci(40)); // Cached: instant!
```

**Benefits:**
- Trades memory for speed
- Perfect for recursive or repetitive calculations
- Use `Map` for better performance than objects

---

## Question 6: Lazy Loading Implementation

**Question:** Implement lazy loading for images using Intersection Observer.

**Answer:**

```javascript
function lazyLoadImages() {
  const images = document.querySelectorAll('img[data-src]');
  
  const imageObserver = new IntersectionObserver((entries, observer) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const img = entry.target;
        img.src = img.dataset.src;
        img.removeAttribute('data-src');
        observer.unobserve(img); // Stop observing once loaded
      }
    });
  }, {
    rootMargin: '50px' // Start loading 50px before visible
  });
  
  images.forEach(img => imageObserver.observe(img));
}
```

**HTML:**

```html
<img data-src="actual-image.jpg" alt="Description">
```

**Benefits:**
- Reduces initial page load time
- Saves bandwidth for users
- Native browser API (no libraries needed)

---

## Question 7: Request Animation Frame

**Question:** Why is `requestAnimationFrame` better than `setInterval` for animations?

**Answer:**

❌ **BAD - Janky animation:**

```javascript
function badAnimation() {
  let pos = 0;
  const element = document.getElementById('box');
  
  setInterval(() => {
    pos += 2;
    element.style.left = pos + 'px';
  }, 16); // Attempting 60fps but not synced with display
}
```

✅ **GOOD - Smooth animation:**

```javascript
function smoothAnimation() {
  let pos = 0;
  const element = document.getElementById('box');
  
  function animate() {
    pos += 2;
    element.style.left = pos + 'px';
    
    if (pos < 500) {
      requestAnimationFrame(animate);
    }
  }
  
  requestAnimationFrame(animate);
}
```

**Why RAF is better:**
- Syncs with display refresh rate (typically 60fps)
- Pauses when tab is not visible (saves CPU/battery)
- Browser optimizes painting with other layout changes
- More consistent frame timing

---

## Question 8: Web Worker for Heavy Computation

**Question:** How do you offload heavy computation to avoid blocking the main thread?

**Answer:**

**Main thread (main.js):**

```javascript
function useWebWorker(data) {
  const worker = new Worker('worker.js');
  
  worker.postMessage(data);
  
  worker.onmessage = (e) => {
    console.log('Result:', e.data);
    worker.terminate(); // Cleanup when done
  };
  
  worker.onerror = (error) => {
    console.error('Worker error:', error);
  };
}

// Usage
useWebWorker([1, 2, 3, 4, 5]);
```

**Worker thread (worker.js):**

```javascript
self.onmessage = (e) => {
  const result = heavyComputation(e.data);
  self.postMessage(result);
};

function heavyComputation(data) {
  // Expensive operation that won't block UI
  return data.map(x => x * x).reduce((a, b) => a + b, 0);
}
```

**Use Cases:**
- Image/video processing
- Large data parsing
- Complex calculations
- Encryption/decryption

---

## Question 9: Object Property Access Optimization

**Question:** How can you optimize repeated nested property access in loops?

**Answer:**

❌ **BAD - Repeated property access:**

```javascript
function slowPropertyAccess(obj, iterations) {
  let sum = 0;
  for (let i = 0; i < iterations; i++) {
    sum += obj.deeply.nested.property.value;
    // Traverses object chain every iteration
  }
  return sum;
}
```

✅ **GOOD - Cache property reference:**

```javascript
function fastPropertyAccess(obj, iterations) {
  const value = obj.deeply.nested.property.value;
  let sum = 0;
  for (let i = 0; i < iterations; i++) {
    sum += value;
    // Uses cached reference
  }
  return sum;
}
```

**Performance Tips:**
- Cache frequently accessed properties
- Especially important in loops
- Can provide 2-3x speedup in tight loops
- Also applies to array.length in loops

---

## Question 10: Virtual Scrolling for Large Lists

**Question:** Implement virtual scrolling to efficiently render large lists.

**Answer:**

```javascript
class VirtualList {
  constructor(container, items, itemHeight) {
    this.container = container;
    this.items = items;
    this.itemHeight = itemHeight;
    this.visibleCount = Math.ceil(container.clientHeight / itemHeight);
    this.init();
  }
  
  init() {
    // Set total height for scrollbar
    this.container.style.height = this.items.length * this.itemHeight + 'px';
    this.container.style.position = 'relative';
    this.container.style.overflow = 'auto';
    
    this.container.addEventListener('scroll', () => {
      this.render();
    });
    
    this.render();
  }
  
  render() {
    const scrollTop = this.container.scrollTop;
    const startIndex = Math.floor(scrollTop / this.itemHeight);
    const endIndex = startIndex + this.visibleCount + 1;
    
    // Clear and render only visible items
    this.container.innerHTML = '';
    
    for (let i = startIndex; i < endIndex && i < this.items.length; i++) {
      const item = document.createElement('div');
      item.style.position = 'absolute';
      item.style.top = i * this.itemHeight + 'px';
      item.style.height = this.itemHeight + 'px';
      item.textContent = this.items[i];
      this.container.appendChild(item);
    }
  }
}
```

**Usage:**

```javascript
const container = document.getElementById('list');
const items = Array.from({length: 10000}, (_, i) => `Item ${i + 1}`);
new VirtualList(container, items, 50);
```

**Benefits:**
- Renders only visible items (20-30 instead of 10,000)
- Constant memory usage regardless of list size
- Smooth scrolling even with massive datasets
- Critical for performance with 1000+ items

---

## Summary

These performance optimization techniques are essential for building fast, responsive web applications:

1. **Function optimization** (debounce, throttle, memoize)
2. **Memory management** (cleanup, avoid leaks)
3. **DOM efficiency** (batch updates, virtual rendering)
4. **Async processing** (Web Workers, RAF)
5. **Algorithm optimization** (reduce iterations, cache access)

Understanding when and how to apply each technique is key to passing performance-focused interviews.