# JavaScript Interview Questions - Senior Level (15+ Years Experience)

## Core JavaScript & Language Internals

### 1. Event Loop & Asynchronous Programming
- Explain the JavaScript event loop in detail, including the call stack, task queue, and microtask queue. How do promises differ from setTimeout in terms of execution order?
- What are the differences between `async/await`, Promises, and callbacks? When would you choose one over the other?
- How would you implement your own Promise from scratch?
- Explain the concept of "back pressure" in stream processing and how you'd handle it in Node.js

### 2. Memory Management & Performance
- Describe how garbage collection works in V8. What are the different generations of objects?
- What are memory leaks in JavaScript? Provide examples of common causes and how to detect/prevent them.
- Explain the difference between stack and heap memory allocation in JavaScript.
- How would you profile and optimize a JavaScript application experiencing performance issues?

### 3. Closures & Scope
- Explain closures with practical use cases. What are the performance implications?
- What is the Temporal Dead Zone (TDZ)? How does it relate to `let` and `const`?
- Describe the difference between lexical scoping and dynamic scoping.
- How do you handle memory leaks caused by closures in long-running applications?

### 4. Prototypal Inheritance
- Explain prototypal inheritance vs classical inheritance. What are the trade-offs?
- How does the prototype chain work? What happens when you access a property?
- What's the difference between `__proto__` and `prototype`?
- How would you implement inheritance without using ES6 classes?

## Advanced Patterns & Architecture

### 5. Design Patterns
- Implement the Module pattern, Revealing Module pattern, and Singleton pattern. When would you use each?
- Explain the Observer pattern and how it's used in modern frameworks (e.g., React's state management).
- Describe the Proxy pattern and provide real-world use cases.
- What is the Mediator pattern and how does it help with complex UIs?

### 6. Functional Programming
- Explain pure functions, immutability, and side effects. Why are they important?
- What are higher-order functions? Provide advanced examples beyond map/filter/reduce.
- Describe function composition and currying with practical applications.
- What is a monad? Can you implement a Maybe/Option monad in JavaScript?

### 7. Object-Oriented Programming
- Compare different ways to create objects in JavaScript (constructor functions, classes, factory functions, Object.create).
- Explain the SOLID principles and how they apply to JavaScript.
- What are mixins and how do they differ from inheritance?
- How would you implement private fields before the `#` syntax was introduced?

## Modern JavaScript & ES6+

### 8. ES6+ Features
- Explain the differences between `var`, `let`, and `const` in terms of scope, hoisting, and the temporal dead zone.
- What are Symbols and WeakMaps? Provide practical use cases for each.
- Describe Proxies and Reflect API. How would you use them for data validation?
- Explain generators and iterators. How do they enable lazy evaluation?

### 9. Modules & Build Systems
- Compare CommonJS, AMD, UMD, and ES6 modules. What are the pros and cons of each?
- How does tree-shaking work? What considerations are needed to make code tree-shakeable?
- Explain code splitting strategies in large applications.
- Describe the difference between static and dynamic imports.

### 10. Advanced Functions
- What is the difference between `.call()`, `.apply()`, and `.bind()`?
- Explain function hoisting vs variable hoisting.
- What are arrow functions and how do they differ from regular functions (beyond syntax)?
- Describe tail call optimization. Is it supported in JavaScript?

## Browser APIs & Web Standards

### 11. DOM & Rendering
- Explain the browser rendering pipeline (parsing, layout, paint, composite).
- What is the Virtual DOM? How does it improve performance?
- Describe the difference between reflow and repaint. How can you minimize them?
- What is the Critical Rendering Path and how do you optimize it?

### 12. Web APIs
- Explain Service Workers and their use cases. How do they differ from Web Workers?
- What are Web Workers and when would you use them?
- Describe the Fetch API vs XMLHttpRequest. What are the advantages of Fetch?
- How do IndexedDB and localStorage differ? When would you use each?

### 13. Security
- What is XSS (Cross-Site Scripting)? How do you prevent it?
- Explain CSRF (Cross-Site Request Forgery) and mitigation strategies.
- What is Content Security Policy (CSP) and how do you implement it?
- Describe the Same-Origin Policy and CORS in detail.

## Node.js & Backend

### 14. Node.js Fundamentals
- Explain the Node.js event loop. How does it differ from browser JavaScript?
- What are streams in Node.js? Describe readable, writable, duplex, and transform streams.
- How does Node.js handle child processes? When would you use cluster vs worker_threads?
- Explain the difference between process.nextTick() and setImmediate().

### 15. Backend Architecture
- How would you design a scalable microservices architecture using Node.js?
- Explain different strategies for handling authentication (JWT, sessions, OAuth).
- Describe caching strategies (Redis, in-memory, CDN) and when to use each.
- How do you handle errors in async/await code at scale?

## Testing & Quality

### 16. Testing Strategies
- Compare unit testing, integration testing, and end-to-end testing. What tools would you use for each?
- How do you test asynchronous code effectively?
- Explain test-driven development (TDD) vs behavior-driven development (BDD).
- What are code coverage metrics? What's a good target and why?

### 17. Debugging & Profiling
- Describe your approach to debugging a production issue in a large application.
- How do you use Chrome DevTools for performance profiling?
- What tools and techniques do you use for memory leak detection?
- Explain source maps and their importance in debugging minified code.

## Framework & Library Knowledge

### 18. React/Vue/Angular (Choose based on candidate)
- Explain how React's reconciliation algorithm works.
- What are the trade-offs between state management solutions (Redux, MobX, Context API)?
- Describe React Hooks lifecycle and rules. How do they compare to class components?
- What is the Virtual DOM diffing algorithm?

### 19. Build Tools & Bundlers
- Compare Webpack, Rollup, Vite, and esbuild. When would you choose each?
- How do you optimize bundle size in a production application?
- Explain lazy loading and code splitting strategies.
- What is Hot Module Replacement (HMR) and how does it work?

## System Design & Scalability

### 20. Frontend Architecture
- How would you architect a large-scale single-page application?
- Describe strategies for state management in complex applications.
- How do you handle code organization in a monorepo vs multiple repositories?
- Explain microfrontends architecture and when you'd use it.

### 21. Performance Optimization
- What metrics do you use to measure frontend performance (FCP, LCP, TTI, CLS)?
- Describe strategies for optimizing Time to Interactive (TTI).
- How do you implement progressive web app (PWA) features?
- Explain strategies for optimizing images and media assets.

### 22. Real-time Applications
- How would you implement real-time features (WebSockets vs Server-Sent Events)?
- Describe strategies for handling offline-first applications.
- How do you implement optimistic UI updates?
- What are the challenges of building a collaborative real-time editor?

## Advanced Topics

### 23. TypeScript
- What are the benefits of TypeScript over JavaScript?
- Explain generics, conditional types, and mapped types.
- How do you handle type safety with third-party libraries?
- What are utility types and when do you create custom ones?

### 24. WebAssembly
- What is WebAssembly and when would you use it?
- How does JavaScript interop with WebAssembly?
- Describe use cases where WebAssembly provides significant benefits.

### 25. Emerging Technologies
- Explain your experience with server-side rendering (SSR) vs static site generation (SSG).
- What are edge functions and how do they differ from traditional serverless?
- Describe your approach to implementing Progressive Enhancement.
- How do you stay current with JavaScript ecosystem changes?

## Behavioral & Leadership

### 26. Technical Leadership
- Describe a time you had to make a critical architectural decision. What was your approach?
- How do you mentor junior developers and establish coding standards?
- Explain your code review process and what you look for.
- How do you balance technical debt with feature development?

### 27. Problem-Solving
- Walk through your process for debugging a complex production issue.
- Describe a challenging performance problem you solved and your approach.
- How do you evaluate and adopt new technologies or frameworks?
- Explain a time you had to refactor a legacy codebase. What was your strategy?

---

## Coding Challenges (To Be Done Live)

1. Implement a deep clone function that handles circular references
2. Create a debounce and throttle function from scratch
3. Implement a Promise.all() function
4. Build an event emitter with on, off, and emit methods
5. Create a function that flattens a nested array to any depth
6. Implement a LRU (Least Recently Used) cache
7. Build a simple Observable pattern implementation
8. Create a retry mechanism for failed API calls with exponential backoff
9. Implement a function to detect memory leaks in a given code snippet
10. Build a simple state management library similar to Redux

---

*Note: These questions are designed to assess deep understanding, practical experience, and problem-solving abilities expected from a senior JavaScript engineer with 15+ years of experience.*