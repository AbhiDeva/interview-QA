# JavaScript Interview Questions & Answers
## For Senior Engineers (15 Years Experience)

---

## Core JavaScript Concepts

### 1. Explain the Event Loop and how it handles asynchronous operations

**Answer:** The Event Loop is JavaScript's concurrency model that handles asynchronous operations in a single-threaded environment. It consists of the call stack, task queue (macrotask queue), microtask queue, and Web APIs.

The process works as follows: synchronous code executes on the call stack first. When asynchronous operations like setTimeout or network requests are encountered, they're handed off to Web APIs. Once completed, callbacks are placed in the task queue. Microtasks (promises, queueMicrotask) have higher priority than macrotasks (setTimeout, setInterval). After each macrotask, the event loop processes all microtasks before moving to the next macrotask.

Example flow: Promise callbacks execute before setTimeout callbacks even if setTimeout has a delay of 0ms, because microtasks are processed immediately after the current script and before any macrotasks.

### 2. What are the differences between `var`, `let`, and `const`? Discuss hoisting behavior.

**Answer:** `var` is function-scoped, hoisted to the top of its scope, and initialized with undefined. `let` and `let` are block-scoped, hoisted but not initialized (temporal dead zone), and cannot be accessed before declaration.

`const` requires initialization at declaration and prevents reassignment, though object properties can still be mutated. `var` allows redeclaration within the same scope, while `let` and `const` do not.

The temporal dead zone for `let` and `const` means accessing them before declaration throws a ReferenceError, unlike `var` which returns undefined. This makes `let` and `const` safer and more predictable.

### 3. Explain prototypal inheritance and how it differs from classical inheritance

**Answer:** JavaScript uses prototypal inheritance where objects inherit directly from other objects through the prototype chain. Every object has an internal [[Prototype]] property that references another object.

When accessing a property, JavaScript first looks at the object itself, then traverses up the prototype chain until found or reaching null. This differs from classical inheritance where classes inherit from classes, creating a blueprint-instance relationship.

Prototypal inheritance is more flexible, allowing runtime modification of inheritance relationships through Object.create() or modifying the prototype. The class syntax introduced in ES6 is syntactic sugar over prototypal inheritance, not a new inheritance model.

### 4. What is closure and provide a practical use case?

**Answer:** A closure is a function that retains access to variables from its outer lexical scope even after the outer function has returned. This happens because JavaScript maintains references to these variables in memory.

Practical use case: creating private variables and methods in module patterns. For example, a counter factory function can return methods that access a private count variable, ensuring it cannot be directly modified from outside. Closures are also fundamental to currying, partial application, event handlers that need to remember state, and React hooks like useState which maintain state across renders.

### 5. Explain `this` binding rules in JavaScript

**Answer:** The value of `this` is determined by how a function is called, not where it's defined. There are four binding rules in order of precedence:

1. **new binding**: When using `new`, `this` refers to the newly created object
2. **Explicit binding**: Using call(), apply(), or bind() explicitly sets `this`
3. **Implicit binding**: When called as a method, `this` refers to the object before the dot
4. **Default binding**: In strict mode `this` is undefined, otherwise it's the global object

Arrow functions don't have their own `this` binding and inherit it lexically from the enclosing scope. This makes them unsuitable for methods but perfect for callbacks where you want to preserve outer context.

---

## Advanced Patterns & Architecture

### 6. Explain the differences between debouncing and throttling, and when to use each

**Answer:** Debouncing delays function execution until after a specified time has passed since the last invocation. It's ideal for search input fields where you want to wait until the user stops typing before making an API call, reducing unnecessary requests.

Throttling ensures a function executes at most once per specified time interval, regardless of how many times it's triggered. It's perfect for scroll handlers or resize events where you want consistent execution at regular intervals without overwhelming the system.

Key difference: debouncing waits for silence, throttling maintains a steady rhythm. Use debouncing for user input validation, autocomplete, or form submissions. Use throttling for infinite scrolling, progress tracking, or performance-critical event handlers.

### 7. What is the Module pattern and what are its benefits?

**Answer:** The Module pattern uses closures to create private and public members, encapsulating implementation details while exposing a clean API. It returns an object literal with public methods that have access to private variables through closure.

Benefits include data privacy (no direct access to internal state), namespace management (avoiding global scope pollution), organization (grouping related functionality), and dependency management (clear interface contracts).

Modern JavaScript has ES6 modules with import/export, which provide similar benefits with better tooling support, static analysis, and tree-shaking capabilities. However, understanding the classical module pattern helps grasp the foundations of encapsulation in JavaScript.

### 8. Explain memoization and provide an implementation

**Answer:** Memoization is an optimization technique that caches function results based on input arguments, avoiding expensive recalculations for repeated calls with the same inputs.

Basic implementation: create a cache object, check if the result exists for given arguments, return cached result if available, otherwise compute, store, and return. This is particularly effective for recursive algorithms like Fibonacci, expensive computations, or API calls with repetitive parameters.

Considerations include cache invalidation strategies, memory consumption for large datasets, serialization of complex arguments for cache keys, and whether the function is pure (same inputs always produce same outputs). Libraries like lodash provide robust memoize implementations with configurable resolvers.

### 9. What are Promises and how do they improve upon callbacks?

**Answer:** Promises represent the eventual completion or failure of an asynchronous operation, providing a cleaner alternative to callback-based patterns. They have three states: pending, fulfilled, or rejected, and can only transition once.

Advantages over callbacks include avoiding callback hell through chaining, better error handling with catch blocks that propagate through the chain, composition through Promise.all/race/allSettled, and improved readability with async/await syntax.

Promises are eager (execute immediately when created) and single-value (resolve once). They enable powerful patterns like retry logic, timeout wrappers, and concurrent execution with controlled parallelism. Understanding promise microtask scheduling is crucial for predictable async behavior.

### 10. Explain async/await and how it handles errors

**Answer:** async/await is syntactic sugar over Promises that allows writing asynchronous code in a synchronous style. An async function always returns a Promise, and await pauses execution until the Promise resolves.

Error handling uses try/catch blocks, which is more intuitive than promise.catch() chains. Unhandled promise rejections in async functions propagate as rejected promises, so top-level error boundaries are important.

Key considerations: await blocks execution sequentially, so independent operations should use Promise.all() for parallelism. Multiple awaits in sequence can create performance bottlenecks. Avoid using async/await in loops for concurrent operations unless you specifically need sequential execution.

---

## Performance & Optimization

### 11. What is the difference between `==` and `===`? Explain type coercion.

**Answer:** The `===` operator checks for strict equality without type coercion, comparing both value and type. The `==` operator performs type coercion before comparison, following complex conversion rules that can lead to unexpected results.

Type coercion rules include converting primitives to numbers for comparison, special handling of null and undefined (they equal each other but nothing else with `==`), and string-to-number conversions. Classic pitfalls include `0 == false` being true, or `"0" == false` being true.

Best practice is to always use `===` for predictable behavior and explicit comparisons. The only exception is checking for null or undefined simultaneously with `value == null`, which is equivalent to `value === null || value === undefined`.

### 12. Explain shallow copy vs deep copy and implementation approaches

**Answer:** Shallow copy creates a new object but copies references for nested objects, meaning changes to nested structures affect both copies. Deep copy recursively copies all nested structures, creating completely independent copies.

Shallow copy methods include spread operator, Object.assign(), or Array.slice(). These work fine for flat structures but share references for nested objects.

Deep copy approaches include JSON.parse(JSON.stringify()) which is simple but fails with functions, dates, undefined, and circular references. Recursive implementation handles these cases but requires circular reference detection. Modern structuredClone() API provides native deep cloning support with better performance and broader compatibility than JSON methods.

### 13. What are Web Workers and when should you use them?

**Answer:** Web Workers enable running JavaScript in background threads separate from the main UI thread, preventing blocking operations from freezing the interface. They communicate via message passing since they don't share memory with the main thread.

Use cases include heavy computation (image processing, data parsing, encryption), long-running background tasks, real-time data processing, or offloading work to maintain 60fps UI performance.

Limitations include no DOM access, message passing overhead for large data (consider Transferable objects), separate global scope, and additional complexity in code organization. SharedWorkers enable communication between multiple browser contexts, while Service Workers handle network requests and enable offline functionality.

### 14. Explain memory leaks in JavaScript and how to prevent them

**Answer:** Memory leaks occur when allocated memory isn't released, typically from unintended references preventing garbage collection. Common causes include forgotten timers (setInterval not cleared), detached DOM nodes still referenced, closures holding unnecessary references, and global variables accumulating data.

Prevention strategies include clearing timers/intervals when done, removing event listeners when components unmount, being careful with closures to not capture entire scopes unnecessarily, using WeakMap/WeakSet for object keys that shouldn't prevent garbage collection, and avoiding global variables.

Detection tools include Chrome DevTools heap snapshots, performance monitoring for memory growth over time, and analyzing retainer trees to understand what's keeping objects alive. In frameworks like React, proper cleanup in useEffect return functions prevents most common leaks.

### 15. What is tree shaking and how does it work?

**Answer:** Tree shaking is dead code elimination during the build process, removing unused exports from the final bundle. It relies on ES6 module syntax which is statically analyzable, unlike CommonJS require() which is dynamic.

Webpack and other bundlers analyze import/export statements to determine which code paths are actually used, eliminating unused functions and modules. This significantly reduces bundle size for production applications.

For effective tree shaking, use ES6 imports, avoid side effects in modules, configure sideEffects field in package.json, use named exports over default exports when possible, and ensure libraries support ESM format. Dynamic imports prevent tree shaking for that path.

---

## ES6+ Features

### 16. Explain destructuring assignment and provide advanced examples

**Answer:** Destructuring extracts values from arrays or properties from objects into distinct variables using concise syntax. It supports default values, renaming, rest operators, and nested patterns.

Advanced patterns include: swapping variables without temp variable using array destructuring, extracting deep nested properties with fallbacks, combining with rest operator for partial extraction, destructuring function parameters for cleaner APIs, and using in loops to iterate over object entries.

It improves code readability, reduces repetitive property access, enables clear function signatures showing expected structure, and works seamlessly with modules for selective imports. Common gotcha: destructuring undefined throws an error unless you provide defaults or use optional chaining.

### 17. What are Symbols and what problems do they solve?

**Answer:** Symbols are primitive values guaranteed to be unique, primarily used as object property keys to avoid naming collisions. Unlike strings, two symbols are never equal even with identical descriptions.

Use cases include creating private-like properties (not truly private but hidden from Object.keys/JSON.stringify), defining custom iteration behavior with Symbol.iterator, implementing well-known symbols like Symbol.toPrimitive for type coercion, and creating internal protocol hooks in libraries without risking property name conflicts.

Symbols don't appear in for-in loops or Object.keys(), though Object.getOwnPropertySymbols() can access them. Well-known symbols enable customizing language behavior, like making custom objects iterable or defining how they convert to primitives.

### 18. Explain Generators and their practical applications

**Answer:** Generators are functions that can pause and resume execution, yielding values on demand using function* syntax and yield keyword. They implement the iterator protocol, making them naturally iterable.

Practical applications include lazy evaluation (generating infinite sequences without memory overhead), custom iteration logic (tree traversal, pagination), asynchronous flow control (before async/await, libraries like co used generators), state machines, and implementing complex iterators without managing state manually.

Generators can yield and receive values, enabling bidirectional communication. They're memory efficient for large datasets since values are generated on-demand. Redux-Saga uses generators for handling side effects in a testable, declarative way.

### 19. What are Proxies and Reflect API? Provide use cases.

**Answer:** Proxies allow intercepting and customizing fundamental operations on objects like property access, assignment, function invocation, and more through handler traps. Reflect API provides methods for interceptable JavaScript operations, mirroring Proxy traps.

Use cases include validation (enforce rules on property setting), logging and debugging (track property access), implementing private properties, creating reactive systems (Vue 3 reactivity), data binding, lazy loading properties, deprecation warnings, and implementing custom property access patterns.

Proxy traps include get, set, has, deleteProperty, apply, construct, and more. Reflect methods ensure proper behavior when forwarding operations, maintaining correct this binding and return values. Performance consideration: proxies have overhead, so avoid wrapping performance-critical paths unnecessarily.

### 20. Explain the difference between Map/Set and Object/Array

**Answer:** Map maintains key-value pairs where keys can be any type (objects, functions, primitives), unlike objects which coerce keys to strings. Map preserves insertion order, has a size property, and provides iteration methods. It's better for frequent additions/deletions and when key types matter.

Set stores unique values of any type, automatically handling uniqueness. Unlike arrays, checking for existence is O(1) instead of O(n), making it ideal for deduplication and membership tests.

Objects are optimized for string/symbol keys and are better for JSON serialization. Arrays maintain order and provide rich manipulation methods. Choose Map for dictionary-like structures with non-string keys, Set for unique collections with fast lookups, Object for JSON-compatible structures, and Array for ordered lists with index access.

---

## Design Patterns & Best Practices

### 21. Explain the Observer pattern and its implementation in JavaScript

**Answer:** The Observer pattern defines one-to-many dependency where multiple observers subscribe to a subject and receive notifications when state changes. It decouples the subject from its observers, enabling loose coupling and extensibility.

Implementation involves a subject maintaining an observers list with subscribe/unsubscribe/notify methods. Observers implement an update method called by the subject. Modern JavaScript uses EventEmitter pattern in Node.js or custom event systems.

Real-world examples include DOM events, RxJS observables, Redux store subscriptions, and custom event systems. Benefits include loose coupling, dynamic relationships, and broadcast communication. Considerations include potential memory leaks from forgotten subscriptions and performance with many observers.

### 22. What is Functional Programming and its core principles in JavaScript?

**Answer:** Functional programming treats computation as evaluation of mathematical functions, avoiding state mutation and side effects. Core principles include pure functions (same input always produces same output with no side effects), immutability (data never changes, new copies created), first-class functions, higher-order functions, and function composition.

JavaScript supports FP through first-class functions, closures, array methods (map/filter/reduce), and tools like Ramda or lodash/fp. Benefits include predictability, testability, easier debugging, parallel execution safety, and better code reusability.

Techniques include avoiding mutations (use spread/Object.assign), preferring declarative over imperative code, using pure functions, composing functions for complex operations, and leveraging techniques like currying and partial application. Trade-offs include potential performance overhead from creating new objects and learning curve for imperative programmers.

### 23. Explain currying and partial application with examples

**Answer:** Currying transforms a function taking multiple arguments into a sequence of functions each taking a single argument. Partial application fixes some arguments of a function, producing a new function with fewer parameters.

Currying enables creating specialized functions from general ones, improves reusability, and enables point-free programming. Example: a curried add function called as add(2)(3) versus add(2, 3).

Partial application is useful for configuration functions, creating specialized versions of utilities, and dependency injection. Both techniques leverage closures to remember arguments across calls. Libraries like lodash provide curry and partial implementations. These patterns shine in functional pipelines and when building configurable APIs.

### 24. What are the SOLID principles and how do they apply to JavaScript?

**Answer:** SOLID principles guide object-oriented design for maintainable, scalable code:

**Single Responsibility**: Each module/class/function should have one reason to change. In JavaScript, keep functions focused on single tasks and separate concerns into different modules.

**Open/Closed**: Open for extension, closed for modification. Use composition, inheritance, or plugins rather than modifying existing code.

**Liskov Substitution**: Subtypes should be substitutable for base types. Ensure derived classes honor base class contracts.

**Interface Segregation**: Prefer specific interfaces over general ones. In JavaScript, don't force implementations to depend on methods they don't use.

**Dependency Inversion**: Depend on abstractions, not concretions. Use dependency injection rather than hardcoding dependencies.

JavaScript's dynamic nature makes some principles less rigid than in statically-typed languages, but the underlying goals of maintainability and modularity remain crucial.

### 25. Explain the differences between imperative and declarative programming

**Answer:** Imperative programming describes how to achieve a result through explicit step-by-step instructions controlling flow. You manage loops, conditions, and state mutations directly.

Declarative programming describes what result you want, abstracting implementation details. SQL queries, React JSX, and array methods like map/filter are declarative.

Example: imperative loop with index and accumulator versus declarative filter().map() chain. Declarative code is often more readable, easier to reason about, and less error-prone since implementation details are handled by well-tested libraries.

JavaScript supports both paradigms. Modern trends favor declarative approaches for clarity, though imperative code can be more performant in critical sections. The best code often combines both, using declarative style for clarity and imperative where performance demands it.

---

## Testing & Debugging

### 26. Explain different types of testing and testing strategies

**Answer:** Unit tests verify individual functions/modules in isolation, mocking dependencies. They're fast, focused, and catch regressions early. Use frameworks like Jest or Vitest.

Integration tests verify multiple components working together, testing interactions and data flow. They catch issues at boundaries between modules.

End-to-end tests simulate real user workflows through the entire application stack, catching integration issues but are slower and more brittle. Tools include Playwright or Cypress.

Testing strategies include Test-Driven Development (write tests first), behavior-driven development (focus on user scenarios), testing pyramid (many unit tests, fewer integration, fewer e2e), and maintaining good test coverage without chasing 100% blindly. Balance speed, confidence, and maintainability.

### 27. What are common debugging techniques and tools?

**Answer:** Console methods beyond log include dir (object structure), table (tabular data), time/timeEnd (performance), trace (call stack), and assert (conditional logging). Strategic logging at key points helps trace execution flow.

Chrome DevTools offers breakpoints (conditional, logpoints), step through execution, watch expressions, call stack inspection, and scope variable examination. Performance profiling identifies bottlenecks, memory profiling finds leaks.

Source maps enable debugging transpiled code. Network panel inspects requests. React DevTools and Vue DevTools help debug component hierarchies and state.

Debugging strategies include reproducing issues consistently, isolating problems through binary search (commenting code sections), rubber duck debugging (explaining the problem), and reading error stack traces carefully for root causes.

### 28. How do you handle errors in JavaScript applications?

**Answer:** Error handling strategies include try-catch for synchronous code and specific error conditions, promise.catch() chains for async operations, async/await with try-catch, and global error handlers for unhandled rejections.

Create custom error classes extending Error for domain-specific errors with additional context. Include error boundaries in React applications to catch rendering errors gracefully.

Centralized error logging services (Sentry, LogRocket) capture production errors with context. Always log enough information for debugging but sanitize sensitive data. Provide user-friendly error messages while logging detailed technical errors.

Defensive programming includes validating inputs, handling edge cases, failing fast with meaningful errors, and gracefully degrading functionality when possible rather than breaking entirely.

### 29. What is TypeScript and what benefits does it provide?

**Answer:** TypeScript is a typed superset of JavaScript that compiles to plain JavaScript, adding static type checking, interfaces, enums, and advanced language features. It catches errors at compile-time rather than runtime.

Benefits include earlier bug detection, better IDE support with autocomplete and refactoring, self-documenting code through types, easier refactoring in large codebases, and improved team collaboration through clear contracts.

Type inference reduces verbosity while maintaining safety. Gradual adoption allows migrating JavaScript projects incrementally. Modern frameworks like Angular require TypeScript, while React and Vue offer excellent TypeScript support.

Trade-offs include build step complexity, learning curve, and potential over-engineering with complex types. For large applications or teams, benefits typically outweigh costs significantly.

### 30. Explain code review best practices

**Answer:** Effective code reviews focus on correctness, design, maintainability, and learning. Review for logic errors, security vulnerabilities, performance issues, and adherence to coding standards. Check test coverage and edge case handling.

Provide constructive feedback focusing on code, not the person. Ask questions rather than making demands. Explain reasoning behind suggestions. Praise good practices and clever solutions.

Keep reviews small and focused (under 400 lines ideally) for better quality and faster turnaround. Use automated tools for formatting and linting so reviews focus on logic. Create checklists for common issues.

As reviewee, provide context in PR descriptions, respond professionally to feedback, and understand it's about improving code quality. Reviews are learning opportunities for both parties.

---

## Coding Challenges

### 31. Implement a debounce function from scratch

**Question:** Write a debounce function that delays invoking func until after wait milliseconds have elapsed since the last time the debounced function was invoked.

**Answer:**
```javascript
function debounce(func, wait) {
  let timeoutId;
  
  return function debounced(...args) {
    const context = this;
    
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      func.apply(context, args);
    }, wait);
  };
}

// Usage
const handleSearch = debounce((query) => {
  console.log('Searching for:', query);
}, 300);

handleSearch('hello'); // won't execute
handleSearch('hello world'); // executes after 300ms
```

**Advanced version with immediate execution option:**
```javascript
function debounce(func, wait, immediate = false) {
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
    }, wait);
    
    if (callNow) {
      func.apply(context, args);
    }
  };
}
```

### 32. Implement a throttle function from scratch

**Question:** Write a throttle function that ensures func is called at most once per specified time period.

**Answer:**
```javascript
function throttle(func, limit) {
  let inThrottle;
  
  return function throttled(...args) {
    const context = this;
    
    if (!inThrottle) {
      func.apply(context, args);
      inThrottle = true;
      
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

// Advanced version with trailing execution
function throttle(func, limit, options = {}) {
  let timeoutId;
  let lastRan;
  let lastFunc;
  
  return function throttled(...args) {
    const context = this;
    
    if (!lastRan) {
      func.apply(context, args);
      lastRan = Date.now();
    } else {
      clearTimeout(timeoutId);
      lastFunc = () => func.apply(context, args);
      
      timeoutId = setTimeout(() => {
        if (Date.now() - lastRan >= limit) {
          lastFunc();
          lastRan = Date.now();
        }
      }, limit - (Date.now() - lastRan));
    }
  };
}

// Usage
const handleScroll = throttle(() => {
  console.log('Scroll position:', window.scrollY);
}, 1000);

window.addEventListener('scroll', handleScroll);
```

### 33. Implement deep clone with circular reference handling

**Question:** Write a function that performs deep cloning of objects, including handling circular references.

**Answer:**
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
  
  // Handle Array
  if (Array.isArray(obj)) {
    const arrCopy = [];
    hash.set(obj, arrCopy);
    obj.forEach((item, index) => {
      arrCopy[index] = deepClone(item, hash);
    });
    return arrCopy;
  }
  
  // Handle Map
  if (obj instanceof Map) {
    const mapCopy = new Map();
    hash.set(obj, mapCopy);
    obj.forEach((value, key) => {
      mapCopy.set(key, deepClone(value, hash));
    });
    return mapCopy;
  }
  
  // Handle Set
  if (obj instanceof Set) {
    const setCopy = new Set();
    hash.set(obj, setCopy);
    obj.forEach(value => {
      setCopy.add(deepClone(value, hash));
    });
    return setCopy;
  }
  
  // Handle Object
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

// Test with circular reference
const obj = { a: 1, b: { c: 2 } };
obj.self = obj;
const cloned = deepClone(obj);
console.log(cloned.self === cloned); // true
```

### 34. Implement Promise.all from scratch

**Question:** Implement your own version of Promise.all that takes an array of promises and returns a single promise.

**Answer:**
```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError('Arguments must be an array'));
    }
    
    const results = [];
    let completedCount = 0;
    const totalPromises = promises.length;
    
    if (totalPromises === 0) {
      return resolve(results);
    }
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = value;
          completedCount++;
          
          if (completedCount === totalPromises) {
            resolve(results);
          }
        })
        .catch(reject);
    });
  });
}

// Usage
promiseAll([
  Promise.resolve(1),
  Promise.resolve(2),
  Promise.resolve(3)
])
.then(results => console.log(results)) // [1, 2, 3]
.catch(err => console.error(err));
```

### 35. Implement a memoization function

**Question:** Create a generic memoization function that caches function results based on arguments.

**Answer:**
```javascript
function memoize(fn) {
  const cache = new Map();
  
  return function memoized(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log('Returning cached result');
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// Advanced version with custom resolver
function memoize(fn, resolver) {
  const cache = new Map();
  
  return function memoized(...args) {
    const key = resolver ? resolver(...args) : JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// Usage example - Fibonacci
const fibonacci = memoize((n) => {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(50)); // Fast due to memoization
```

### 36. Implement a function to flatten nested arrays

**Question:** Write a function that flattens a nested array to a specified depth.

**Answer:**
```javascript
function flatten(arr, depth = 1) {
  if (depth === 0) return arr;
  
  return arr.reduce((acc, item) => {
    if (Array.isArray(item)) {
      acc.push(...flatten(item, depth - 1));
    } else {
      acc.push(item);
    }
    return acc;
  }, []);
}

// Infinite depth version
function flattenDeep(arr) {
  return arr.reduce((acc, item) => {
    return acc.concat(Array.isArray(item) ? flattenDeep(item) : item);
  }, []);
}

// Iterative version
function flattenIterative(arr, depth = Infinity) {
  const stack = arr.map(item => [item, depth]);
  const result = [];
  
  while (stack.length > 0) {
    const [item, d] = stack.pop();
    
    if (Array.isArray(item) && d > 0) {
      stack.push(...item.map(i => [i, d - 1]));
    } else {
      result.push(item);
    }
  }
  
  return result.reverse();
}

// Usage
console.log(flatten([1, [2, [3, [4]], 5]], 2)); // [1, 2, 3, [4], 5]
console.log(flattenDeep([1, [2, [3, [4]], 5]])); // [1, 2, 3, 4, 5]
```

### 37. Implement a custom bind function

**Question:** Implement your own version of Function.prototype.bind.

**Answer:**
```javascript
Function.prototype.myBind = function(context, ...boundArgs) {
  const originalFunc = this;
  
  return function boundFunction(...args) {
    return originalFunc.apply(context, [...boundArgs, ...args]);
  };
};

// Advanced version handling new operator
Function.prototype.myBind = function(context, ...boundArgs) {
  const originalFunc = this;
  
  function boundFunction(...args) {
    // Check if called with new operator
    if (this instanceof boundFunction) {
      return new originalFunc(...boundArgs, ...args);
    }
    return originalFunc.apply(context, [...boundArgs, ...args]);
  }
  
  // Maintain prototype chain
  boundFunction.prototype = Object.create(originalFunc.prototype);
  
  return boundFunction;
};

// Usage
const obj = { name: 'John' };
function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

const boundGreet = greet.myBind(obj, 'Hello');
console.log(boundGreet('!')); // "Hello, John!"
```

### 38. Implement a retry mechanism for async functions

**Question:** Create a function that retries an async operation with exponential backoff.

**Answer:**
```javascript
async function retry(fn, options = {}) {
  const {
    maxAttempts = 3,
    delay = 1000,
    backoff = 2,
    onRetry = null
  } = options;
  
  let lastError;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      
      if (attempt === maxAttempts) {
        throw lastError;
      }
      
      const waitTime = delay * Math.pow(backoff, attempt - 1);
      
      if (onRetry) {
        onRetry(attempt, waitTime, error);
      }
      
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
  }
  
  throw lastError;
}

// Usage
const fetchData = () => fetch('/api/data').then(res => res.json());

retry(fetchData, {
  maxAttempts: 5,
  delay: 1000,
  backoff: 2,
  onRetry: (attempt, delay, error) => {
    console.log(`Retry attempt ${attempt} after ${delay}ms. Error: ${error.message}`);
  }
})
.then(data => console.log('Success:', data))
.catch(error => console.error('Failed after retries:', error));
```

### 39. Implement a LRU (Least Recently Used) Cache

**Question:** Design and implement a Least Recently Used (LRU) cache with get and put operations in O(1) time complexity.

**Answer:**
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
    // If key exists, delete it first
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }
    
    // Add new entry
    this.cache.set(key, value);
    
    // If over capacity, remove least recently used (first item)
    if (this.cache.size > this.capacity) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
  }
  
  // Additional helper methods
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

// Usage
const cache = new LRUCache(3);
cache.put(1, 'one');
cache.put(2, 'two');
cache.put(3, 'three');
console.log(cache.get(1)); // 'one' (now most recent)
cache.put(4, 'four'); // Evicts key 2
console.log(cache.get(2)); // -1 (not found)
```

### 40. Implement a pub/sub (Event Emitter) system

**Question:** Create an event emitter class with subscribe, unsubscribe, and emit functionality.

**Answer:**
```javascript
class EventEmitter {
  constructor() {
    this.events = new Map();
  }
  
  on(event, callback) {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    
    this.events.get(event).push(callback);
    
    // Return unsubscribe function
    return () => this.off(event, callback);
  }
  
  once(event, callback) {
    const onceWrapper = (...args) => {
      callback(...args);
      this.off(event, onceWrapper);
    };
    
    return this.on(event, onceWrapper);
  }
  
  off(event, callback) {
    if (!this.events.has(event)) return;
    
    const callbacks = this.events.get(event);
    const index = callbacks.indexOf(callback);
    
    if (index !== -1) {
      callbacks.splice(index, 1);
    }
    
    // Clean up if no more listeners
    if (callbacks.length === 0) {
      this.events.delete(event);
    }
  }
  
  emit(event, ...args) {
    if (!this.events.has(event)) return;
    
    const callbacks = this.events.get(event);
    callbacks.forEach(callback => {
      try {
        callback(...args);
      } catch (error) {
        console.error(`Error in event handler for "${event}":`, error);
      }
    });
  }
  
  removeAllListeners(event) {
    if (event) {
      this.events.delete(event);
    } else {
      this.events.clear();
    }
  }
  
  listenerCount(event) {
    return this.events.has(event) ? this.events.get(event).length : 0;
  }
}

// Usage
const emitter = new EventEmitter();

const unsubscribe = emitter.on('data', (data) => {
  console.log('Received:', data);
});

emitter.emit('data', { message: 'Hello' }); // Received: { message: 'Hello' }

emitter.once('connect', () => {
  console.log('Connected!');
});

emitter.emit('connect'); // Connected!
emitter.emit('connect'); // (nothing - only fires once)

unsubscribe(); // Remove listener
```

### 41. Implement a function composition utility

**Question:** Create a compose/pipe function for functional programming composition.

**Answer:**
```javascript
// Compose - right to left execution
function compose(...fns) {
  return function composed(arg) {
    return fns.reduceRight((acc, fn) => fn(acc), arg);
  };
}

// Pipe - left to right execution
function pipe(...fns) {
  return function piped(arg) {
    return fns.reduce((acc, fn) => fn(acc), arg);
  };
}

// Async versions
function composeAsync(...fns) {
  return function composed(arg) {
    return fns.reduceRight(
      (acc, fn) => acc.then(fn),
      Promise.resolve(arg)
    );
  };
}

function pipeAsync(...fns) {
  return function piped(arg) {
    return fns.reduce(
      (acc, fn) => acc.then(fn),
      Promise.resolve(arg)
    );
  };
}

// Usage
const add5 = x => x + 5;
const multiply3 = x => x * 3;
const subtract2 = x => x - 2;

const calculate = compose(subtract2, multiply3, add5);
console.log(calculate(10)); // (10 + 5) * 3 - 2 = 43

const calculatePipe = pipe(add5, multiply3, subtract2);
console.log(calculatePipe(10)); // (10 + 5) * 3 - 2 = 43
```

### 42. Implement a debounced autocomplete search

**Question:** Create an autocomplete search component with debouncing and request cancellation.

**Answer:**
```javascript
class AutoComplete {
  constructor(options = {}) {
    this.delay = options.delay || 300;
    this.minChars = options.minChars || 2;
    this.onSearch = options.onSearch;
    this.onResults = options.onResults;
    this.onError = options.onError;
    
    this.timeoutId = null;
    this.abortController = null;
  }
  
  handleInput(query) {
    // Clear previous timeout
    clearTimeout(this.timeoutId);
    
    // Cancel previous request
    if (this.abortController) {
      this.abortController.abort();
    }
    
    // Check minimum characters
    if (query.length < this.minChars) {
      this.onResults([]);
      return;
    }
    
    // Debounce the search
    this.timeoutId = setTimeout(() => {
      this.search(query);
    }, this.delay);
  }
  
  async search(query) {
    try {
      // Create new abort controller for this request
      this.abortController = new AbortController();
      
      const results = await this.onSearch(query, {
        signal: this.abortController.signal
      });
      
      this.onResults(results);
    } catch (error) {
      if (error.name !== 'AbortError') {
        this.onError(error);
      }
    }
  }
  
  destroy() {
    clearTimeout(this.timeoutId);
    if (this.abortController) {
      this.abortController.abort();
    }
  }
}

// Usage
const autocomplete = new AutoComplete({
  delay: 300,
  minChars: 2,
  onSearch: async (query, options) => {
    const response = await fetch(
      `/api/search?q=${encodeURIComponent(query)}`,
      options
    );
    return response.json();
  },
  onResults: (results) => {
    console.log('Search results:', results);
  },
  onError: (error) => {
    console.error('Search error:', error);
  }
});

// In your input handler
document.querySelector('#search').addEventListener('input', (e) => {
  autocomplete.handleInput(e.target.value);
});
```

### 43. Implement a promisify utility

**Question:** Create a utility function that converts callback-based functions to promise-based functions.

**Answer:**
```javascript
function promisify(fn) {
  return function promisified(...args) {
    return new Promise((resolve, reject) => {
      fn(...args, (error, result) => {
        if (error) {
          reject(error);
        } else {
          resolve(result);
        }
      });
    });
  };
}

// Advanced version with custom callback position
function promisify(fn, options = {}) {
  return function promisified(...args) {
    return new Promise((resolve, reject) => {
      const callback = (error, ...results) => {
        if (error) {
          reject(error);
        } else {
          resolve(options.multiArgs ? results : results[0]);
        }
      };
      
      args.push(callback);
      fn.apply(this, args);
    });
  };
}

// Promisify all methods of an object
function promisifyAll(obj) {
  const promisified = {};
  
  Object.keys(obj).forEach(key => {
    if (typeof obj[key] === 'function') {
      promisified[key] = promisify(obj[key].bind(obj));
    }
  });
  
  return promisified;
}

// Usage
const fs = require('fs');

const readFileAsync = promisify(fs.readFile);

readFileAsync('file.txt', 'utf8')
  .then(content => console.log(content))
  .catch(error => console.error(error));
```

### 44. Implement a virtual DOM diff algorithm (simplified)

**Question:** Create a simplified virtual DOM diff function that compares two virtual DOM trees.

**Answer:**
```javascript
function diff(oldVNode, newVNode) {
  // Different types - replace
  if (oldVNode.type !== newVNode.type) {
    return {
      type: 'REPLACE',
      newVNode
    };
  }
  
  // Text nodes - update if different
  if (typeof oldVNode === 'string' || typeof newVNode === 'string') {
    if (oldVNode !== newVNode) {
      return {
        type: 'TEXT',
        newVNode
      };
    }
    return null;
  }
  
  // Compare props
  const propsPatches = diffProps(oldVNode.props, newVNode.props);
  
  // Compare children
  const childrenPatches = diffChildren(oldVNode.children, newVNode.children);
  
  if (propsPatches || childrenPatches.length > 0) {
    return {
      type: 'UPDATE',
      props: propsPatches,
      children: childrenPatches
    };
  }
  
  return null;
}

function diffProps(oldProps, newProps) {
  const patches = {};
  
  // Check for changed or removed props
  for (let key in oldProps) {
    if (oldProps[key] !== newProps[key]) {
      patches[key] = newProps[key];
    }
  }
  
  // Check for new props
  for (let key in newProps) {
    if (!(key in oldProps)) {
      patches[key] = newProps[key];
    }
  }
  
  return Object.keys(patches).length > 0 ? patches : null;
}

function diffChildren(oldChildren, newChildren) {
  const patches = [];
  const maxLength = Math.max(oldChildren.length, newChildren.length);
  
  for (let i = 0; i < maxLength; i++) {
    patches.push(diff(oldChildren[i], newChildren[i]));
  }
  
  return patches;
}

// Virtual node creator
function h(type, props, ...children) {
  return {
    type,
    props: props || {},
    children: children.flat()
  };
}

// Usage
const oldTree = h('div', { id: 'container' },
  h('h1', null, 'Hello'),
  h('p', null, 'World')
);

const newTree = h('div', { id: 'container', class: 'updated' },
  h('h1', null, 'Hello'),
  h('p', null, 'Updated World')
);

const patches = diff(oldTree, newTree);
console.log(patches);
```

### 45. Implement a rate limiter

**Question:** Create a rate limiter that limits function calls to a maximum number within a time window.

**Answer:**
```javascript
class RateLimiter {
  constructor(maxCalls, timeWindow) {
    this.maxCalls = maxCalls;
    this.timeWindow = timeWindow; // in milliseconds
    this.calls = [];
  }
  
  tryCall(fn, ...args) {
    const now = Date.now();
    
    // Remove calls outside the time window
    this.calls = this.calls.filter(
      timestamp => now - timestamp < this.timeWindow
    );
    
    // Check if we can make the call
    if (this.calls.length < this.maxCalls) {
      this.calls.push(now);
      return fn(...args);
    }
    
    // Calculate time until next available slot
    const oldestCall = this.calls[0];
    const timeUntilNextSlot = this.timeWindow - (now - oldestCall);
    
    throw new Error(
      `Rate limit exceeded. Try again in ${Math.ceil(timeUntilNextSlot / 1000)}s`
    );
  }
  
  async tryCallAsync(fn, ...args) {
    const now = Date.now();
    
    this.calls = this.calls.filter(
      timestamp => now - timestamp < this.timeWindow
    );
    
    if (this.calls.length < this.maxCalls) {
      this.calls.push(now);
      return await fn(...args);
    }
    
    const oldestCall = this.calls[0];
    const timeUntilNextSlot = this.timeWindow - (now - oldestCall);
    
    // Wait and retry
    await new Promise(resolve => setTimeout(resolve, timeUntilNextSlot));
    return this.tryCallAsync(fn, ...args);
  }
  
  reset() {
    this.calls = [];
  }
}

// Token bucket algorithm (more sophisticated)
class TokenBucket {
  constructor(capacity, refillRate) {
    this.capacity = capacity;
    this.tokens = capacity;
    this.refillRate = refillRate; // tokens per second
    this.lastRefill = Date.now();
  }
  
  refill() {
    const now = Date.now();
    const timePassed = (now - this.lastRefill) / 1000;
    const tokensToAdd = timePassed * this.refillRate;
    
    this.tokens = Math.min(this.capacity, this.tokens + tokensToAdd);
    this.lastRefill = now;
  }
  
  tryConsume(tokens = 1) {
    this.refill();
    
    if (this.tokens >= tokens) {
      this.tokens -= tokens;
      return true;
    }
    
    return false;
  }
  
  async consume(tokens = 1) {
    while (!this.tryConsume(tokens)) {
      await new Promise(resolve => setTimeout(resolve, 100));
    }
    return true;
  }
}

// Usage
const limiter = new RateLimiter(5, 60000); // 5 calls per minute

try {
  limiter.tryCall(() => console.log('API call 1'));
  limiter.tryCall(() => console.log('API call 2'));
  // ... up to 5 calls succeed
  limiter.tryCall(() => console.log('API call 6')); // throws error
} catch (error) {
  console.error(error.message);
}

// Token bucket usage
const bucket = new TokenBucket(10, 1); // 10 tokens, refill 1 per second

bucket.consume(3).then(() => {
  console.log('Consumed 3 tokens');
});
```

### 46. Implement a deep equality check function

**Question:** Write a function that performs deep equality comparison between two values, including nested objects, arrays, and special types.

**Answer:**
```javascript
function deepEqual(obj1, obj2) {
  // Same reference
  if (obj1 === obj2) return true;
  
  // Check for null/undefined
  if (obj1 == null || obj2 == null) return false;
  
  // Check if both are objects
  if (typeof obj1 !== 'object' || typeof obj2 !== 'object') {
    return Object.is(obj1, obj2); // Handles NaN, -0, +0
  }
  
  // Check for Date
  if (obj1 instanceof Date && obj2 instanceof Date) {
    return obj1.getTime() === obj2.getTime();
  }
  
  // Check for RegExp
  if (obj1 instanceof RegExp && obj2 instanceof RegExp) {
    return obj1.toString() === obj2.toString();
  }
  
  // Check for Array
  if (Array.isArray(obj1) && Array.isArray(obj2)) {
    if (obj1.length !== obj2.length) return false;
    return obj1.every((item, index) => deepEqual(item, obj2[index]));
  }
  
  // Check for Map
  if (obj1 instanceof Map && obj2 instanceof Map) {
    if (obj1.size !== obj2.size) return false;
    for (let [key, value] of obj1) {
      if (!obj2.has(key) || !deepEqual(value, obj2.get(key))) {
        return false;
      }
    }
    return true;
  }
  
  // Check for Set
  if (obj1 instanceof Set && obj2 instanceof Set) {
    if (obj1.size !== obj2.size) return false;
    for (let item of obj1) {
      if (!obj2.has(item)) return false;
    }
    return true;
  }
  
  // Compare plain objects
  const keys1 = Object.keys(obj1);
  const keys2 = Object.keys(obj2);
  
  if (keys1.length !== keys2.length) return false;
  
  return keys1.every(key => 
    keys2.includes(key) && deepEqual(obj1[key], obj2[key])
  );
}

// Usage
console.log(deepEqual({ a: 1, b: { c: 2 } }, { a: 1, b: { c: 2 } })); // true
console.log(deepEqual([1, 2, [3, 4]], [1, 2, [3, 4]])); // true
console.log(deepEqual(NaN, NaN)); // true
```

### 47. Implement a curry function

**Question:** Create a generic curry function that transforms a multi-argument function into a sequence of single-argument functions.

**Answer:**
```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    
    return function(...nextArgs) {
      return curried.apply(this, [...args, ...nextArgs]);
    };
  };
}

// Advanced version with placeholders
const _ = Symbol('placeholder');

function curry(fn, arity = fn.length) {
  return function curried(...args) {
    // Replace placeholders with actual values
    const processedArgs = args.slice(0, arity);
    
    // Count non-placeholder arguments
    const validArgs = processedArgs.filter(arg => arg !== _).length;
    
    if (validArgs >= arity) {
      return fn.apply(this, processedArgs);
    }
    
    return function(...nextArgs) {
      // Merge arguments, replacing placeholders
      const mergedArgs = processedArgs.map(arg => 
        arg === _ && nextArgs.length > 0 ? nextArgs.shift() : arg
      );
      
      return curried(...mergedArgs, ...nextArgs);
    };
  };
}

// Usage
const sum = (a, b, c) => a + b + c;
const curriedSum = curry(sum);

console.log(curriedSum(1)(2)(3)); // 6
console.log(curriedSum(1, 2)(3)); // 6
console.log(curriedSum(1)(2, 3)); // 6

// With placeholders
console.log(curriedSum(_, 2)(1, 3)); // 6
console.log(curriedSum(_, _, 3)(1)(2)); // 6
```

### 48. Implement an async queue with concurrency control

**Question:** Create a queue that processes async tasks with a maximum concurrency limit.

**Answer:**
```javascript
class AsyncQueue {
  constructor(concurrency = 1) {
    this.concurrency = concurrency;
    this.running = 0;
    this.queue = [];
  }
  
  async enqueue(task) {
    return new Promise((resolve, reject) => {
      this.queue.push({ task, resolve, reject });
      this.process();
    });
  }
  
  async process() {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }
    
    this.running++;
    const { task, resolve, reject } = this.queue.shift();
    
    try {
      const result = await task();
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.running--;
      this.process();
    }
  }
  
  async drain() {
    while (this.queue.length > 0 || this.running > 0) {
      await new Promise(resolve => setTimeout(resolve, 10));
    }
  }
  
  clear() {
    this.queue = [];
  }
  
  size() {
    return this.queue.length;
  }
}

// Advanced version with priority support
class PriorityAsyncQueue {
  constructor(concurrency = 1) {
    this.concurrency = concurrency;
    this.running = 0;
    this.queue = [];
  }
  
  async enqueue(task, priority = 0) {
    return new Promise((resolve, reject) => {
      const item = { task, resolve, reject, priority };
      
      // Insert by priority (higher priority first)
      const index = this.queue.findIndex(i => i.priority < priority);
      if (index === -1) {
        this.queue.push(item);
      } else {
        this.queue.splice(index, 0, item);
      }
      
      this.process();
    });
  }
  
  async process() {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }
    
    this.running++;
    const { task, resolve, reject } = this.queue.shift();
    
    try {
      const result = await task();
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.running--;
      this.process();
    }
  }
}

// Usage
const queue = new AsyncQueue(2); // Max 2 concurrent tasks

const tasks = [
  () => new Promise(resolve => setTimeout(() => resolve('Task 1'), 1000)),
  () => new Promise(resolve => setTimeout(() => resolve('Task 2'), 500)),
  () => new Promise(resolve => setTimeout(() => resolve('Task 3'), 300)),
  () => new Promise(resolve => setTimeout(() => resolve('Task 4'), 200)),
];

Promise.all(tasks.map(task => queue.enqueue(task)))
  .then(results => console.log('All tasks completed:', results));
```

### 49. Implement a object path getter/setter (like lodash get/set)

**Question:** Create utilities to safely get and set deeply nested object properties using string paths.

**Answer:**
```javascript
function get(obj, path, defaultValue = undefined) {
  // Convert string path to array
  const keys = Array.isArray(path) 
    ? path 
    : path.replace(/\[(\d+)\]/g, '.$1').split('.');
  
  let result = obj;
  
  for (let key of keys) {
    if (result == null) {
      return defaultValue;
    }
    result = result[key];
  }
  
  return result !== undefined ? result : defaultValue;
}

function set(obj, path, value) {
  const keys = Array.isArray(path)
    ? path
    : path.replace(/\[(\d+)\]/g, '.$1').split('.');
  
  const lastKey = keys.pop();
  
  let current = obj;
  
  for (let key of keys) {
    // Create intermediate objects/arrays if they don't exist
    if (current[key] == null) {
      const nextKey = keys[keys.indexOf(key) + 1];
      current[key] = /^\d+$/.test(nextKey) ? [] : {};
    }
    current = current[key];
  }
  
  current[lastKey] = value;
  return obj;
}

function has(obj, path) {
  const keys = Array.isArray(path)
    ? path
    : path.replace(/\[(\d+)\]/g, '.$1').split('.');
  
  let current = obj;
  
  for (let key of keys) {
    if (!Object.prototype.hasOwnProperty.call(current, key)) {
      return false;
    }
    current = current[key];
  }
  
  return true;
}

function unset(obj, path) {
  const keys = Array.isArray(path)
    ? path
    : path.replace(/\[(\d+)\]/g, '.$1').split('.');
  
  const lastKey = keys.pop();
  
  let current = obj;
  
  for (let key of keys) {
    if (current[key] == null) {
      return true;
    }
    current = current[key];
  }
  
  if (Array.isArray(current)) {
    current.splice(lastKey, 1);
  } else {
    delete current[lastKey];
  }
  
  return true;
}

// Usage
const data = {
  user: {
    name: 'John',
    addresses: [
      { city: 'New York', zip: '10001' },
      { city: 'Los Angeles', zip: '90001' }
    ]
  }
};

console.log(get(data, 'user.name')); // 'John'
console.log(get(data, 'user.addresses[0].city')); // 'New York'
console.log(get(data, 'user.age', 30)); // 30 (default value)

set(data, 'user.age', 25);
set(data, 'user.addresses[1].country', 'USA');
console.log(data.user.age); // 25

console.log(has(data, 'user.name')); // true
console.log(has(data, 'user.email')); // false

unset(data, 'user.addresses[0]');
```

### 50. Implement a scheduler with task dependencies

**Question:** Create a task scheduler that handles dependencies between tasks (like a build system).

**Answer:**
```javascript
class TaskScheduler {
  constructor() {
    this.tasks = new Map();
    this.results = new Map();
    this.running = new Set();
  }
  
  addTask(name, dependencies = [], fn) {
    this.tasks.set(name, {
      dependencies,
      fn,
      status: 'pending' // pending, running, completed, failed
    });
  }
  
  async run(taskName) {
    // Return cached result if already completed
    if (this.results.has(taskName)) {
      return this.results.get(taskName);
    }
    
    // Check for circular dependencies
    if (this.running.has(taskName)) {
      throw new Error(`Circular dependency detected: ${taskName}`);
    }
    
    const task = this.tasks.get(taskName);
    if (!task) {
      throw new Error(`Task not found: ${taskName}`);
    }
    
    if (task.status === 'failed') {
      throw new Error(`Task failed: ${taskName}`);
    }
    
    // Mark as running
    this.running.add(taskName);
    task.status = 'running';
    
    try {
      // Run dependencies first
      const depResults = await Promise.all(
        task.dependencies.map(dep => this.run(dep))
      );
      
      // Run the task with dependency results
      const result = await task.fn(...depResults);
      
      // Cache result
      this.results.set(taskName, result);
      task.status = 'completed';
      this.running.delete(taskName);
      
      return result;
    } catch (error) {
      task.status = 'failed';
      this.running.delete(taskName);
      throw error;
    }
  }
  
  async runAll() {
    const taskNames = Array.from(this.tasks.keys());
    const results = {};
    
    for (let taskName of taskNames) {
      try {
        results[taskName] = await this.run(taskName);
      } catch (error) {
        console.error(`Task ${taskName} failed:`, error);
      }
    }
    
    return results;
  }
  
  reset() {
    this.results.clear();
    this.running.clear();
    this.tasks.forEach(task => {
      task.status = 'pending';
    });
  }
}

// Usage
const scheduler = new TaskScheduler();

scheduler.addTask('fetchUser', [], async () => {
  console.log('Fetching user...');
  await new Promise(resolve => setTimeout(resolve, 1000));
  return { id: 1, name: 'John' };
});

scheduler.addTask('fetchPosts', ['fetchUser'], async (user) => {
  console.log('Fetching posts for user:', user.name);
  await new Promise(resolve => setTimeout(resolve, 500));
  return [{ id: 1, title: 'Post 1' }, { id: 2, title: 'Post 2' }];
});

scheduler.addTask('fetchComments', ['fetchPosts'], async (posts) => {
  console.log('Fetching comments for posts');
  await new Promise(resolve => setTimeout(resolve, 500));
  return { comments: ['Great!', 'Nice post'] };
});

scheduler.addTask('aggregate', ['fetchUser', 'fetchPosts', 'fetchComments'], 
  async (user, posts, comments) => {
    return { user, posts, comments };
  }
);

scheduler.run('aggregate')
  .then(result => console.log('Final result:', result))
  .catch(error => console.error('Error:', error));
```

### 51. Implement a state machine

**Question:** Create a finite state machine implementation with transition validation.

**Answer:**
```javascript
class StateMachine {
  constructor(config) {
    this.states = config.states;
    this.initialState = config.initialState;
    this.currentState = config.initialState;
    this.transitions = config.transitions || {};
    this.listeners = new Map();
  }
  
  can(action) {
    const allowedTransitions = this.transitions[this.currentState] || {};
    return action in allowedTransitions;
  }
  
  transition(action, data = {}) {
    if (!this.can(action)) {
      throw new Error(
        `Invalid transition: ${action} from state ${this.currentState}`
      );
    }
    
    const prevState = this.currentState;
    const nextState = this.transitions[this.currentState][action];
    
    // Call onExit hook for previous state
    const prevStateConfig = this.states[prevState];
    if (prevStateConfig?.onExit) {
      prevStateConfig.onExit(data);
    }
    
    // Update state
    this.currentState = nextState;
    
    // Call onEnter hook for new state
    const nextStateConfig = this.states[nextState];
    if (nextStateConfig?.onEnter) {
      nextStateConfig.onEnter(data);
    }
    
    // Emit transition event
    this.emit('transition', {
      from: prevState,
      to: nextState,
      action,
      data
    });
    
    return this.currentState;
  }
  
  getState() {
    return this.currentState;
  }
  
  reset() {
    this.currentState = this.initialState;
  }
  
  on(event, callback) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event).push(callback);
  }
  
  emit(event, data) {
    const callbacks = this.listeners.get(event) || [];
    callbacks.forEach(callback => callback(data));
  }
}

// Usage - Traffic Light
const trafficLight = new StateMachine({
  initialState: 'red',
  states: {
    red: {
      onEnter: () => console.log('🔴 STOP'),
      onExit: () => console.log('Red light ending...')
    },
    yellow: {
      onEnter: () => console.log('🟡 PREPARE'),
      onExit: () => console.log('Yellow light ending...')
    },
    green: {
      onEnter: () => console.log('🟢 GO'),
      onExit: () => console.log('Green light ending...')
    }
  },
  transitions: {
    red: { next: 'green' },
    green: { next: 'yellow' },
    yellow: { next: 'red' }
  }
});

trafficLight.on('transition', ({ from, to }) => {
  console.log(`Transitioned from ${from} to ${to}`);
});

trafficLight.transition('next'); // red -> green
trafficLight.transition('next'); // green -> yellow
trafficLight.transition('next'); // yellow -> red

// Usage - Order Processing
const orderMachine = new StateMachine({
  initialState: 'pending',
  states: {
    pending: {},
    processing: {},
    shipped: {},
    delivered: {},
    cancelled: {}
  },
  transitions: {
    pending: { 
      process: 'processing',
      cancel: 'cancelled'
    },
    processing: { 
      ship: 'shipped',
      cancel: 'cancelled'
    },
    shipped: { 
      deliver: 'delivered'
    }
  }
});
```

### 52. Implement a lazy evaluation iterator

**Question:** Create a lazy iterator that generates values on-demand with chainable operations.

**Answer:**
```javascript
class LazyIterator {
  constructor(iterable) {
    this.iterable = iterable;
    this.operations = [];
  }
  
  map(fn) {
    this.operations.push({
      type: 'map',
      fn
    });
    return this;
  }
  
  filter(fn) {
    this.operations.push({
      type: 'filter',
      fn
    });
    return this;
  }
  
  take(n) {
    this.operations.push({
      type: 'take',
      count: n
    });
    return this;
  }
  
  skip(n) {
    this.operations.push({
      type: 'skip',
      count: n
    });
    return this;
  }
  
  *[Symbol.iterator]() {
    let taken = 0;
    let skipped = 0;
    
    for (let value of this.iterable) {
      let currentValue = value;
      let shouldYield = true;
      
      for (let operation of this.operations) {
        if (!shouldYield) break;
        
        switch (operation.type) {
          case 'map':
            currentValue = operation.fn(currentValue);
            break;
            
          case 'filter':
            if (!operation.fn(currentValue)) {
              shouldYield = false;
            }
            break;
            
          case 'skip':
            if (skipped < operation.count) {
              skipped++;
              shouldYield = false;
            }
            break;
            
          case 'take':
            if (taken >= operation.count) {
              return;
            }
            break;
        }
      }
      
      if (shouldYield) {
        taken++;
        yield currentValue;
        
        // Check if we should stop after yielding
        for (let operation of this.operations) {
          if (operation.type === 'take' && taken >= operation.count) {
            return;
          }
        }
      }
    }
  }
  
  toArray() {
    return Array.from(this);
  }
  
  reduce(fn, initialValue) {
    let accumulator = initialValue;
    let first = true;
    
    for (let value of this) {
      if (first && accumulator === undefined) {
        accumulator = value;
        first = false;
      } else {
        accumulator = fn(accumulator, value);
      }
    }
    
    return accumulator;
  }
  
  forEach(fn) {
    for (let value of this) {
      fn(value);
    }
  }
  
  find(fn) {
    for (let value of this) {
      if (fn(value)) {
        return value;
      }
    }
    return undefined;
  }
  
  some(fn) {
    for (let value of this) {
      if (fn(value)) {
        return true;
      }
    }
    return false;
  }
  
  every(fn) {
    for (let value of this) {
      if (!fn(value)) {
        return false;
      }
    }
    return true;
  }
}

// Helper function
function lazy(iterable) {
  return new LazyIterator(iterable);
}

// Infinite generator
function* fibonacci() {
  let a = 0, b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

// Usage
const result = lazy(fibonacci())
  .filter(n => n % 2 === 0)  // Only even numbers
  .map(n => n * 2)            // Double them
  .skip(2)                    // Skip first 2
  .take(5)                    // Take next 5
  .toArray();

console.log(result); // [8, 68, 272, 1096, 4392]

// Large array processing without loading all into memory
const numbers = lazy(Array.from({ length: 1000000 }, (_, i) => i))
  .filter(n => n % 2 === 0)
  .map(n => n * n)
  .take(10)
  .toArray();
```

### 53. Implement a blockchain (simplified)

**Question:** Create a basic blockchain implementation with block creation and validation.

**Answer:**
```javascript
const crypto = require('crypto');

class Block {
  constructor(index, timestamp, data, previousHash = '') {
    this.index = index;
    this.timestamp = timestamp;
    this.data = data;
    this.previousHash = previousHash;
    this.nonce = 0;
    this.hash = this.calculateHash();
  }
  
  calculateHash() {
    return crypto
      .createHash('sha256')
      .update(
        this.index +
        this.previousHash +
        this.timestamp +
        JSON.stringify(this.data) +
        this.nonce
      )
      .digest('hex');
  }
  
  mineBlock(difficulty) {
    const target = Array(difficulty + 1).join('0');
    
    while (this.hash.substring(0, difficulty) !== target) {
      this.nonce++;
      this.hash = this.calculateHash();
    }
    
    console.log(`Block mined: ${this.hash}`);
  }
}

class Blockchain {
  constructor() {
    this.chain = [this.createGenesisBlock()];
    this.difficulty = 2;
    this.pendingTransactions = [];
    this.miningReward = 100;
  }
  
  createGenesisBlock() {
    return new Block(0, Date.now(), 'Genesis Block', '0');
  }
  
  getLatestBlock() {
    return this.chain[this.chain.length - 1];
  }
  
  minePendingTransactions(miningRewardAddress) {
    const block = new Block(
      this.chain.length,
      Date.now(),
      this.pendingTransactions,
      this.getLatestBlock().hash
    );
    
    block.mineBlock(this.difficulty);
    
    this.chain.push(block);
    
    // Reset pending transactions and add mining reward
    this.pendingTransactions = [
      {
        from: null,
        to: miningRewardAddress,
        amount: this.miningReward
      }
    ];
  }
  
  addTransaction(transaction) {
    if (!transaction.from || !transaction.to) {
      throw new Error('Transaction must include from and to address');
    }
    
    if (transaction.amount <= 0) {
      throw new Error('Transaction amount must be positive');
    }
    
    this.pendingTransactions.push(transaction);
  }
  
  getBalanceOfAddress(address) {
    let balance = 0;
    
    for (const block of this.chain) {
      if (Array.isArray(block.data)) {
        for (const transaction of block.data) {
          if (transaction.from === address) {
            balance -= transaction.amount;
          }
          if (transaction.to === address) {
            balance += transaction.amount;
          }
        }
      }
    }
    
    return balance;
  }
  
  isChainValid() {
    for (let i = 1; i < this.chain.length; i++) {
      const currentBlock = this.chain[i];
      const previousBlock = this.chain[i - 1];
      
      // Verify hash
      if (currentBlock.hash !== currentBlock.calculateHash()) {
        return false;
      }
      
      // Verify link to previous block
      if (currentBlock.previousHash !== previousBlock.hash) {
        return false;
      }
      
      // Verify proof of work
      const target = Array(this.difficulty + 1).join('0');
      if (currentBlock.hash.substring(0, this.difficulty) !== target) {
        return false;
      }
    }
    
    return true;
  }
}

// Usage
const myBlockchain = new Blockchain();

myBlockchain.addTransaction({ from: 'address1', to: 'address2', amount: 100 });
myBlockchain.addTransaction({ from: 'address2', to: 'address1', amount: 50 });

console.log('Starting mining...');
myBlockchain.minePendingTransactions('miner-address');

console.log('Balance of miner:', myBlockchain.getBalanceOfAddress('miner-address'));

myBlockchain.addTransaction({ from: 'address1', to: 'address2', amount: 25 });
myBlockchain.minePendingTransactions('miner-address');

console.log('Balance of miner:', myBlockchain.getBalanceOfAddress('miner-address'));
console.log('Chain valid?', myBlockchain.isChainValid());

// Try to tamper with the blockchain
myBlockchain.chain[1].data = [{ from: 'hacker', to: 'hacker', amount: 1000 }];
myBlockchain.chain[1].hash = myBlockchain.chain[1].calculateHash();

console.log('Chain valid after tampering?', myBlockchain.isChainValid());
```

### 54. Implement an observable pattern with RxJS-like operators

**Question:** Create an Observable implementation with common operators like map, filter, and subscribe.

**Answer:**
```javascript
class Observable {
  constructor(subscribe) {
    this._subscribe = subscribe;
  }
  
  subscribe(observer) {
    const safeObserver = {
      next: observer.next?.bind(observer) || (() => {}),
      error: observer.error?.bind(observer) || ((err) => { throw err; }),
      complete: observer.complete?.bind(observer) || (() => {})
    };
    
    try {
      const unsubscribe = this._subscribe(safeObserver);
      return {
        unsubscribe: typeof unsubscribe === 'function' 
          ? unsubscribe 
          : () => {}
      };
    } catch (error) {
      safeObserver.error(error);
      return { unsubscribe: () => {} };
    }
  }
  
  pipe(...operators) {
    return operators.reduce((source, operator) => operator(source), this);
  }
  
  // Static creation methods
  static of(...values) {
    return new Observable(observer => {
      values.forEach(value => observer.next(value));
      observer.complete();
    });
  }
  
  static from(iterable) {
    return new Observable(observer => {
      try {
        for (let value of iterable) {
          observer.next(value);
        }
        observer.complete();
      } catch (error) {
        observer.error(error);
      }
    });
  }
  
  static interval(period) {
    return new Observable(observer => {
      let count = 0;
      const id = setInterval(() => {
        observer.next(count++);
      }, period);
      
      return () => clearInterval(id);
    });
  }
  
  static fromEvent(element, eventName) {
    return new Observable(observer => {
      const handler = (event) => observer.next(event);
      element.addEventListener(eventName, handler);
      
      return () => element.removeEventListener(eventName, handler);
    });
  }
}

// Operators
function map(transformFn) {
  return (source) => {
    return new Observable(observer => {
      return source.subscribe({
        next: (value) => {
          try {
            observer.next(transformFn(value));
          } catch (error) {
            observer.error(error);
          }
        },
        error: (err) => observer.error(err),
        complete: () => observer.complete()
      });
    });
  };
}

function filter(predicateFn) {
  return (source) => {
    return new Observable(observer => {
      return source.subscribe({
        next: (value) => {
          try {
            if (predicateFn(value)) {
              observer.next(value);
            }
          } catch (error) {
            observer.error(error);
          }
        },
        error: (err) => observer.error(err),
        complete: () => observer.complete()
      });
    });
  };
}

function take(count) {
  return (source) => {
    return new Observable(observer => {
      let taken = 0;
      const subscription = source.subscribe({
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
      
      return () => subscription.un