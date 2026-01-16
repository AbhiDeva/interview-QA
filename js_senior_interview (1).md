# JavaScript Interview Questions & Answers - Senior Level (15+ Years Experience)

## Core JavaScript & Language Internals

### 1. Event Loop & Asynchronous Programming

**Q: Explain the JavaScript event loop in detail, including the call stack, task queue, and microtask queue.**

**A:** The JavaScript event loop is a mechanism that handles asynchronous operations in a single-threaded environment. It consists of:

- **Call Stack**: Executes synchronous code in LIFO order
- **Task Queue (Macrotask Queue)**: Contains callbacks from setTimeout, setInterval, I/O operations
- **Microtask Queue**: Contains Promise callbacks, queueMicrotask, MutationObserver callbacks

**Execution Order:**
1. Execute all synchronous code on the call stack
2. Process ALL microtasks until the queue is empty
3. Execute ONE macrotask
4. Process ALL microtasks again
5. Render (if needed)
6. Repeat from step 3

```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

console.log('4');
// Output: 1, 4, 3, 2
// Microtasks (Promise) execute before macrotasks (setTimeout)
```

**Q: What are the differences between async/await, Promises, and callbacks?**

**A:**

**Callbacks:**
- Oldest pattern, leads to "callback hell"
- No built-in error handling
- Difficult to compose and manage

**Promises:**
- Better error handling with `.catch()`
- Chainable with `.then()`
- Can use `Promise.all()`, `Promise.race()`, etc.
- Still requires chaining

**Async/Await:**
- Syntactic sugar over Promises
- Makes async code look synchronous
- Better error handling with try/catch
- Easier to read and debug
- Can use with loops and conditional logic

```javascript
// Callback
fetchData((err, data) => {
  if (err) return handleError(err);
  processData(data, (err, result) => {
    if (err) return handleError(err);
    // More nesting...
  });
});

// Promise
fetchData()
  .then(processData)
  .catch(handleError);

// Async/Await
try {
  const data = await fetchData();
  const result = await processData(data);
} catch (err) {
  handleError(err);
}
```

**Q: How would you implement your own Promise from scratch?**

**A:**
```javascript
class MyPromise {
  constructor(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.callbacks = [];

    const resolve = (value) => {
      if (this.state !== 'pending') return;
      this.state = 'fulfilled';
      this.value = value;
      this.callbacks.forEach(cb => cb.onFulfilled(value));
    };

    const reject = (reason) => {
      if (this.state !== 'pending') return;
      this.state = 'rejected';
      this.value = reason;
      this.callbacks.forEach(cb => cb.onRejected(reason));
    };

    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      const handle = () => {
        if (this.state === 'fulfilled') {
          try {
            const result = onFulfilled ? onFulfilled(this.value) : this.value;
            resolve(result);
          } catch (err) {
            reject(err);
          }
        } else if (this.state === 'rejected') {
          if (onRejected) {
            try {
              const result = onRejected(this.value);
              resolve(result);
            } catch (err) {
              reject(err);
            }
          } else {
            reject(this.value);
          }
        }
      };

      if (this.state === 'pending') {
        this.callbacks.push({ onFulfilled, onRejected });
      } else {
        setTimeout(handle, 0);
      }
    });
  }

  catch(onRejected) {
    return this.then(null, onRejected);
  }
}
```

### 2. Memory Management & Performance

**Q: Describe how garbage collection works in V8.**

**A:** V8 uses a generational garbage collection strategy with two main areas:

**Young Generation (New Space):**
- Short-lived objects (1-8MB)
- Uses Scavenger algorithm (Cheney's copying collection)
- Very fast, runs frequently
- Objects that survive are promoted to Old Generation

**Old Generation (Old Space):**
- Long-lived objects
- Uses Mark-Sweep and Mark-Compact algorithms
- Slower but less frequent
- Major GC pauses can affect performance

**Phases:**
1. **Marking**: Identifies live objects starting from roots
2. **Sweeping**: Removes unmarked objects
3. **Compacting**: Defragments memory to prevent fragmentation

V8 also uses incremental marking and concurrent marking to reduce pause times.

**Q: What are memory leaks? Provide examples.**

**A:** Memory leaks occur when memory that's no longer needed isn't released.

**Common Causes:**

```javascript
// 1. Global variables
function leak() {
  globalVar = new Array(1000000); // Accidental global
}

// 2. Forgotten timers
const timer = setInterval(() => {
  // Large closure captured
  const data = fetchLargeData();
}, 1000);
// Never cleared with clearInterval(timer)

// 3. Event listeners not removed
element.addEventListener('click', handler);
// element removed from DOM but listener not removed

// 4. Closures holding references
function outer() {
  const largeData = new Array(1000000);
  return function inner() {
    // largeData is held in closure even if not used
    console.log('hello');
  };
}

// 5. Detached DOM nodes
let detached = document.getElementById('element');
document.body.removeChild(detached);
// detached still holds reference to DOM node
```

**Detection:**
- Chrome DevTools Heap Snapshot
- Memory Timeline profiling
- Performance monitor
- Look for increasing memory over time

### 3. Closures & Scope

**Q: Explain closures with practical use cases.**

**A:** A closure is a function that has access to variables from its outer (enclosing) lexical scope, even after the outer function has returned.

**Practical Use Cases:**

```javascript
// 1. Data Privacy (Private Variables)
function createCounter() {
  let count = 0; // private
  return {
    increment() { return ++count; },
    decrement() { return --count; },
    getCount() { return count; }
  };
}

// 2. Function Factories
function multiplier(factor) {
  return function(number) {
    return number * factor;
  };
}
const double = multiplier(2);
const triple = multiplier(3);

// 3. Memoization
function memoize(fn) {
  const cache = {};
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache[key]) return cache[key];
    cache[key] = fn(...args);
    return cache[key];
  };
}

// 4. Event Handlers with Context
function setupHandlers(elements) {
  elements.forEach((element, index) => {
    element.addEventListener('click', () => {
      console.log(`Clicked element ${index}`);
      // index is captured in closure
    });
  });
}
```

**Performance Implications:**
- Closures consume memory (keep variables in scope)
- Can prevent garbage collection if not managed
- Minimal performance impact in modern engines
- Consider WeakMap for large data sets

**Q: What is the Temporal Dead Zone (TDZ)?**

**A:** The TDZ is the period between entering scope and the variable declaration being executed, during which the variable cannot be accessed.

```javascript
console.log(a); // ReferenceError: Cannot access 'a' before initialization
let a = 5;

// TDZ exists for let and const, but not var
console.log(b); // undefined (hoisted)
var b = 5;

// TDZ in function parameters
function test(a = b, b = 2) {
  // ReferenceError: b is in TDZ when a is initialized
}

// TDZ in blocks
{
  // TDZ starts
  console.log(x); // ReferenceError
  let x = 1; // TDZ ends
}
```

### 4. Prototypal Inheritance

**Q: Explain prototypal inheritance vs classical inheritance.**

**A:**

**Prototypal Inheritance:**
- Objects inherit directly from other objects
- More flexible, objects can be extended at runtime
- Uses prototype chain
- JavaScript's native model

```javascript
const animal = {
  eat() { console.log('eating'); }
};

const dog = Object.create(animal);
dog.bark = function() { console.log('barking'); };
```

**Classical Inheritance (Classes):**
- Classes inherit from classes
- More structured, familiar to OOP developers
- Syntactic sugar over prototypal inheritance
- Established at compile time

```javascript
class Animal {
  eat() { console.log('eating'); }
}

class Dog extends Animal {
  bark() { console.log('barking'); }
}
```

**Trade-offs:**
- Prototypal: More flexible, dynamic, true to JavaScript
- Classical: More familiar, better tooling support, clearer hierarchy

**Q: How does the prototype chain work?**

**A:** When accessing a property on an object:
1. Check if property exists on the object itself
2. If not, check the object's `[[Prototype]]` (accessible via `__proto__`)
3. Continue up the chain until property is found or reach `null`

```javascript
const obj = { a: 1 };
Object.getPrototypeOf(obj) === Object.prototype; // true
Object.getPrototypeOf(Object.prototype) === null; // true

// Lookup chain
obj.toString(); // Found in Object.prototype
obj.a; // Found on obj itself
obj.nonExistent; // undefined (reached null)
```

**Q: Difference between `__proto__` and `prototype`?**

**A:**
- `prototype`: Property on constructor functions, used when creating new instances
- `__proto__`: Property on objects, points to the object's prototype

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() {
  console.log(`Hi, I'm ${this.name}`);
};

const john = new Person('John');

// Person.prototype is the prototype object
// john.__proto__ points to Person.prototype
john.__proto__ === Person.prototype; // true
```

## Advanced Patterns & Architecture

### 5. Design Patterns

**Q: Implement Module, Revealing Module, and Singleton patterns.**

**A:**

```javascript
// Module Pattern
const Module = (function() {
  let privateVar = 0;
  
  function privateMethod() {
    return privateVar++;
  }
  
  return {
    publicMethod() {
      return privateMethod();
    }
  };
})();

// Revealing Module Pattern
const RevealingModule = (function() {
  let privateVar = 0;
  
  function increment() {
    return ++privateVar;
  }
  
  function get() {
    return privateVar;
  }
  
  // Reveal only what's needed
  return {
    increment,
    getValue: get
  };
})();

// Singleton Pattern
const Singleton = (function() {
  let instance;
  
  function createInstance() {
    return {
      property: 'value',
      method() { return 'singleton method'; }
    };
  }
  
  return {
    getInstance() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

// ES6 Singleton with Class
class SingletonClass {
  constructor() {
    if (SingletonClass.instance) {
      return SingletonClass.instance;
    }
    SingletonClass.instance = this;
    this.data = [];
  }
}
```

**Use Cases:**
- **Module**: Encapsulation, organizing code, avoiding global pollution
- **Revealing Module**: Clear public API, better minification
- **Singleton**: Database connections, configuration, logging

**Q: Explain the Observer pattern.**

**A:** The Observer pattern defines a one-to-many dependency where multiple observers are notified when a subject's state changes.

```javascript
class Subject {
  constructor() {
    this.observers = [];
  }
  
  subscribe(observer) {
    this.observers.push(observer);
  }
  
  unsubscribe(observer) {
    this.observers = this.observers.filter(obs => obs !== observer);
  }
  
  notify(data) {
    this.observers.forEach(observer => observer.update(data));
  }
}

class Observer {
  constructor(name) {
    this.name = name;
  }
  
  update(data) {
    console.log(`${this.name} received:`, data);
  }
}

// Usage
const subject = new Subject();
const observer1 = new Observer('Observer 1');
const observer2 = new Observer('Observer 2');

subject.subscribe(observer1);
subject.subscribe(observer2);
subject.notify('New data available');
```

**Used in:**
- Event systems
- State management (Redux, MobX)
- Reactive programming (RxJS)
- UI frameworks (Vue, React hooks)

### 6. Functional Programming

**Q: Explain pure functions, immutability, and side effects.**

**A:**

**Pure Functions:**
- Same input always produces same output
- No side effects
- Referentially transparent

```javascript
// Pure
function add(a, b) {
  return a + b;
}

// Impure (modifies external state)
let total = 0;
function addToTotal(value) {
  total += value;
  return total;
}

// Impure (depends on external state)
function getTax(price) {
  return price * window.taxRate;
}
```

**Immutability:**
- Data cannot be changed after creation
- Create new data structures instead of modifying

```javascript
// Mutable
const arr = [1, 2, 3];
arr.push(4); // Modifies original

// Immutable
const arr2 = [1, 2, 3];
const newArr = [...arr2, 4]; // Creates new array
```

**Benefits:**
- Predictable code
- Easier testing
- Time-travel debugging
- Concurrent programming
- Memoization opportunities

**Q: Higher-order functions with advanced examples.**

**A:** Functions that take functions as arguments or return functions.

```javascript
// Function composition
const compose = (...fns) => x => 
  fns.reduceRight((acc, fn) => fn(acc), x);

const addOne = x => x + 1;
const double = x => x * 2;
const square = x => x * x;

const pipeline = compose(square, double, addOne);
pipeline(3); // (3 + 1) * 2 ^ 2 = 64

// Partial application
const partial = (fn, ...args) => 
  (...moreArgs) => fn(...args, ...moreArgs);

const multiply = (a, b, c) => a * b * c;
const double = partial(multiply, 2);
double(3, 4); // 2 * 3 * 4 = 24

// Currying
const curry = (fn) => {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return (...moreArgs) => curried(...args, ...moreArgs);
  };
};

const sum = (a, b, c) => a + b + c;
const curriedSum = curry(sum);
curriedSum(1)(2)(3); // 6
curriedSum(1, 2)(3); // 6
```

**Q: Implement a Maybe monad.**

**A:**
```javascript
class Maybe {
  constructor(value) {
    this.value = value;
  }
  
  static of(value) {
    return new Maybe(value);
  }
  
  isNothing() {
    return this.value === null || this.value === undefined;
  }
  
  map(fn) {
    return this.isNothing() ? Maybe.of(null) : Maybe.of(fn(this.value));
  }
  
  flatMap(fn) {
    return this.isNothing() ? Maybe.of(null) : fn(this.value);
  }
  
  getOrElse(defaultValue) {
    return this.isNothing() ? defaultValue : this.value;
  }
}

// Usage
const user = { name: 'John', address: { city: 'NYC' } };

Maybe.of(user)
  .map(u => u.address)
  .map(addr => addr.city)
  .map(city => city.toUpperCase())
  .getOrElse('Unknown'); // 'NYC'

// Handles null safely
Maybe.of(null)
  .map(u => u.address)
  .map(addr => addr.city)
  .getOrElse('Unknown'); // 'Unknown'
```

## Modern JavaScript & ES6+

### 7. ES6+ Features

**Q: Differences between var, let, and const.**

**A:**

```javascript
// VAR
// - Function scoped
// - Hoisted (initialized as undefined)
// - Can be redeclared
// - Creates property on global object

function testVar() {
  console.log(x); // undefined (hoisted)
  var x = 5;
  if (true) {
    var x = 10; // Same variable
  }
  console.log(x); // 10
}

// LET
// - Block scoped
// - Hoisted but in TDZ
// - Cannot be redeclared
// - No global property

function testLet() {
  // console.log(x); // ReferenceError
  let x = 5;
  if (true) {
    let x = 10; // Different variable
    console.log(x); // 10
  }
  console.log(x); // 5
}

// CONST
// - Block scoped
// - Hoisted but in TDZ
// - Cannot be reassigned
// - Must be initialized
// - Objects/arrays can be mutated

const obj = { a: 1 };
// obj = {}; // Error
obj.a = 2; // OK

const arr = [1, 2];
// arr = []; // Error
arr.push(3); // OK
```

**Q: Symbols and WeakMaps use cases.**

**A:**

**Symbols:**
- Unique, immutable primitive values
- Used as object property keys
- Not enumerable in for...in loops

```javascript
// Private-ish properties
const _private = Symbol('private');
class MyClass {
  constructor() {
    this[_private] = 'secret';
  }
  getPrivate() {
    return this[_private];
  }
}

// Well-known symbols
const obj = {
  [Symbol.iterator]() {
    let count = 0;
    return {
      next() {
        return count < 3 
          ? { value: count++, done: false }
          : { done: true };
      }
    };
  }
};

// Metaprogramming
class Collection {
  [Symbol.toStringTag] = 'Collection';
}
String(new Collection()); // '[object Collection]'
```

**WeakMaps:**
- Keys must be objects
- Weak references (don't prevent GC)
- Not enumerable
- No size property

```javascript
// Use case: Private data
const privateData = new WeakMap();

class Person {
  constructor(name, ssn) {
    this.name = name;
    privateData.set(this, { ssn });
  }
  
  getSSN() {
    return privateData.get(this).ssn;
  }
}

// Use case: DOM metadata
const elementMetadata = new WeakMap();

function attachMetadata(element, data) {
  elementMetadata.set(element, data);
}

// When element is removed from DOM, metadata is GC'd
```

**Q: Proxies and Reflect API.**

**A:**

```javascript
// Proxy for validation
const validator = {
  set(target, property, value) {
    if (property === 'age') {
      if (typeof value !== 'number' || value < 0) {
        throw new TypeError('Age must be a positive number');
      }
    }
    target[property] = value;
    return true;
  }
};

const person = new Proxy({}, validator);
person.age = 30; // OK
// person.age = -5; // TypeError

// Proxy for logging
const createLoggingProxy = (target, name) => {
  return new Proxy(target, {
    get(target, property) {
      console.log(`[${name}] Getting ${property}`);
      return Reflect.get(target, property);
    },
    set(target, property, value) {
      console.log(`[${name}] Setting ${property} = ${value}`);
      return Reflect.set(target, property, value);
    }
  });
};

// Proxy for default values
const withDefaults = (target, defaults) => {
  return new Proxy(target, {
    get(target, property) {
      return property in target 
        ? target[property] 
        : defaults[property];
    }
  });
};

const config = withDefaults({}, {
  timeout: 3000,
  retries: 3
});

// Proxy for negative array indexing
const createNegativeArray = (arr) => {
  return new Proxy(arr, {
    get(target, property) {
      const index = Number(property);
      if (index < 0) {
        return target[target.length + index];
      }
      return Reflect.get(target, property);
    }
  });
};

const arr = createNegativeArray([1, 2, 3, 4]);
arr[-1]; // 4
arr[-2]; // 3
```

**Q: Generators and iterators.**

**A:**

```javascript
// Basic generator
function* numberGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = numberGenerator();
gen.next(); // { value: 1, done: false }
gen.next(); // { value: 2, done: false }
gen.next(); // { value: 3, done: false }
gen.next(); // { value: undefined, done: true }

// Infinite generator
function* fibonacci() {
  let [a, b] = [0, 1];
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

// Lazy evaluation
function* range(start, end) {
  for (let i = start; i < end; i++) {
    yield i;
  }
}

function* map(iterable, fn) {
  for (const item of iterable) {
    yield fn(item);
  }
}

function* filter(iterable, predicate) {
  for (const item of iterable) {
    if (predicate(item)) {
      yield item;
    }
  }
}

// Composing generators
const numbers = range(1, 10);
const doubled = map(numbers, x => x * 2);
const evens = filter(doubled, x => x % 2 === 0);

// Generator delegation
function* inner() {
  yield 'inner';
}

function* outer() {
  yield 'outer';
  yield* inner();
  yield 'done';
}

// Async generators
async function* asyncGenerator() {
  for (let i = 0; i < 3; i++) {
    await new Promise(resolve => setTimeout(resolve, 1000));
    yield i;
  }
}

// Usage
for await (const value of asyncGenerator()) {
  console.log(value); // 0, 1, 2 (each after 1 second)
}
```

## Browser APIs & Web Standards

### 8. DOM & Rendering

**Q: Explain the browser rendering pipeline.**

**A:**

**Pipeline Stages:**
1. **Parse HTML** → DOM tree
2. **Parse CSS** → CSSOM tree
3. **Combine** → Render tree (only visible nodes)
4. **Layout** (Reflow) → Calculate positions and sizes
5. **Paint** → Fill in pixels (text, colors, borders)
6. **Composite** → Combine layers into final image

**Reflow (Layout):**
- Recalculates element positions/sizes
- Triggered by: DOM changes, size changes, layout properties
- Expensive operation

```javascript
// Causes reflow
element.style.width = '100px';
const height = element.offsetHeight; // Forces reflow

// Minimize reflows
const width = element.offsetWidth; // Read
const height = element.offsetHeight; // Read
element.style.width = width + 10 + 'px'; // Write
element.style.height = height + 10 + 'px'; // Write
```

**Repaint:**
- Updates pixels without layout changes
- Triggered by: color, visibility, background changes
- Less expensive than reflow

**Optimization Strategies:**
```javascript
// Batch DOM changes
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  fragment.appendChild(div);
}
document.body.appendChild(fragment); // Single reflow

// Use CSS classes
element.classList.add('active'); // Better than inline styles

// Use transform and opacity (GPU accelerated)
element.style.transform = 'translateX(100px)'; // No layout
element.style.opacity = 0.5; // No layout

// RequestAnimationFrame
requestAnimationFrame(() => {
  element.style.transform = 'translateX(100px)';
});
```

**Q: Virtual DOM benefits.**

**A:** The Virtual DOM is an in-memory representation of the actual DOM.

**Benefits:**
1. **Batched Updates**: Multiple changes batched into single DOM update
2. **Efficient Diffing**: Only changed elements are updated
3. **Reduced Reflows**: Minimize expensive DOM operations
4. **Cross-platform**: Can render to different targets
5. **Developer Experience**: Declarative programming model

**How it works:**
```javascript
// Simplified Virtual DOM implementation
function createElement(type, props, ...children) {
  return { type, props, children };
}

function diff(oldVNode, newVNode) {
  // Compare and return patches
  if (oldVNode.type !== newVNode.type) {
    return { type: 'REPLACE', newVNode };
  }
  
  const propPatches = diffProps(oldVNode.props, newVNode.props);
  const childPatches = diffChildren(oldVNode.children, newVNode.children);
  
  return { type: 'UPDATE', propPatches, childPatches };
}

function patch(domNode, patches) {
  // Apply minimal changes to real DOM
  switch (patches.type) {
    case 'REPLACE':
      domNode.replaceWith(render(patches.newVNode));
      break;
    case 'UPDATE':
      updateProps(domNode, patches.propPatches);
      patchChildren(domNode, patches.childPatches);
      break;
  }
}
```

### 9. Web APIs

**Q: Service Workers vs Web Workers.**

**A:**

**Service Workers:**
- Runs in background, separate thread
- Acts as proxy between browser and network
- Can intercept network requests
- Enables offline functionality
- Has access to Cache API
- Lifecycle: install, activate, fetch

```javascript
// Register service worker
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(reg => console.log('SW registered', reg))
    .catch(err => console.error('SW error', err));
}

// service-worker.js
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open('v1').then(cache => {
      return cache.addAll([
        '/',
        '/styles.css',
        '/script.js'
      ]);
    })
  );
});

self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => response || fetch(event.request))
  );
});
```

**Web Workers:**
- Runs JavaScript in background thread
- For CPU-intensive computations
- No DOM access
- Communicate via messages

```javascript
// Main thread
const worker = new Worker('worker.js');
worker.postMessage({ data: largeArray });
worker.onmessage = (e) => {
  console.log('Result:', e.data);
};

// worker.js
self.onmessage = (e) => {
  const result = processData(e.data);
  self.postMessage(result);
};
```

**Key Differences:**
- Service Workers: Network proxy, offline, one per origin
- Web Workers: Computation, multiple instances

### 10. Security

**Q: XSS prevention strategies.**

**A:**

**Types of XSS:**
1. **Stored XSS**: Malicious script stored in database
2. **Reflected XSS**: Script reflected from URL/input
3. **DOM-based XSS**: Client-side script manipulation

**Prevention:**

```javascript
// 1. Input Validation
function sanitizeInput(input) {
  return input
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;')
    .replace(/\//g, '&#x2F;');
}

// 2. Use textContent instead of innerHTML
element.textContent = userInput; // Safe
// element.innerHTML = userInput; // Dangerous

// 3. DOMPurify library
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(dirty);

// 4. Content Security Policy (CSP)
// HTTP Header or meta tag
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self' https://trusted.com">

// 5. Use frameworks that auto-escape
// React, Vue, Angular automatically escape by default

// 6. HttpOnly cookies
// Set-Cookie: sessionId=abc123; HttpOnly; Secure

// 7. Encode output context-appropriate
function encodeForHTML(str) {
  return String(str)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;');
}

function encodeForJS(str) {
  return String(str)
    .replace(/\\/g, '\\\\')
    .replace(/'/g, "\\'")
    .replace(/"/g, '\\"')
    .replace(/\n/g, '\\n');
}
```

**Q: CSRF and mitigation.**

**A:** CSRF tricks users into executing unwanted actions on authenticated sites.

**Prevention:**

```javascript
// 1. CSRF Tokens
// Server generates unique token per session
<form method="POST">
  <input type="hidden" name="csrf_token" value="${csrfToken}">
  <!-- form fields -->
</form>

// 2. SameSite Cookie attribute
// Set-Cookie: sessionId=abc123; SameSite=Strict

// 3. Verify Origin/Referer headers
app.post('/transfer', (req, res) => {
  const origin = req.get('origin');
  if (origin !== 'https://mybank.com') {
    return res.status(403).send('Forbidden');
  }
  // Process request
});

// 4. Custom request headers (XHR/Fetch)
fetch('/api/data', {
  method: 'POST',
  headers: {
    'X-Requested-With': 'XMLHttpRequest',
    'X-CSRF-Token': getCsrfToken()
  }
});

// 5. Double Submit Cookie
// Cookie and request parameter must match
document.cookie = `csrf_token=${token}`;
fetch('/api', {
  method: 'POST',
  headers: { 'X-CSRF-Token': token }
});
```

**Q: Content Security Policy implementation.**

**A:**

```javascript
// HTTP Header
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' 'unsafe-inline' https://trusted.com;
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  font-src 'self' data:;
  connect-src 'self' https://api.example.com;
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';

// Meta tag
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self' https://trusted.com">

// Report violations
Content-Security-Policy: 
  default-src 'self';
  report-uri /csp-violation-report;

// Gradually enforce
Content-Security-Policy-Report-Only: 
  default-src 'self';
  report-uri /csp-report;

// Handle violations
app.post('/csp-violation-report', (req, res) => {
  console.log('CSP Violation:', req.body);
  // Log to monitoring service
});
```

## Node.js & Backend

### 11. Node.js Fundamentals

**Q: Node.js event loop vs browser.**

**A:**

**Phases of Node.js Event Loop:**
1. **Timers**: setTimeout, setInterval callbacks
2. **Pending callbacks**: I/O callbacks deferred to next iteration
3. **Idle, prepare**: Internal use only
4. **Poll**: Retrieve new I/O events
5. **Check**: setImmediate() callbacks
6. **Close callbacks**: Socket close events

```javascript
setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('Promise'));

// Output: nextTick, Promise, setTimeout/setImmediate (order varies)

// Key Differences:
// - process.nextTick has highest priority (before microtasks)
// - setImmediate runs after I/O events
// - Browser has no setImmediate or process.nextTick
```

**Q: Node.js Streams explained.**

**A:**

```javascript
// Readable Stream
const { Readable } = require('stream');

class NumberStream extends Readable {
  constructor(max) {
    super();
    this.current = 0;
    this.max = max;
  }
  
  _read() {
    if (this.current <= this.max) {
      this.push(String(this.current++));
    } else {
      this.push(null); // End stream
    }
  }
}

// Writable Stream
const { Writable } = require('stream');

class LogStream extends Writable {
  _write(chunk, encoding, callback) {
    console.log(chunk.toString());
    callback();
  }
}

// Transform Stream
const { Transform } = require('stream');

class UpperCaseTransform extends Transform {
  _transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
}

// Duplex Stream (both readable and writable)
const { Duplex } = require('stream');

// Piping streams
const fs = require('fs');
fs.createReadStream('input.txt')
  .pipe(new UpperCaseTransform())
  .pipe(fs.createWriteStream('output.txt'));

// Backpressure handling
const readable = fs.createReadStream('large-file.txt');
const writable = fs.createWriteStream('output.txt');

readable.on('data', (chunk) => {
  const canContinue = writable.write(chunk);
  if (!canContinue) {
    readable.pause(); // Handle backpressure
  }
});

writable.on('drain', () => {
  readable.resume(); // Resume when buffer drains
});
```

**Q: Cluster vs worker_threads.**

**A:**

**Cluster:**
- Multiple processes (separate memory)
- Share server port
- IPC for communication
- Better for I/O-bound tasks
- Isolated crashes

```javascript
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);
  
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // Restart worker
  });
} else {
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('Hello from worker ' + process.pid);
  }).listen(8000);
}
```

**Worker Threads:**
- Multiple threads (shared memory)
- Share memory via SharedArrayBuffer
- For CPU-intensive tasks
- Lighter weight

```javascript
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');

if (isMainThread) {
  // Main thread
  const worker = new Worker(__filename, {
    workerData: { num: 5 }
  });
  
  worker.on('message', (result) => {
    console.log('Result:', result);
  });
} else {
  // Worker thread
  function fibonacci(n) {
    if (n < 2) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
  }
  
  const result = fibonacci(workerData.num);
  parentPort.postMessage(result);
}
```

**Q: process.nextTick() vs setImmediate().**

**A:**

```javascript
// process.nextTick() - Executes before any I/O
// setImmediate() - Executes after I/O in check phase

setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('nextTick'));

// Output: nextTick, setTimeout, setImmediate (or setTimeout/setImmediate may swap)

// Use cases:
// process.nextTick() - Emit events, ensure async execution
process.nextTick(() => {
  emit('event'); // Ensures listeners are registered
});

// setImmediate() - Break up long operations
function processLargeArray(array) {
  const chunk = array.splice(0, 100);
  process(chunk);
  
  if (array.length > 0) {
    setImmediate(() => processLargeArray(array));
  }
}
```

## Testing & Quality

### 12. Testing Strategies

**Q: Unit vs Integration vs E2E testing.**

**A:**

**Unit Testing:**
- Test individual functions/components in isolation
- Fast, focused, many tests
- Mock dependencies

```javascript
// Jest example
describe('Calculator', () => {
  test('adds two numbers', () => {
    expect(add(2, 3)).toBe(5);
  });
  
  test('handles edge cases', () => {
    expect(add(0, 0)).toBe(0);
    expect(add(-1, 1)).toBe(0);
  });
});

// Mock dependencies
jest.mock('./api', () => ({
  fetchUser: jest.fn(() => Promise.resolve({ name: 'John' }))
}));
```

**Integration Testing:**
- Test multiple units working together
- Test API endpoints, database interactions
- Medium speed, fewer tests

```javascript
// Supertest example
const request = require('supertest');
const app = require('./app');

describe('User API', () => {
  test('GET /users returns users', async () => {
    const response = await request(app)
      .get('/users')
      .expect(200);
    
    expect(response.body).toHaveLength(10);
  });
  
  test('POST /users creates user', async () => {
    const user = { name: 'John', email: 'john@example.com' };
    const response = await request(app)
      .post('/users')
      .send(user)
      .expect(201);
    
    expect(response.body.name).toBe('John');
  });
});
```

**E2E Testing:**
- Test entire application flow
- Simulate real user interactions
- Slow, few tests, high confidence

```javascript
// Playwright example
const { test, expect } = require('@playwright/test');

test('user can login and view dashboard', async ({ page }) => {
  await page.goto('https://example.com');
  await page.fill('#username', 'testuser');
  await page.fill('#password', 'password123');
  await page.click('#login-button');
  
  await expect(page).toHaveURL(/.*dashboard/);
  await expect(page.locator('h1')).toContainText('Dashboard');
});
```

**Testing Pyramid:**
- Many unit tests (70%)
- Some integration tests (20%)
- Few E2E tests (10%)

**Q: Testing async code.**

**A:**

```javascript
// Promises
test('fetches user data', () => {
  return fetchUser(1).then(user => {
    expect(user.name).toBe('John');
  });
});

// Async/Await
test('fetches user data', async () => {
  const user = await fetchUser(1);
  expect(user.name).toBe('John');
});

// Error handling
test('handles fetch error', async () => {
  await expect(fetchUser(-1)).rejects.toThrow('User not found');
});

// Multiple async operations
test('processes in parallel', async () => {
  const [user1, user2] = await Promise.all([
    fetchUser(1),
    fetchUser(2)
  ]);
  expect(user1.name).toBe('John');
  expect(user2.name).toBe('Jane');
});

// Timers
jest.useFakeTimers();
test('delays execution', () => {
  const callback = jest.fn();
  setTimeout(callback, 1000);
  
  jest.advanceTimersByTime(1000);
  expect(callback).toHaveBeenCalledTimes(1);
});
```

## Coding Challenges Solutions

### 1. Deep Clone with Circular References

```javascript
function deepClone(obj, hash = new WeakMap()) {
  // Handle primitives and null
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  // Handle Date
  if (obj instanceof Date) {
    return new Date(obj.getTime());
  }
  
  // Handle RegExp
  if (obj instanceof RegExp) {
    return new RegExp(obj.source, obj.flags);
  }
  
  // Handle circular references
  if (hash.has(obj)) {
    return hash.get(obj);
  }
  
  // Handle Arrays
  if (Array.isArray(obj)) {
    const arrCopy = [];
    hash.set(obj, arrCopy);
    obj.forEach((item, index) => {
      arrCopy[index] = deepClone(item, hash);
    });
    return arrCopy;
  }
  
  // Handle Objects
  const objCopy = Object.create(Object.getPrototypeOf(obj));
  hash.set(obj, objCopy);
  
  Object.keys(obj).forEach(key => {
    objCopy[key] = deepClone(obj[key], hash);
  });
  
  // Handle Symbols
  Object.getOwnPropertySymbols(obj).forEach(symbol => {
    objCopy[symbol] = deepClone(obj[symbol], hash);
  });
  
  return objCopy;
}

// Test
const obj = { a: 1, b: { c: 2 } };
obj.self = obj; // Circular reference
const cloned = deepClone(obj);
```

### 2. Debounce and Throttle

```javascript
// Debounce - Execute after delay, reset on new calls
function debounce(func, delay) {
  let timeoutId;
  
  return function debounced(...args) {
    const context = this;
    
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      func.apply(context, args);
    }, delay);
  };
}

// With immediate execution option
function debounce(func, delay, immediate = false) {
  let timeoutId;
  
  return function debounced(...args) {
    const context = this;
    const callNow = immediate && !timeoutId;
    
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      timeoutId = null;
      if (!immediate) {
        func.apply(context, args);
      }
    }, delay);
    
    if (callNow) {
      func.apply(context, args);
    }
  };
}

// Throttle - Execute at most once per interval
function throttle(func, limit) {
  let inThrottle;
  let lastResult;
  
  return function throttled(...args) {
    const context = this;
    
    if (!inThrottle) {
      lastResult = func.apply(context, args);
      inThrottle = true;
      
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
    
    return lastResult;
  };
}

// Advanced throttle with leading and trailing options
function throttle(func, limit, options = {}) {
  let timeout;
  let previous = 0;
  
  return function throttled(...args) {
    const now = Date.now();
    const context = this;
    
    if (!previous && options.leading === false) {
      previous = now;
    }
    
    const remaining = limit - (now - previous);
    
    if (remaining <= 0 || remaining > limit) {
      if (timeout) {
        clearTimeout(timeout);
        timeout = null;
      }
      previous = now;
      func.apply(context, args);
    } else if (!timeout && options.trailing !== false) {
      timeout = setTimeout(() => {
        previous = options.leading === false ? 0 : Date.now();
        timeout = null;
        func.apply(context, args);
      }, remaining);
    }
  };
}
```

### 3. Promise.all() Implementation

```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError('Argument must be an array'));
    }
    
    const results = [];
    let completed = 0;
    
    if (promises.length === 0) {
      return resolve(results);
    }
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = value;
          completed++;
          
          if (completed === promises.length) {
            resolve(results);
          }
        })
        .catch(error => {
          reject(error);
        });
    });
  });
}

// Promise.allSettled implementation
function promiseAllSettled(promises) {
  return Promise.all(
    promises.map(promise =>
      Promise.resolve(promise)
        .then(value => ({ status: 'fulfilled', value }))
        .catch(reason => ({ status: 'rejected', reason }))
    )
  );
}

// Promise.race implementation
function promiseRace(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError('Argument must be an array'));
    }
    
    promises.forEach(promise => {
      Promise.resolve(promise)
        .then(resolve)
        .catch(reject);
    });
  });
}
```

### 4. Event Emitter

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
    return this;
  }
  
  once(event, listener) {
    const onceWrapper = (...args) => {
      listener.apply(this, args);
      this.off(event, onceWrapper);
    };
    return this.on(event, onceWrapper);
  }
  
  off(event, listenerToRemove) {
    if (!this.events[event]) return this;
    
    this.events[event] = this.events[event].filter(
      listener => listener !== listenerToRemove
    );
    
    return this;
  }
  
  emit(event, ...args) {
    if (!this.events[event]) return false;
    
    this.events[event].forEach(listener => {
      listener.apply(this, args);
    });
    
    return true;
  }
  
  removeAllListeners(event) {
    if (event) {
      delete this.events[event];
    } else {
      this.events = {};
    }
    return this;
  }
  
  listenerCount(event) {
    return this.events[event] ? this.events[event].length : 0;
  }
  
  listeners(event) {
    return this.events[event] ? [...this.events[event]] : [];
  }
}

// Usage
const emitter = new EventEmitter();

emitter.on('data', (data) => console.log('Received:', data));
emitter.once('data', (data) => console.log('Once:', data));

emitter.emit('data', { message: 'Hello' });
emitter.emit('data', { message: 'World' });
```

### 5. Flatten Nested Array

```javascript
// Recursive approach
function flatten(arr, depth = Infinity) {
  if (depth === 0) return arr;
  
  return arr.reduce((acc, item) => {
    if (Array.isArray(item)) {
      return acc.concat(flatten(item, depth - 1));
    }
    return acc.concat(item);
  }, []);
}

// Iterative approach with stack
function flattenIterative(arr) {
  const stack = [...arr];
  const result = [];
  
  while (stack.length) {
    const item = stack.pop();
    
    if (Array.isArray(item)) {
      stack.push(...item);
    } else {
      result.unshift(item);
    }
  }
  
  return result;
}

// Using generator
function* flattenGenerator(arr, depth = Infinity) {
  for (const item of arr) {
    if (Array.isArray(item) && depth > 0) {
      yield* flattenGenerator(item, depth - 1);
    } else {
      yield item;
    }
  }
}

// Test
flatten([1, [2, [3, [4]], 5]]); // [1, 2, 3, 4, 5]
flatten([1, [2, [3, [4]], 5]], 2); // [1, 2, 3, [4], 5]

### 6. LRU Cache

```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }
  
  get(key) {
    if (!this.cache.has(key)) {
      return -1;
    }
    
    // Move to end (most recently used)
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    
    return value;
  }
  
  put(key, value) {
    // Delete if exists (to reorder)
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }
    
    // Add to end
    this.cache.set(key, value);
    
    // Remove least recently used if over capacity
    if (this.cache.size > this.capacity) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
  }
  
  // Additional methods
  has(key) {
    return this.cache.has(key);
  }
  
  clear() {
    this.cache.clear();
  }
  
  size() {
    return this.cache.size;
  }
}

// Alternative implementation with doubly linked list for O(1) operations
class Node {
  constructor(key, value) {
    this.key = key;
    this.value = value;
    this.prev = null;
    this.next = null;
  }
}

class LRUCacheOptimized {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
    this.head = new Node(null, null);
    this.tail = new Node(null, null);
    this.head.next = this.tail;
    this.tail.prev = this.head;
  }
  
  _removeNode(node) {
    node.prev.next = node.next;
    node.next.prev = node.prev;
  }
  
  _addToHead(node) {
    node.next = this.head.next;
    node.prev = this.head;
    this.head.next.prev = node;
    this.head.next = node;
  }
  
  get(key) {
    if (!this.cache.has(key)) {
      return -1;
    }
    
    const node = this.cache.get(key);
    this._removeNode(node);
    this._addToHead(node);
    
    return node.value;
  }
  
  put(key, value) {
    if (this.cache.has(key)) {
      const node = this.cache.get(key);
      node.value = value;
      this._removeNode(node);
      this._addToHead(node);
    } else {
      const newNode = new Node(key, value);
      this.cache.set(key, newNode);
      this._addToHead(newNode);
      
      if (this.cache.size > this.capacity) {
        const lru = this.tail.prev;
        this._removeNode(lru);
        this.cache.delete(lru.key);
      }
    }
  }
}

// Usage
const cache = new LRUCache(2);
cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // returns 1
cache.put(3, 3);    // evicts key 2
cache.get(2);       // returns -1 (not found)
```

### 7. Observable Pattern

```javascript
class Observable {
  constructor(subscribe) {
    this._subscribe = subscribe;
  }
  
  subscribe(observer) {
    return this._subscribe(observer);
  }
  
  static create(subscribe) {
    return new Observable(subscribe);
  }
  
  static fromArray(array) {
    return new Observable(observer => {
      array.forEach(item => observer.next(item));
      observer.complete();
      
      return { unsubscribe: () => {} };
    });
  }
  
  static fromEvent(element, eventName) {
    return new Observable(observer => {
      const handler = (event) => observer.next(event);
      element.addEventListener(eventName, handler);
      
      return {
        unsubscribe: () => {
          element.removeEventListener(eventName, handler);
        }
      };
    });
  }
  
  static interval(period) {
    return new Observable(observer => {
      let count = 0;
      const intervalId = setInterval(() => {
        observer.next(count++);
      }, period);
      
      return {
        unsubscribe: () => clearInterval(intervalId)
      };
    });
  }
  
  map(transformFn) {
    return new Observable(observer => {
      return this.subscribe({
        next: (value) => observer.next(transformFn(value)),
        error: (err) => observer.error(err),
        complete: () => observer.complete()
      });
    });
  }
  
  filter(predicateFn) {
    return new Observable(observer => {
      return this.subscribe({
        next: (value) => {
          if (predicateFn(value)) {
            observer.next(value);
          }
        },
        error: (err) => observer.error(err),
        complete: () => observer.complete()
      });
    });
  }
  
  take(count) {
    return new Observable(observer => {
      let taken = 0;
      const subscription = this.subscribe({
        next: (value) => {
          if (taken < count) {
            observer.next(value);
            taken++;
            if (taken === count) {
              observer.complete();
              subscription.unsubscribe();
            }
          }
        },
        error: (err) => observer.error(err),
        complete: () => observer.complete()
      });
      
      return subscription;
    });
  }
  
  debounce(duration) {
    return new Observable(observer => {
      let timeoutId;
      
      return this.subscribe({
        next: (value) => {
          clearTimeout(timeoutId);
          timeoutId = setTimeout(() => {
            observer.next(value);
          }, duration);
        },
        error: (err) => observer.error(err),
        complete: () => observer.complete()
      });
    });
  }
}

// Usage
const observable = Observable.create(observer => {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  
  setTimeout(() => {
    observer.next(4);
    observer.complete();
  }, 1000);
  
  return {
    unsubscribe: () => console.log('Unsubscribed')
  };
});

const subscription = observable
  .map(x => x * 2)
  .filter(x => x > 2)
  .subscribe({
    next: (value) => console.log('Next:', value),
    error: (err) => console.error('Error:', err),
    complete: () => console.log('Complete!')
  });

// Later: subscription.unsubscribe();
```

### 8. Retry Mechanism with Exponential Backoff

```javascript
async function retry(fn, options = {}) {
  const {
    maxAttempts = 3,
    delay = 1000,
    backoff = 2,
    maxDelay = 30000,
    onRetry = () => {}
  } = options;
  
  let lastError;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      
      if (attempt === maxAttempts) {
        throw error;
      }
      
      const waitTime = Math.min(
        delay * Math.pow(backoff, attempt - 1),
        maxDelay
      );
      
      onRetry({
        attempt,
        error,
        waitTime
      });
      
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
  }
  
  throw lastError;
}

// Advanced retry with conditional retry logic
async function retryAdvanced(fn, options = {}) {
  const {
    maxAttempts = 3,
    delay = 1000,
    backoff = 2,
    shouldRetry = () => true,
    onRetry = () => {},
    timeout = null
  } = options;
  
  let lastError;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      if (timeout) {
        return await Promise.race([
          fn(),
          new Promise((_, reject) =>
            setTimeout(() => reject(new Error('Timeout')), timeout)
          )
        ]);
      }
      return await fn();
    } catch (error) {
      lastError = error;
      
      if (attempt === maxAttempts || !shouldRetry(error, attempt)) {
        throw error;
      }
      
      const waitTime = delay * Math.pow(backoff, attempt - 1);
      onRetry({ attempt, error, waitTime });
      
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
  }
  
  throw lastError;
}

// Usage examples
async function fetchData() {
  const response = await fetch('https://api.example.com/data');
  if (!response.ok) throw new Error('Failed to fetch');
  return response.json();
}

// Basic retry
try {
  const data = await retry(fetchData, {
    maxAttempts: 5,
    delay: 1000,
    backoff: 2,
    onRetry: ({ attempt, error, waitTime }) => {
      console.log(`Attempt ${attempt} failed. Retrying in ${waitTime}ms...`);
    }
  });
} catch (error) {
  console.error('All retry attempts failed:', error);
}

// Advanced retry with conditional logic
try {
  const data = await retryAdvanced(fetchData, {
    maxAttempts: 5,
    shouldRetry: (error, attempt) => {
      // Only retry on network errors, not 404s
      return error.message.includes('network') || error.message.includes('timeout');
    },
    timeout: 5000
  });
} catch (error) {
  console.error('Failed:', error);
}

// Retry with jitter (prevents thundering herd)
async function retryWithJitter(fn, options = {}) {
  const {
    maxAttempts = 3,
    delay = 1000,
    maxDelay = 30000
  } = options;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxAttempts) throw error;
      
      const exponentialDelay = delay * Math.pow(2, attempt - 1);
      const jitter = Math.random() * exponentialDelay;
      const waitTime = Math.min(exponentialDelay + jitter, maxDelay);
      
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
  }
}
```

### 9. Memory Leak Detection

```javascript
// Memory Leak Detector
class MemoryLeakDetector {
  constructor() {
    this.snapshots = [];
    this.objects = new WeakMap();
  }
  
  // Track object allocation
  track(obj, name) {
    this.objects.set(obj, { name, timestamp: Date.now() });
  }
  
  // Take heap snapshot
  takeSnapshot() {
    const snapshot = {
      timestamp: Date.now(),
      memory: performance.memory ? {
        usedJSHeapSize: performance.memory.usedJSHeapSize,
        totalJSHeapSize: performance.memory.totalJSHeapSize
      } : null
    };
    this.snapshots.push(snapshot);
    return snapshot;
  }
  
  // Detect growing memory over time
  detectGrowth(threshold = 1.5) {
    if (this.snapshots.length < 2) {
      return { detected: false, message: 'Need more snapshots' };
    }
    
    const first = this.snapshots[0];
    const last = this.snapshots[this.snapshots.length - 1];
    
    if (!first.memory || !last.memory) {
      return { detected: false, message: 'Memory API not available' };
    }
    
    const growthRatio = last.memory.usedJSHeapSize / first.memory.usedJSHeapSize;
    
    return {
      detected: growthRatio > threshold,
      growthRatio,
      initialSize: first.memory.usedJSHeapSize,
      currentSize: last.memory.usedJSHeapSize,
      difference: last.memory.usedJSHeapSize - first.memory.usedJSHeapSize
    };
  }
  
  // Monitor specific patterns
  static checkCommonLeaks(code) {
    const leaks = [];
    
    // Check for global variables
    if (/^\s*\w+\s*=/.test(code) && !/var|let|const/.test(code)) {
      leaks.push({
        type: 'global_variable',
        message: 'Potential global variable detected',
        severity: 'medium'
      });
    }
    
    // Check for forgotten timers
    if (/setInterval|setTimeout/.test(code) && !/clearInterval|clearTimeout/.test(code)) {
      leaks.push({
        type: 'timer_leak',
        message: 'Timer without clear function',
        severity: 'high'
      });
    }
    
    // Check for event listeners
    if (/addEventListener/.test(code) && !/removeEventListener/.test(code)) {
      leaks.push({
        type: 'event_listener',
        message: 'Event listener without removal',
        severity: 'medium'
      });
    }
    
    // Check for large closures
    if (/function.*\{[\s\S]{1000,}\}/.test(code)) {
      leaks.push({
        type: 'large_closure',
        message: 'Large function closure detected',
        severity: 'low'
      });
    }
    
    return leaks;
  }
}

// Example usage
const detector = new MemoryLeakDetector();

// Monitor over time
setInterval(() => {
  detector.takeSnapshot();
  const result = detector.detectGrowth();
  
  if (result.detected) {
    console.warn('Memory leak detected!', result);
  }
}, 5000);

// Static analysis
const code = `
  setInterval(() => {
    data = fetchData(); // Global variable
  }, 1000);
`;

const leaks = MemoryLeakDetector.checkCommonLeaks(code);
console.log('Detected leaks:', leaks);

// Practical leak detection helpers
function findDetachedDOMNodes() {
  const walker = document.createTreeWalker(
    document.body,
    NodeFilter.SHOW_ELEMENT
  );
  
  const detached = [];
  let node;
  
  while (node = walker.nextNode()) {
    if (!document.body.contains(node)) {
      detached.push(node);
    }
  }
  
  return detached;
}

function findEventListenerLeaks() {
  const leaks = [];
  
  // Check elements that might have listeners
  document.querySelectorAll('*').forEach(element => {
    const listeners = getEventListeners?.(element);
    if (listeners && Object.keys(listeners).length > 0) {
      leaks.push({
        element,
        listeners: Object.keys(listeners)
      });
    }
  });
  
  return leaks;
}
```

### 10. State Management Library (Redux-like)

```javascript
class Store {
  constructor(reducer, initialState = {}) {
    this.reducer = reducer;
    this.state = initialState;
    this.listeners = [];
    this.middlewares = [];
  }
  
  getState() {
    return this.state;
  }
  
  dispatch(action) {
    // Apply middlewares
    let dispatch = (action) => {
      this.state = this.reducer(this.state, action);
      this.listeners.forEach(listener => listener());
    };
    
    // Create middleware chain
    this.middlewares.reverse().forEach(middleware => {
      dispatch = middleware(this)(dispatch);
    });
    
    return dispatch(action);
  }
  
  subscribe(listener) {
    this.listeners.push(listener);
    
    // Return unsubscribe function
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener);
    };
  }
  
  applyMiddleware(...middlewares) {
    this.middlewares = middlewares;
  }
}

// Middleware examples
const loggerMiddleware = store => next => action => {
  console.log('Dispatching:', action);
  const result = next(action);
  console.log('New state:', store.getState());
  return result;
};

const thunkMiddleware = store => next => action => {
  if (typeof action === 'function') {
    return action(store.dispatch, store.getState);
  }
  return next(action);
};

const asyncMiddleware = store => next => action => {
  if (action.type && action.type.endsWith('_ASYNC')) {
    const { type, payload } = action;
    const baseType = type.replace('_ASYNC', '');
    
    store.dispatch({ type: `${baseType}_PENDING` });
    
    return payload()
      .then(result => {
        store.dispatch({ type: `${baseType}_SUCCESS`, payload: result });
        return result;
      })
      .catch(error => {
        store.dispatch({ type: `${baseType}_FAILURE`, payload: error });
        throw error;
      });
  }
  
  return next(action);
};

// Helper to combine reducers
function combineReducers(reducers) {
  return (state = {}, action) => {
    return Object.keys(reducers).reduce((nextState, key) => {
      nextState[key] = reducers[key](state[key], action);
      return nextState;
    }, {});
  };
}

// Usage example
const counterReducer = (state = { count: 0 }, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'ADD':
      return { count: state.count + action.payload };
    default:
      return state;
  }
};

const userReducer = (state = { name: '', loggedIn: false }, action) => {
  switch (action.type) {
    case 'LOGIN':
      return { name: action.payload, loggedIn: true };
    case 'LOGOUT':
      return { name: '', loggedIn: false };
    default:
      return state;
  }
};

const rootReducer = combineReducers({
  counter: counterReducer,
  user: userReducer
});

const store = new Store(rootReducer);
store.applyMiddleware(loggerMiddleware, thunkMiddleware, asyncMiddleware);

// Subscribe to changes
const unsubscribe = store.subscribe(() => {
  console.log('State changed:', store.getState());
});

// Dispatch actions
store.dispatch({ type: 'INCREMENT' });
store.dispatch({ type: 'ADD', payload: 5 });
store.dispatch({ type: 'LOGIN', payload: 'John' });

// Thunk action
const incrementAsync = (delay) => (dispatch) => {
  setTimeout(() => {
    dispatch({ type: 'INCREMENT' });
  }, delay);
};

store.dispatch(incrementAsync(1000));

// Async action
store.dispatch({
  type: 'FETCH_USER_ASYNC',
  payload: () => fetch('/api/user').then(r => r.json())
});

// Clean up
unsubscribe();
```

---

*These comprehensive answers demonstrate deep JavaScript knowledge, practical problem-solving abilities, and the kind of nuanced understanding expected from a senior engineer with 15+ years of experience.*