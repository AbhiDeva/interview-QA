# 🔥 Hard JavaScript Developer Interview Questions

## 📚 Table of Contents
1. [Core JavaScript Concepts](#core-concepts)
2. [Closures & Scope](#closures)
3. [Prototypes & Inheritance](#prototypes)
4. [Async JavaScript](#async)
5. [Event Loop & Concurrency](#event-loop)
6. [ES6+ Features](#es6)
7. [Functional Programming](#functional)
8. [Memory Management & Performance](#performance)
9. [Design Patterns](#patterns)
10. [Security](#security)
11. [Advanced Concepts](#advanced)
12. [Real-World Scenarios](#scenarios)

---

## <a id="core-concepts"></a>⚡ Core JavaScript Concepts

### Q1: Explain `this` keyword in JavaScript. What are the different ways to bind `this`?

**Expected Answer:**
```javascript
// 1. Global context
console.log(this); // window (browser) or global (Node.js)

// 2. Object method
const obj = {
  name: 'John',
  greet() {
    console.log(this.name); // 'John'
  }
};

// 3. Constructor function
function Person(name) {
  this.name = name; // 'this' refers to new instance
}
const person = new Person('Jane');

// 4. Arrow functions (lexical this)
const obj2 = {
  name: 'Bob',
  greet: () => {
    console.log(this.name); // undefined - arrow function doesn't bind this
  }
};

// 5. Explicit binding
function greet() {
  console.log(this.name);
}
greet.call({ name: 'Alice' });    // 'Alice'
greet.apply({ name: 'Bob' });     // 'Bob'
const boundGreet = greet.bind({ name: 'Charlie' });
boundGreet();                      // 'Charlie'

// 6. Event handlers
button.addEventListener('click', function() {
  console.log(this); // button element
});

button.addEventListener('click', () => {
  console.log(this); // lexical scope, not button
});
```

### Q2: What's the difference between `call`, `apply`, and `bind`?

**Expected Answer:**
```javascript
function introduce(greeting, punctuation) {
  return `${greeting}, I'm ${this.name}${punctuation}`;
}

const person = { name: 'John' };

// call - arguments passed individually
console.log(introduce.call(person, 'Hello', '!')); 
// "Hello, I'm John!"

// apply - arguments passed as array
console.log(introduce.apply(person, ['Hi', '.'])); 
// "Hi, I'm John."

// bind - returns new function with bound this
const boundIntroduce = introduce.bind(person, 'Hey');
console.log(boundIntroduce('?')); 
// "Hey, I'm John?"

// Practical use case: borrowing methods
const numbers = [1, 2, 3, 4, 5];
const max = Math.max.apply(null, numbers); // 5
// Modern alternative: Math.max(...numbers)
```

### Q3: Explain hoisting with examples. What gets hoisted and what doesn't?

**Expected Answer:**
```javascript
// Variable hoisting
console.log(x); // undefined (declaration hoisted, not initialization)
var x = 5;

console.log(y); // ReferenceError (temporal dead zone)
let y = 10;

console.log(z); // ReferenceError (temporal dead zone)
const z = 15;

// Function hoisting
greet(); // "Hello" - function declarations are fully hoisted
function greet() {
  console.log("Hello");
}

sayHi(); // TypeError: sayHi is not a function
var sayHi = function() {
  console.log("Hi");
};

// Class hoisting
const p = new Person(); // ReferenceError - classes aren't hoisted
class Person {}

// What actually happens (conceptually):
// var x;
// function greet() { console.log("Hello"); }
// var sayHi;
// console.log(x);
// x = 5;
// greet();
// sayHi();
// sayHi = function() { ... }
```

### Q4: What are the differences between `==` and `===`? Explain type coercion.

**Expected Answer:**
```javascript
// === (strict equality) - no type coercion
5 === 5;           // true
5 === '5';         // false
true === 1;        // false
null === undefined; // false

// == (loose equality) - performs type coercion
5 == '5';          // true (string converted to number)
true == 1;         // true (boolean converted to number)
false == 0;        // true
null == undefined; // true (special case)
'' == 0;           // true (empty string converted to 0)

// Tricky coercion examples
console.log([] == ![]);  // true (![] becomes false, then [] == false)
console.log([] == 0);    // true
console.log([''] == ''); // true
console.log([0] == 0);   // true
console.log([null] == ''); // true

// Why to always use ===
const value = 0;
if (value == false) {  // true - unexpected!
  console.log('executed');
}

if (value === false) { // false - expected
  console.log('not executed');
}

// Coercion rules
String(123);           // "123"
Number("456");         // 456
Number("abc");         // NaN
Boolean(0);            // false
Boolean("");           // false
Boolean([]);           // true
Boolean({});           // true
```

### Q5: Explain the difference between `null` and `undefined`.

**Expected Answer:**
```javascript
// undefined - variable declared but not assigned
let x;
console.log(x); // undefined
console.log(typeof x); // "undefined"

// null - intentional absence of value
let y = null;
console.log(y); // null
console.log(typeof y); // "object" (historical bug in JS)

// Differences
null == undefined;  // true
null === undefined; // false

// Use cases
function findUser(id) {
  // Return null when user not found (intentional)
  return users.find(u => u.id === id) || null;
}

// Function parameter not provided = undefined
function greet(name) {
  console.log(name); // undefined if not provided
}

// Object property doesn't exist = undefined
const obj = {};
console.log(obj.nonExistent); // undefined

// Explicitly setting to null
let data = { name: 'John' };
data = null; // Clear reference for garbage collection
```

---

## <a id="closures"></a>🔒 Closures & Scope

### Q6: What is a closure? Provide real-world examples.

**Expected Answer:**
```javascript
// Basic closure
function outer() {
  const message = 'Hello';
  
  function inner() {
    console.log(message); // inner has access to outer's scope
  }
  
  return inner;
}

const myFunc = outer();
myFunc(); // "Hello" - closure preserves outer scope

// Real-world example 1: Private variables
function createCounter() {
  let count = 0; // private variable
  
  return {
    increment() {
      count++;
      return count;
    },
    decrement() {
      count--;
      return count;
    },
    getCount() {
      return count;
    }
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.count);        // undefined (private)

// Real-world example 2: Function factory
function createMultiplier(multiplier) {
  return function(number) {
    return number * multiplier;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15

// Real-world example 3: Event handlers with private state
function setupButton(buttonId) {
  let clickCount = 0;
  
  document.getElementById(buttonId).addEventListener('click', function() {
    clickCount++;
    console.log(`Clicked ${clickCount} times`);
  });
}

// Real-world example 4: Memoization
function memoize(fn) {
  const cache = {};
  
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache[key]) {
      return cache[key];
    }
    const result = fn.apply(this, args);
    cache[key] = result;
    return result;
  };
}

const expensiveFunction = memoize((n) => {
  console.log('Computing...');
  return n * 2;
});

console.log(expensiveFunction(5)); // Computing... 10
console.log(expensiveFunction(5)); // 10 (cached)
```

### Q7: Explain the classic closure loop problem and its solutions.

**Expected Answer:**
```javascript
// Problem: All functions reference the same 'i'
for (var i = 0; i < 5; i++) {
  setTimeout(function() {
    console.log(i); // 5, 5, 5, 5, 5 (not 0, 1, 2, 3, 4)
  }, 1000);
}

// Solution 1: IIFE (Immediately Invoked Function Expression)
for (var i = 0; i < 5; i++) {
  (function(j) {
    setTimeout(function() {
      console.log(j); // 0, 1, 2, 3, 4
    }, 1000);
  })(i);
}

// Solution 2: let (block scope)
for (let i = 0; i < 5; i++) {
  setTimeout(function() {
    console.log(i); // 0, 1, 2, 3, 4
  }, 1000);
}

// Solution 3: bind
for (var i = 0; i < 5; i++) {
  setTimeout(function(j) {
    console.log(j); // 0, 1, 2, 3, 4
  }.bind(null, i), 1000);
}

// Solution 4: Arrow function with let
for (let i = 0; i < 5; i++) {
  setTimeout(() => console.log(i), 1000); // 0, 1, 2, 3, 4
}

// Why var fails:
// var is function-scoped, so there's only ONE 'i' variable
// By the time setTimeout callbacks execute, loop is done and i = 5

// Why let works:
// let is block-scoped, so each iteration creates a NEW 'i'
// Each setTimeout callback closes over its own 'i'
```

### Q8: What is lexical scoping?

**Expected Answer:**
```javascript
// Lexical scoping: inner functions have access to outer scope
// Determined at write-time, not runtime

const globalVar = 'global';

function outer() {
  const outerVar = 'outer';
  
  function middle() {
    const middleVar = 'middle';
    
    function inner() {
      const innerVar = 'inner';
      
      // Can access all outer scopes
      console.log(innerVar);   // 'inner'
      console.log(middleVar);  // 'middle'
      console.log(outerVar);   // 'outer'
      console.log(globalVar);  // 'global'
    }
    
    inner();
    // console.log(innerVar); // ReferenceError - can't access inner scope
  }
  
  middle();
}

outer();

// Lexical vs Dynamic scoping
function foo() {
  console.log(a); // Lexical: looks in surrounding code
}

function bar() {
  const a = 3;
  foo(); // In dynamic scoping, this would be 3
}

const a = 2;
bar(); // 2 (lexical scoping)

// Arrow functions and lexical this
const obj = {
  name: 'Object',
  regularFunc: function() {
    console.log(this.name); // 'Object'
    
    setTimeout(function() {
      console.log(this.name); // undefined (new 'this' context)
    }, 100);
    
    setTimeout(() => {
      console.log(this.name); // 'Object' (lexical 'this')
    }, 100);
  }
};
```

---

## <a id="prototypes"></a>🧬 Prototypes & Inheritance

### Q9: Explain prototypal inheritance. How does the prototype chain work?

**Expected Answer:**
```javascript
// Every object has an internal [[Prototype]] property
// Accessed via __proto__ or Object.getPrototypeOf()

const animal = {
  eats: true,
  walk() {
    console.log('Animal walks');
  }
};

const rabbit = {
  jumps: true
};

// Set prototype
rabbit.__proto__ = animal; // or Object.setPrototypeOf(rabbit, animal)

console.log(rabbit.eats);  // true (inherited)
console.log(rabbit.jumps); // true (own property)
rabbit.walk();             // "Animal walks" (inherited)

// Prototype chain
console.log(rabbit.toString()); // [object Object]
// Lookup: rabbit -> animal -> Object.prototype -> null

// Constructor functions and prototypes
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() {
  return `Hello, I'm ${this.name}`;
};

const john = new Person('John');
const jane = new Person('Jane');

console.log(john.greet()); // "Hello, I'm John"
console.log(jane.greet()); // "Hello, I'm Jane"

// Both share same prototype
console.log(john.greet === jane.greet); // true

// Prototype chain visualization
// john -> Person.prototype -> Object.prototype -> null

// Checking prototype
console.log(john.__proto__ === Person.prototype); // true
console.log(Person.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__); // null

// hasOwnProperty vs in operator
console.log(john.hasOwnProperty('name'));  // true
console.log(john.hasOwnProperty('greet')); // false
console.log('greet' in john);              // true (checks chain)
```

### Q10: What's the difference between `__proto__` and `prototype`?

**Expected Answer:**
```javascript
// prototype: Property of constructor functions
// __proto__: Property of instances (actual prototype reference)

function Dog(name) {
  this.name = name;
}

Dog.prototype.bark = function() {
  console.log('Woof!');
};

const buddy = new Dog('Buddy');

// prototype vs __proto__
console.log(Dog.prototype);           // { bark: f, constructor: Dog }
console.log(buddy.__proto__);         // { bark: f, constructor: Dog }
console.log(buddy.__proto__ === Dog.prototype); // true

// Dog (constructor) has .prototype
// buddy (instance) has .__proto__ pointing to Dog.prototype

// Visual representation:
// buddy -> buddy.__proto__ -> Dog.prototype -> Object.prototype -> null

// Constructor property
console.log(buddy.constructor === Dog); // true
console.log(Dog.prototype.constructor === Dog); // true

// Modern way: Object.getPrototypeOf()
console.log(Object.getPrototypeOf(buddy) === Dog.prototype); // true

// Setting prototype
const cat = {};
Object.setPrototypeOf(cat, Dog.prototype);
cat.bark(); // "Woof!" (now inherits from Dog)

// Why functions have prototype but objects don't
function MyFunc() {}
console.log(MyFunc.prototype);    // { constructor: MyFunc }

const obj = {};
console.log(obj.prototype);       // undefined
console.log(obj.__proto__);       // Object.prototype
```

### Q11: Implement inheritance without using ES6 classes.

**Expected Answer:**
```javascript
// Parent constructor
function Animal(name) {
  this.name = name;
  this.speed = 0;
}

Animal.prototype.run = function(speed) {
  this.speed = speed;
  console.log(`${this.name} runs at ${speed} km/h`);
};

Animal.prototype.stop = function() {
  this.speed = 0;
  console.log(`${this.name} stopped`);
};

// Child constructor
function Rabbit(name, earLength) {
  Animal.call(this, name); // Call parent constructor
  this.earLength = earLength;
}

// Inherit from Animal
Rabbit.prototype = Object.create(Animal.prototype);
Rabbit.prototype.constructor = Rabbit;

// Add child methods
Rabbit.prototype.jump = function() {
  console.log(`${this.name} jumps!`);
};

// Override parent method
Rabbit.prototype.stop = function() {
  this.jump();
  Animal.prototype.stop.call(this); // Call parent method
};

// Usage
const rabbit = new Rabbit('White Rabbit', 10);
rabbit.run(5);  // "White Rabbit runs at 5 km/h"
rabbit.jump();  // "White Rabbit jumps!"
rabbit.stop();  // "White Rabbit jumps!" then "White Rabbit stopped"

console.log(rabbit instanceof Rabbit); // true
console.log(rabbit instanceof Animal); // true

// Prototype chain
console.log(Object.getPrototypeOf(rabbit) === Rabbit.prototype); // true
console.log(Object.getPrototypeOf(Rabbit.prototype) === Animal.prototype); // true
```

### Q12: Compare ES5 inheritance vs ES6 classes.

**Expected Answer:**
```javascript
// ES5 - Prototypal inheritance
function PersonES5(name, age) {
  this.name = name;
  this.age = age;
}

PersonES5.prototype.greet = function() {
  return `Hi, I'm ${this.name}`;
};

PersonES5.staticMethod = function() {
  return 'Static method';
};

// ES6 - Class syntax (syntactic sugar over prototypes)
class PersonES6 {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  greet() {
    return `Hi, I'm ${this.name}`;
  }
  
  static staticMethod() {
    return 'Static method';
  }
}

// Both create same structure
const p1 = new PersonES5('John', 30);
const p2 = new PersonES6('Jane', 25);

console.log(p1.greet()); // "Hi, I'm John"
console.log(p2.greet()); // "Hi, I'm Jane"

// Key differences:

// 1. Classes must be invoked with 'new'
const p3 = PersonES5('Bob', 20); // Works (p3 is undefined, pollutes global)
// const p4 = PersonES6('Alice', 22); // TypeError: Class must be invoked with 'new'

// 2. Class methods are non-enumerable
console.log(Object.keys(PersonES5.prototype)); // ['greet']
console.log(Object.keys(PersonES6.prototype)); // []

// 3. Classes have strict mode by default
// In class methods, 'this' is undefined if not called on instance

// 4. No hoisting for classes
// const p = new MyClass(); // ReferenceError
// class MyClass {}

// ES6 inheritance
class Student extends PersonES6 {
  constructor(name, age, grade) {
    super(name, age); // Must call super()
    this.grade = grade;
  }
  
  study() {
    return `${this.name} is studying`;
  }
  
  greet() {
    return `${super.greet()}, grade ${this.grade}`;
  }
}

const student = new Student('Tom', 18, 'A');
console.log(student.greet()); // "Hi, I'm Tom, grade A"
```

---

## <a id="async"></a>⏳ Async JavaScript

### Q13: Explain Promises. How do they work internally?

**Expected Answer:**
```javascript
// Promise states: pending -> fulfilled/rejected
// Once settled (fulfilled/rejected), state can't change

// Creating a promise
const promise = new Promise((resolve, reject) => {
  // Async operation
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve('Operation successful');
    } else {
      reject('Operation failed');
    }
  }, 1000);
});

// Consuming a promise
promise
  .then(result => {
    console.log(result); // "Operation successful"
    return result.toUpperCase();
  })
  .then(upperResult => {
    console.log(upperResult); // "OPERATION SUCCESSFUL"
  })
  .catch(error => {
    console.error(error);
  })
  .finally(() => {
    console.log('Cleanup');
  });

// Promise chaining
fetch('/api/user')
  .then(response => response.json())
  .then(user => fetch(`/api/posts/${user.id}`))
  .then(response => response.json())
  .then(posts => console.log(posts))
  .catch(error => console.error(error));

// Promise.all - wait for all to resolve
const promise1 = Promise.resolve(3);
const promise2 = 42;
const promise3 = new Promise((resolve) => setTimeout(() => resolve('foo'), 100));

Promise.all([promise1, promise2, promise3])
  .then(values => {
    console.log(values); // [3, 42, "foo"]
  });

// If any rejects, Promise.all rejects immediately
Promise.all([
  Promise.resolve(1),
  Promise.reject('Error'),
  Promise.resolve(3)
]).catch(err => console.log(err)); // "Error"

// Promise.allSettled - wait for all to settle (resolve or reject)
Promise.allSettled([
  Promise.resolve(1),
  Promise.reject('Error'),
  Promise.resolve(3)
]).then(results => {
  console.log(results);
  // [
  //   { status: 'fulfilled', value: 1 },
  //   { status: 'rejected', reason: 'Error' },
  //   { status: 'fulfilled', value: 3 }
  // ]
});

// Promise.race - resolves/rejects with first settled promise
Promise.race([
  new Promise(resolve => setTimeout(() => resolve('slow'), 1000)),
  new Promise(resolve => setTimeout(() => resolve('fast'), 100))
]).then(result => console.log(result)); // "fast"

// Promise.any - resolves with first fulfilled promise
Promise.any([
  Promise.reject('Error 1'),
  new Promise(resolve => setTimeout(() => resolve('Success'), 100)),
  Promise.reject('Error 2')
]).then(result => console.log(result)); // "Success"
```

### Q14: What is async/await? How is it different from Promises?

**Expected Answer:**
```javascript
// async/await is syntactic sugar over Promises
// Makes async code look synchronous

// Promise version
function getUserPosts() {
  return fetch('/api/user')
    .then(response => response.json())
    .then(user => fetch(`/api/posts/${user.id}`))
    .then(response => response.json())
    .then(posts => {
      console.log(posts);
      return posts;
    })
    .catch(error => {
      console.error(error);
      throw error;
    });
}

// async/await version (much cleaner)
async function getUserPostsAsync() {
  try {
    const response = await fetch('/api/user');
    const user = await response.json();
    
    const postsResponse = await fetch(`/api/posts/${user.id}`);
    const posts = await postsResponse.json();
    
    console.log(posts);
    return posts;
  } catch (error) {
    console.error(error);
    throw error;
  }
}

// Key differences:

// 1. async function always returns a Promise
async function example() {
  return 'Hello'; // Wrapped in Promise.resolve('Hello')
}
example().then(result => console.log(result)); // "Hello"

// 2. await can only be used inside async functions
// await promise; // SyntaxError outside async function

// 3. Error handling with try/catch
async function fetchData() {
  try {
    const data = await fetch('/api/data');
    return data.json();
  } catch (error) {
    console.error('Fetch failed:', error);
    return null;
  }
}

// 4. Parallel execution with await
// Sequential (slower)
async function sequential() {
  const user = await fetchUser();     // Wait 1s
  const posts = await fetchPosts();   // Wait 1s
  // Total: 2s
}

// Parallel (faster)
async function parallel() {
  const [user, posts] = await Promise.all([
    fetchUser(),    // Both start immediately
    fetchPosts()
  ]);
  // Total: 1s (whichever is slower)
}

// Real-world example: Processing multiple items
async function processItems(items) {
  // BAD: Sequential processing
  for (const item of items) {
    await processItem(item); // Waits for each
  }
  
  // GOOD: Parallel processing
  await Promise.all(items.map(item => processItem(item)));
  
  // GOOD: Controlled concurrency
  const results = [];
  const batchSize = 5;
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(item => processItem(item))
    );
    results.push(...batchResults);
  }
  return results;
}

// Top-level await (ES2022)
// Only in modules
const data = await fetch('/api/data');
```

### Q15: Implement Promise.all from scratch.

**Expected Answer:**
```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    // Handle empty array
    if (promises.length === 0) {
      resolve([]);
      return;
    }
    
    const results = [];
    let completedCount = 0;
    
    promises.forEach((promise, index) => {
      // Ensure we're working with a promise
      Promise.resolve(promise)
        .then(value => {
          results[index] = value;
          completedCount++;
          
          // All promises resolved
          if (completedCount === promises.length) {
            resolve(results);
          }
        })
        .catch(error => {
          // Any rejection causes immediate rejection
          reject(error);
        });
    });
  });
}

// Test
promiseAll([
  Promise.resolve(1),
  Promise.resolve(2),
  Promise.resolve(3)
]).then(results => console.log(results)); // [1, 2, 3]

promiseAll([
  Promise.resolve(1),
  Promise.reject('Error'),
  Promise.resolve(3)
]).catch(error => console.log(error)); // "Error"

// Implement Promise.allSettled
function promiseAllSettled(promises) {
  return new Promise((resolve) => {
    const results = [];
    let completedCount = 0;
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = { status: 'fulfilled', value };
        })
        .catch(reason => {
          results[index] = { status: 'rejected', reason };
        })
        .finally(() => {
          completedCount++;
          if (completedCount === promises.length) {
            resolve(results);
          }
        });
    });
  });
}
```

### Q16: What are microtasks and macrotasks? Explain the difference.

**Expected Answer:**
```javascript
// Macrotasks (Task Queue):
// - setTimeout, setInterval
// - setImmediate (Node.js)
// - I/O operations
// - UI rendering

// Microtasks (Microtask Queue):
// - Promise callbacks (.then, .catch, .finally)
// - queueMicrotask
// - MutationObserver
// - process.nextTick (Node.js)

// Execution order: Call stack -> Microtasks -> Macrotasks

console.log('1: Script start');

setTimeout(() => {
  console.log('2: setTimeout'); // Macrotask
}, 0);

Promise.resolve()
  .then(() => {
    console.log('3: Promise 1'); // Microtask
  })
  .then(() => {
    console.log('4: Promise 2'); // Microtask
  });

console.log('5: Script end');

// Output:
// 1: Script start
// 5: Script end
// 3: Promise 1
// 4: Promise 2
// 2: setTimeout

// Why? Call stack runs first, then all microtasks, then macrotasks

// Complex example
console.log('Start');

setTimeout(() => console.log('Timeout 1'), 0);

Promise.resolve()
  .then(() => {
    console.log('Promise 1');
    setTimeout(() => console.log('Timeout 2'), 0);
  })
  .then(() => console.log('Promise 2'));

setTimeout(() => console.log('Timeout 3'), 0);

console.log('End');

// Output:
// Start
// End
// Promise 1
// Promise 2
// Timeout 1
// Timeout 3
// Timeout 2

// Real-world impact
async function example() {
  console.log('1');
  
  await Promise.resolve();
  console.log('2'); // Microtask
  
  setTimeout(() => console.log('3'), 0); // Macrotask
  
  await Promise.resolve();
  console.log('4'); // Microtask
}

example();
console.log('5');

// Output: 1, 5, 2, 4, 3
```

---

## <a id="event-loop"></a>🔄 Event Loop & Concurrency

### Q17: Explain JavaScript event loop in detail.

**Expected Answer:**
```javascript
// Event loop components:
// 1. Call Stack
// 2. Web APIs (browser) / C++ APIs (Node.js)
// 3. Callback Queue (Macrotask Queue)
// 4. Microtask Queue
// 5. Event Loop

// How it works:
// 1. Execute synchronous code (call stack)
// 2. When call stack is empty:
//    a. Process ALL microtasks
//    b. Process ONE macrotask
//    c. Repeat

function example() {
  console.log('1');
  
  setTimeout(() => console.log('2'), 0);
  
  Promise.resolve().then(() => console.log('3'));
  
  console.log('4');
}

example();

// Execution flow:
// Call Stack: example() -> console.log('1') -> setTimeout() -> Promise -> console.log('4')
// Web APIs: setTimeout registered
// Microtask Queue: Promise callback
// Macrotask Queue: setTimeout callback
// Call Stack empty -> Process microtasks -> console.log('3')
// Call Stack empty -> Process macrotask -> console.log('2')

// Output: 1, 4, 3, 2

// Blocking vs Non-blocking
function blockingExample() {
  console.log('Start');
  
  // Blocks the thread for 3 seconds
  const start = Date.now();
  while (Date.now() - start < 3000) {}
  
  console.log('End'); // After 3 seconds
}

function nonBlockingExample() {
  console.log('Start');
  
  // Doesn't block - uses event loop
  setTimeout(() => {
    console.log('End');
  }, 3000); // After 3 seconds, but doesn't block
  
  console.log('Middle'); // Executes immediately
}

// Visualizing the event loop
console.log('Script start');         // 1. Call stack

setTimeout(function() {               // 2. Registered in Web APIs
  console.log('setTimeout');          // 6. Macrotask queue -> call stack
}, 0);

Promise.resolve()                     // 3. Microtask queue
  .then(function() {
    console.log('Promise 1');         // 4. Microtask -> call stack
  })
  .then(function() {
    console.log('Promise 2');         // 5. Microtask -> call stack
  });

console.log('Script end');            // 3. Call stack

// Output:
// Script start
// Script end
// Promise 1
// Promise 2
// setTimeout
```

### Q18: How would you implement a sleep function?

**Expected Answer:**
```javascript
// Promise-based sleep (non-blocking)
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Usage with async/await
async function example() {
  console.log('Start');
  await sleep(2000);
  console.log('After 2 seconds');
}

// Usage with .then()
sleep(1000).then(() => console.log('After 1 second'));

// Blocking sleep (BAD - blocks event loop)
function blockingSleep(ms) {
  const start = Date.now();
  while (Date.now() - start < ms) {}
}

// This blocks everything
console.log('Before');
blockingSleep(3000); // UI freezes, no events processed
console.log('After');

// Modern approach with AbortController
function cancellableSleep(ms, signal) {
  return new Promise((resolve, reject) => {
    const timeout = setTimeout(resolve, ms);
    
    if (signal) {
      signal.addEventListener('abort', () => {
        clearTimeout(timeout);
        reject(new Error('Sleep cancelled'));
      });
    }
  });
}

// Usage
const controller = new AbortController();

async function example() {
  try {
    await cancellableSleep(5000, controller.signal);
    console.log('Completed');
  } catch (error) {
    console.log('Cancelled');
  }
}

example();
setTimeout(() => controller.abort(), 2000); // Cancel after 2s
```

---

## <a id="es6"></a>🆕 ES6+ Features

### Q19: Explain destructuring, spread operator, and rest parameters.

**Expected Answer:**
```javascript
// DESTRUCTURING

// Array destructuring
const [a, b, c] = [1, 2, 3];
const [first, , third] = [1, 2, 3]; // Skip elements
const [x, y, ...rest] = [1, 2, 3, 4, 5]; // rest = [3, 4, 5]

// Default values
const [p = 1, q = 2] = [10]; // p = 10, q = 2

// Swapping variables
let m = 1, n = 2;
[m, n] = [n, m]; // m = 2, n = 1

// Object destructuring
const user = { name: 'John', age: 30, city: 'NYC' };
const { name, age } = user;

// Rename variables
const { name: userName, age: userAge } = user;

// Default values
const { country = 'USA' } = user;

// Nested destructuring
const data = {
  user: {
    name: 'Jane',
    address: { city: 'LA', zip: 90001 }
  }
};
const { user: { address: { city } } } = data; // city = 'LA'

// Function parameter destructuring
function greet({ name, age }) {
  console.log(`${name} is ${age}`);
}
greet({ name: 'Bob', age: 25 });

// SPREAD OPERATOR (...)

// Array spreading
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// Copy array (shallow)
const original = [1, 2, 3];
const copy = [...original];

// Object spreading
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };
const merged = { ...obj1, ...obj2 }; // { a: 1, b: 2, c: 3, d: 4 }

// Override properties
const defaults = { theme: 'dark', lang: 'en' };
const userPrefs = { lang: 'fr' };
const settings = { ...defaults, ...userPrefs }; // lang: 'fr' wins

// Function arguments
function sum(a, b, c) {
  return a + b + c;
}
const numbers = [1, 2, 3];
sum(...numbers); // 6

// REST PARAMETERS

// Collect remaining arguments
function multiply(multiplier, ...numbers) {
  return numbers.map(n => n * multiplier);
}
multiply(2, 1, 2, 3); // [2, 4, 6]

// Must be last parameter
// function invalid(...args, last) {} // SyntaxError

// Difference from arguments object
function oldWay() {
  const args = Array.from(arguments); // Convert to array
  return args.reduce((sum, n) => sum + n, 0);
}

function newWay(...numbers) {
  // numbers is already an array
  return numbers.reduce((sum, n) => sum + n, 0);
}
```

### Q20: What are template literals and tagged templates?

**Expected Answer:**
```javascript
// TEMPLATE LITERALS

// Basic usage
const name = 'John';
const greeting = `Hello, ${name}!`; // "Hello, John!"

// Multi-line strings
const html = `
  <div>
    <h1>${name}</h1>
    <p>Welcome!</p>
  </div>
`;

// Expressions
const a = 5, b = 10;
console.log(`Sum: ${a + b}`); // "Sum: 15"

// Nested templates
const nested = `Outer ${`Inner ${5 + 5}`}`; // "Outer Inner 10"

// TAGGED TEMPLATES

// Custom processing of template literals
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    return `${result}${str}${values[i] ? `<mark>${values[i]}</mark>` : ''}`;
  }, '');
}

const name2 = 'Alice';
const age = 25;
const message = highlight`User ${name2} is ${age} years old`;
// "User <mark>Alice</mark> is <mark>25</mark> years old"

// SQL template tag (safe queries)
function sql(strings, ...values) {
  // Escape values to prevent SQL injection
  const escaped = values.map(val => {
    if (typeof val === 'string') {
      return val.replace(/'/g, "''");
    }
    return val;
  });
  
  return strings.reduce((query, str, i) => {
    return query + str + (escaped[i] || '');
  }, '');
}

const username = "John'; DROP TABLE users--";
const query = sql`SELECT * FROM users WHERE name = '${username}'`;
// Safely escapes the malicious input

// Styled-components example (real-world)
function css(strings, ...values) {
  return strings.reduce((styles, str, i) => {
    const value = values[i];
    if (typeof value === 'function') {
      // Handle functions (props => value)
      return styles + str + value;
    }
    return styles + str + (value || '');
  }, '');
}

const primary = '#007bff';
const styles = css`
  color: ${primary};
  padding: ${props => props.large ? '20px' : '10px'};
`;

// i18n template tag
function i18n(strings, ...values) {
  // Get translation key
  const key = strings[0].trim();
  
  // Get translation from dictionary
  const translation = translations[key] || key;
  
  // Replace placeholders
  return translation.replace(/\{(\d+)\}/g, (_, index) => {
    return values[index];
  });
}

const count = 5;
const text = i18n`You have {0} messages`(count);
```

### Q21: Explain Map, Set, WeakMap, and WeakSet.

**Expected Answer:**
```javascript
// MAP - Key-value pairs with any type of key

const map = new Map();

// Set values
map.set('name', 'John');
map.set(1, 'number key');
map.set(true, 'boolean key');
map.set({}, 'object key');

// Get values
console.log(map.get('name')); // 'John'
console.log(map.get(1));      // 'number key'

// Check existence
console.log(map.has('name')); // true

// Delete
map.delete('name');

// Size
console.log(map.size); // 3

// Iterate
map.forEach((value, key) => {
  console.log(key, value);
});

for (const [key, value] of map) {
  console.log(key, value);
}

// Convert to/from object
const obj = { a: 1, b: 2 };
const mapFromObj = new Map(Object.entries(obj));
const objFromMap = Object.fromEntries(map);

// SET - Unique values collection

const set = new Set();

// Add values
set.add(1);
set.add(2);
set.add(2); // Ignored - duplicate
set.add(3);

console.log(set.size); // 3

// Check existence
console.log(set.has(2)); // true

// Delete
set.delete(2);

// Clear all
set.clear();

// Iterate
set.forEach(value => console.log(value));

// Remove duplicates from array
const numbers = [1, 2, 2, 3, 3, 4];
const unique = [...new Set(numbers)]; // [1, 2, 3, 4]

// WEAKMAP - Keys must be objects, allows garbage collection

let obj1 = { id: 1 };
let obj2 = { id: 2 };

const weakMap = new WeakMap();
weakMap.set(obj1, 'data for obj1');
weakMap.set(obj2, 'data for obj2');

console.log(weakMap.get(obj1)); // 'data for obj1'

// When obj1 is no longer referenced, it can be garbage collected
obj1 = null; // WeakMap entry is automatically removed

// No size property, not iterable
// console.log(weakMap.size); // undefined
// for (const [k, v] of weakMap) {} // TypeError

// Use case: Private data
const privateData = new WeakMap();

class User {
  constructor(name) {
    privateData.set(this, { name });
  }
  
  getName() {
    return privateData.get(this).name;
  }
}

// WEAKSET - Values must be objects

let item1 = { id: 1 };
let item2 = { id: 2 };

const weakSet = new WeakSet();
weakSet.add(item1);
weakSet.add(item2);

console.log(weakSet.has(item1)); // true

item1 = null; // Entry removed automatically

// Use case: Tracking object references
const visited = new WeakSet();

function traverse(node) {
  if (visited.has(node)) return;
  visited.add(node);
  // Process node...
}

// Map vs Object differences
const objMap = {};
const map2 = new Map();

// 1. Key types
objMap[{}] = 'value'; // Key becomes "[object Object]" string
map2.set({}, 'value'); // Key is actual object

// 2. Insertion order
// Map preserves insertion order, object doesn't guarantee order

// 3. Size
console.log(Object.keys(objMap).length); // Manual counting
console.log(map2.size);                   // Built-in

// 4. Performance
// Map is faster for frequent additions/deletions
```

---

## <a id="functional"></a>🔧 Functional Programming

### Q22: Implement map, filter, and reduce from scratch.

**Expected Answer:**
```javascript
// MAP
Array.prototype.myMap = function(callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    result.push(callback(this[i], i, this));
  }
  return result;
};

[1, 2, 3].myMap(x => x * 2); // [2, 4, 6]

// FILTER
Array.prototype.myFilter = function(callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (callback(this[i], i, this)) {
      result.push(this[i]);
    }
  }
  return result;
};

[1, 2, 3, 4].myFilter(x => x % 2 === 0); // [2, 4]

// REDUCE
Array.prototype.myReduce = function(callback, initialValue) {
  let accumulator = initialValue;
  let startIndex = 0;
  
  // If no initial value, use first element
  if (accumulator === undefined) {
    accumulator = this[0];
    startIndex = 1;
  }
  
  for (let i = startIndex; i < this.length; i++) {
    accumulator = callback(accumulator, this[i], i, this);
  }
  
  return accumulator;
};

[1, 2, 3, 4].myReduce((sum, x) => sum + x, 0); // 10

// FOREACH
Array.prototype.myForEach = function(callback) {
  for (let i = 0; i < this.length; i++) {
    callback(this[i], i, this);
  }
};

// SOME
Array.prototype.mySome = function(callback) {
  for (let i = 0; i < this.length; i++) {
    if (callback(this[i], i, this)) {
      return true;
    }
  }
  return false;
};

// EVERY
Array.prototype.myEvery = function(callback) {
  for (let i = 0; i < this.length; i++) {
    if (!callback(this[i], i, this)) {
      return false;
    }
  }
  return true;
};

// FIND
Array.prototype.myFind = function(callback) {
  for (let i = 0; i < this.length; i++) {
    if (callback(this[i], i, this)) {
      return this[i];
    }
  }
  return undefined;
};

// FLAT
Array.prototype.myFlat = function(depth = 1) {
  const result = [];
  
  const flatten = (arr, currentDepth) => {
    for (const item of arr) {
      if (Array.isArray(item) && currentDepth < depth) {
        flatten(item, currentDepth + 1);
      } else {
        result.push(item);
      }
    }
  };
  
  flatten(this, 0);
  return result;
};

[1, [2, [3, [4]]]].myFlat(2); // [1, 2, 3, [4]]
```

### Q23: What are pure functions? Implement examples.

**Expected Answer:**
```javascript
// Pure function: Same input always returns same output, no side effects

// PURE FUNCTIONS

// Example 1: Simple calculation
function add(a, b) {
  return a + b; // Pure - deterministic, no side effects
}

// Example 2: Array transformation
function double(numbers) {
  return numbers.map(n => n * 2); // Pure - returns new array
}

// Example 3: Object transformation
function updateUser(user, updates) {
  return { ...user, ...updates }; // Pure - returns new object
}

// IMPURE FUNCTIONS

// Example 1: Uses external variable
let count = 0;
function increment() {
  count++; // Impure - modifies external state
  return count;
}

// Example 2: Random output
function getRandom() {
  return Math.random(); // Impure - non-deterministic
}

// Example 3: Mutates input
function sortArray(arr) {
  arr.sort(); // Impure - mutates original array
  return arr;
}

// Pure version
function pureSort(arr) {
  return [...arr].sort(); // Pure - doesn't mutate input
}

// Example 4: Side effects (API call, DOM manipulation)
function saveUser(user) {
  fetch('/api/user', { // Impure - network side effect
    method: 'POST',
    body: JSON.stringify(user)
  });
}

// Example 5: Date dependency
function getCurrentYear() {
  return new Date().getFullYear(); // Impure - depends on current time
}

// Pure version
function getYear(date) {
  return date.getFullYear(); // Pure - deterministic
}

// BENEFITS OF PURE FUNCTIONS

// 1. Testability
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price, 0);
}
// Easy to test - predictable output

// 2. Cacheable (memoization)
function memoize(fn) {
  const cache = {};
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache[key]) {
      return cache[key];
    }
    const result = fn.apply(this, args);
    cache[key] = result;
    return result;
  };
}

const expensiveCalculation = memoize((n) => {
  // Pure function - can be safely cached
  return n * n;
});

// 3. Parallelizable
const numbers = [1, 2, 3, 4, 5];
// Can safely process in parallel since pure
numbers.map(n => n * 2); // No race conditions

// 4. Composable
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);

const addOne = x => x + 1;
const double = x => x * 2;
const square = x => x * x;

const transform = compose(square, double, addOne);
console.log(transform(3)); // ((3 + 1) * 2)² = 64
```

### Q24: Explain currying and implement curry function.

**Expected Answer:**
```javascript
// Currying: Transform function f(a, b, c) into f(a)(b)(c)

// Manual currying
function add(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}

console.log(add(1)(2)(3)); // 6

// Generic curry function
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      return function(...nextArgs) {
        return curried.apply(this, [...args, ...nextArgs]);
      };
    }
  };
}

// Usage
function multiply(a, b, c) {
  return a * b * c;
}

const curriedMultiply = curry(multiply);

console.log(curriedMultiply(2)(3)(4)); // 24
console.log(curriedMultiply(2, 3)(4)); // 24
console.log(curriedMultiply(2)(3, 4)); // 24

// Real-world use cases

// 1. Partial application
const add = curry((a, b, c) => a + b + c);
const add5 = add(5);
const add5and10 = add5(10);
console.log(add5and10(3)); // 18

// 2. Function composition
const map = curry((fn, array) => array.map(fn));
const filter = curry((predicate, array) => array.filter(predicate));

const double = x => x * 2;
const isEven = x => x % 2 === 0;

const doubleEvens = compose(
  map(double),
  filter(isEven)
);

console.log(doubleEvens([1, 2, 3, 4])); // [4, 8]

// 3. Event handlers
const handleEvent = curry((type, handler, element) => {
  element.addEventListener(type, handler);
});

const onClick = handleEvent('click');
const onSubmit = handleEvent('submit');

onClick(handler1, button1);
onClick(handler2, button2);

// 4. API requests
const apiCall = curry((method, url, data) => {
  return fetch(url, {
    method,
    body: JSON.stringify(data)
  });
});

const get = apiCall('GET');
const post = apiCall('POST');

get('/api/users')().then(res => res.json());
post('/api/users')({ name: 'John' }).then(res => res.json());
```

---

## <a id="performance"></a>⚡ Memory Management & Performance

### Q25: Explain garbage collection in JavaScript.

**Expected Answer:**
```javascript
// JavaScript uses automatic garbage collection
// Main algorithm: Mark-and-Sweep

// MEMORY LEAKS - Common causes

// 1. Global variables
function createLeak() {
  leakedVar = 'I am global'; // No var/let/const - becomes global
}

// 2. Forgotten timers
const someResource = getData();
setInterval(() => {
  const node = document.getElementById('node');
  if (node) {
    node.innerHTML = JSON.stringify(someResource);
  }
}, 1000);
// Even if node is removed, callback keeps reference to someResource

// Fix: Clear timer
const intervalId = setInterval(() => {}, 1000);
clearInterval(intervalId);

// 3. Closures holding references
function createClosure() {
  const largeData = new Array(1000000);
  
  return function() {
    // Even though largeData isn't used, it's kept in memory
    console.log('Hello');
  };
}

// Fix: Remove unused references
function createClosure() {
  const largeData = new Array(1000000);
  const processedData = processData(largeData);
  
  return function() {
    console.log(processedData);
  };
  // largeData can be garbage collected
}

// 4. DOM references
const elements = [];
function addElement() {
  const el = document.createElement('div');
  document.body.appendChild(el);
  elements.push(el); // Keeps reference even if removed from DOM
}

document.body.innerHTML = ''; // Removes elements but array still references them

// Fix: Clear references
elements.length = 0;

// 5. Event listeners
const button = document.querySelector('button');
function handleClick() {
  // Heavy operation
}
button.addEventListener('click', handleClick);

// When removing button, listener still exists
// Fix: Remove listener
button.removeEventListener('click', handleClick);

// BEST PRACTICES

// 1. Use WeakMap/WeakSet for object references
const cache = new WeakMap();
function process(obj) {
  if (!cache.has(obj)) {
    cache.set(obj, heavyComputation(obj));
  }
  return cache.get(obj);
}
// When obj is garbage collected, cache entry is too

// 2. Nullify references
let heavyObject = { data: new Array(1000000) };
// Done with it
heavyObject = null; // Eligible for GC

// 3. Avoid circular references (though modern engines handle this)
function CircularRef() {
  this.me = this; // Circular reference
}
// Modern engines handle this, but avoid if possible

// 4. Use local variables
function process() {
  // Good: local scope
  const data = getData();
  return transform(data);
  // data is eligible for GC after function returns
}

function processBad() {
  // Bad: accidentally global
  data = getData();
  return transform(data);
  // data stays in memory
}
```

### Q26: What is debouncing and throttling? Implement both.

**Expected Answer:**
```javascript
// DEBOUNCING
// Delays execution until after wait time has elapsed since last call
// Use case: Search input, window resize

function debounce(func, wait) {
  let timeoutId;
  
  return function(...args) {
    const context = this;
    
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      func.apply(context, args);
    }, wait);
  };
}

// Usage: Search
const searchInput = document.querySelector('#search');
const handleSearch = debounce((event) => {
  console.log('Searching for:', event.target.value);
  // API call here
}, 300);

searchInput.addEventListener('input', handleSearch);
// User types "hello" - only fires once, 300ms after last keystroke

// THROTTLING
// Ensures function is called at most once per specified time period
// Use case: Scroll events, mouse movement

function throttle(func, limit) {
  let inThrottle;
  
  return function(...args) {
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

// Usage: Scroll
const handleScroll = throttle(() => {
  console.log('Scroll position:', window.scrollY);
}, 1000);

window.addEventListener('scroll', handleScroll);
// Fires once per second while scrolling

// ADVANCED: Leading vs Trailing execution

// Debounce with leading option
function debounceLeading(func, wait, leading = false) {
  let timeoutId;
  
  return function(...args) {
    const context = this;
    const callNow = leading && !timeoutId;
    
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      timeoutId = null;
      if (!leading) func.apply(context, args);
    }, wait);
    
    if (callNow) func.apply(context, args);
  };
}

// Throttle with trailing option
function throttleTrailing(func, limit, trailing = true) {
  let inThrottle;
  let lastFunc;
  let lastTime;
  
  return function(...args) {
    const context = this;
    
    if (!inThrottle) {
      func.apply(context, args);
      lastTime = Date.now();
      inThrottle = true;
    } else if (trailing) {
      clearTimeout(lastFunc);
      lastFunc = setTimeout(() => {
        if (Date.now() - lastTime >= limit) {
          func.apply(context, args);
          lastTime = Date.now();
        }
      }, Math.max(limit - (Date.now() - lastTime), 0));
    }
    
    setTimeout(() => {
      inThrottle = false;
    }, limit);
  };
}

// COMPARISON

// Debounce: "Wait until user stops typing"
// ---|-----|-----|-----|-----| (keystrokes)
//                              X (fires once at end)

// Throttle: "Fire once per second while user types"
// ---|-----|-----|-----|-----| (keystrokes)
// X--------X--------X--------X (fires at intervals)
```

### Q27: How do you optimize performance for large lists?

**Expected Answer:**
```javascript
// PROBLEM: Rendering 10,000+ items

// BAD: Render everything at once
function renderAll(items) {
  const container = document.getElementById('list');
  items.forEach(item => {
    const div = document.createElement('div');
    div.textContent = item.name;
    container.appendChild(div);
  });
  // Slow: Triggers 10,000 reflows
}

// SOLUTION 1: Document Fragment
function renderWithFragment(items) {
  const fragment = document.createDocumentFragment();
  items.forEach(item => {
    const div = document.createElement('div');
    div.textContent = item.name;
    fragment.appendChild(div);
  });
  document.getElementById('list').appendChild(fragment);
  // Better: Single reflow
}

// SOLUTION 2: Virtual Scrolling
class VirtualScroller {
  constructor(items, container, itemHeight) {
    this.items = items;
    this.container = container;
    this.itemHeight = itemHeight;
    this.visibleCount = Math.ceil(container.clientHeight / itemHeight);
    
    this.render();
    this.container.addEventListener('scroll', () => this.render());
  }
  
  render() {
    const scrollTop = this.container.scrollTop;
    const startIndex = Math.floor(scrollTop / this.itemHeight);
    const endIndex = startIndex + this.visibleCount;
    
    const visibleItems = this.items.slice(startIndex, endIndex);
    
    this.container.innerHTML = '';
    this.container.style.height = `${this.items.length * this.itemHeight}px`;
    
    visibleItems.forEach((item, i) => {
      const div = document.createElement('div');
      div.style.position = 'absolute';
      div.style.top = `${(startIndex + i) * this.itemHeight}px`;
      div.textContent = item.name;
      this.container.appendChild(div);
    });
  }
}

// Usage
const items = Array.from({ length: 100000 }, (_, i) => ({ name: `Item ${i}` }));
new VirtualScroller(items, document.getElementById('list'), 50);

// SOLUTION 3: Pagination
function renderPage(items, page, pageSize) {
  const start = page * pageSize;
  const end = start + pageSize;
  const pageItems = items.slice(start, end);
  
  const container = document.getElementById('list');
  container.innerHTML = '';
  
  pageItems.forEach(item => {
    const div = document.createElement('div');
    div.textContent = item.name;
    container.appendChild(div);
  });
}

// SOLUTION 4: Lazy Loading / Infinite Scroll
class InfiniteScroller {
  constructor(loadMore) {
    this.loadMore = loadMore;
    this.loading = false;
    
    window.addEventListener('scroll', () => {
      if (this.isNearBottom() && !this.loading) {
        this.loading = true;
        this.loadMore().then(() => {
          this.loading = false;
        });
      }
    });
  }
  
  isNearBottom() {
    const scrollTop = window.scrollY;
    const windowHeight = window.innerHeight;
    const documentHeight = document.documentElement.scrollHeight;
    return scrollTop + windowHeight >= documentHeight - 100;
  }
}

// Usage
let page = 0;
const scroller = new InfiniteScroller(async () => {
  const newItems = await fetchPage(page++);
  appendItems(newItems);
});

// SOLUTION 5: RequestAnimationFrame for smooth updates
function renderChunked(items, chunkSize = 100) {
  let index = 0;
  
  function renderChunk() {
    const chunk = items.slice(index, index + chunkSize);
    const fragment = document.createDocumentFragment();
    
    chunk.forEach(item => {
      const div = document.createElement('div');
      div.textContent = item.name;
      fragment.appendChild(div);
    });
    
    document.getElementById('list').appendChild(fragment);
    
    index += chunkSize;
    
    if (index < items.length) {
      requestAnimationFrame(renderChunk);
    }
  }
  
  renderChunk();
}
```

---

## <a id="patterns"></a>🎨 Design Patterns

### Q28: Implement the Module Pattern.

**Expected Answer:**
```javascript
// MODULE PATTERN
// Encapsulation: private and public members

// Classic Module Pattern
const Calculator = (function() {
  // Private variables and functions
  let result = 0;
  
  function validateNumber(num) {
    return typeof num === 'number' && !isNaN(num);
  }
  
  // Public API
  return {
    add(num) {
      if (validateNumber(num)) {
        result += num;
      }
      return this; // Chaining
    },
    
    subtract(num) {
      if (validateNumber(num)) {
        result -= num;
      }
      return this;
    },
    
    getResult() {
      return result;
    },
    
    reset() {
      result = 0;
      return this;
    }
  };
})();

// Usage
Calculator.add(5).add(3).subtract(2).getResult(); // 6
// console.log(result); // ReferenceError - private

// REVEALING MODULE PATTERN
const UserManager = (function() {
  // Private
  const users = [];
  
  function findById(id) {
    return users.find(user => user.id === id);
  }
  
  function addUser(user) {
    if (!findById(user.id)) {
      users.push(user);
      return true;
    }
    return false;
  }
  
  function removeUser(id) {
    const index = users.findIndex(user => user.id === id);
    if (index !== -1) {
      users.splice(index, 1);
      return true;
    }
    return false;
  }
  
  function getAllUsers() {
    return [...users]; // Return copy
  }
  
  // Reveal public methods
  return {
    add: addUser,
    remove: removeUser,
    getAll: getAllUsers,
    find: findById
  };
})();

// ES6 MODULE PATTERN
class CounterModule {
  #count = 0; // Private field
  
  increment() {
    this.#count++;
    return this;
  }
  
  decrement() {
    this.#count--;
    return this;
  }
  
  get value() {
    return this.#count;
  }
}

// SINGLETON MODULE
const DatabaseConnection = (function() {
  let instance;
  
  function createInstance() {
    return {
      connect() {
        console.log('Connected to database');
      },
      query(sql) {
        console.log('Executing:', sql);
      }
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

// Usage
const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();
console.log(db1 === db2); // true - same instance
```

### Q29: Implement Observer Pattern (Pub/Sub).

**Expected Answer:**
```javascript
// OBSERVER PATTERN / PUB-SUB

class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  // Subscribe to event
  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
    
    // Return unsubscribe function
    return () => this.off(event, callback);
  }
  
  // Subscribe once
  once(event, callback) {
    const wrapper = (...args) => {
      callback(...args);
      this.off(event, wrapper);
    };
    this.on(event, wrapper);
  }
  
  // Unsubscribe
  off(event, callback) {
    if (!this.events[event]) return;
    
    this.events[event] = this.events[event].filter(cb => cb !== callback);
  }
  
  // Emit event
  emit(event, ...args) {
    if (!this.events[event]) return;
    
    this.events[event].forEach(callback => {
      callback(...args);
    });
  }
  
  // Remove all listeners
  removeAllListeners(event) {
    if (event) {
      delete this.events[event];
    } else {
      this.events = {};
    }
  }
}

// Usage
const emitter = new EventEmitter();

// Subscribe
const unsubscribe = emitter.on('user:login', (user) => {
  console.log('User logged in:', user.name);
});

emitter.on('user:login', (user) => {
  console.log('Send welcome email to:', user.email);
});

// Emit
emitter.emit('user:login', { name: 'John', email: 'john@example.com' });

// Unsubscribe
unsubscribe();

// Real-world example: Store
class Store extends EventEmitter {
  constructor(initialState = {}) {
    super();
    this.state = initialState;
  }
  
  getState() {
    return this.state;
  }
  
  setState(newState) {
    const oldState = this.state;
    this.state = { ...this.state, ...newState };
    this.emit('change', this.state, oldState);
  }
  
  subscribe(callback) {
    return this.on('change', callback);
  }
}

// Usage
const store = new Store({ count: 0 });

store.subscribe((newState, oldState) => {
  console.log('State changed:', oldState, '->', newState);
});

store.setState({ count: 1 });
store.setState({ count: 2 });

// Advanced: Async Event Emitter
class AsyncEventEmitter extends EventEmitter {
  async emitAsync(event, ...args) {
    if (!this.events[event]) return;
    
    await Promise.all(
      this.events[event].map(callback => callback(...args))
    );
  }
}

// Usage with async handlers
const asyncEmitter = new AsyncEventEmitter();

asyncEmitter.on('data:fetch', async (url) => {
  const response = await fetch(url);
  const data = await response.json();
  console.log('Data fetched:', data);
});

await asyncEmitter.emitAsync('data:fetch', '/api/users');
```

### Q30: Implement Factory and Builder patterns.

**Expected Answer:**
```javascript
// FACTORY PATTERN
// Creates objects without specifying exact class

class Car {
  constructor(options) {
    this.doors = options.doors || 4;
    this.state = options.state || 'brand new';
    this.color = options.color || 'silver';
  }
}

class Truck {
  constructor(options) {
    this.doors = options.doors || 2;
    this.state = options.state || 'used';
    this.wheelSize = options.wheelSize || 'large';
  }
}

class VehicleFactory {
  createVehicle(type, options) {
    switch(type) {
      case 'car':
        return new Car(options);
      case 'truck':
        return new Truck(options);
      default:
        throw new Error('Unknown vehicle type');
    }
  }
}

// Usage
const factory = new VehicleFactory();
const myCar = factory.createVehicle('car', { color: 'red' });
const myTruck = factory.createVehicle('truck', { wheelSize: 'extra large' });

// ABSTRACT FACTORY PATTERN
class Button {
  render() {
    throw new Error('Must implement render');
  }
}

class WindowsButton extends Button {
  render() {
    return '<button>Windows Button</button>';
  }
}

class MacButton extends Button {
  render() {
    return '<button>Mac Button</button>';
  }
}

class UIFactory {
  createButton() {
    throw new Error('Must implement createButton');
  }
}

class WindowsFactory extends UIFactory {
  createButton() {
    return new WindowsButton();
  }
}

class MacFactory extends UIFactory {
  createButton() {
    return new MacButton();
  }
}

// Usage
function renderUI(factory) {
  const button = factory.createButton();
  document.body.innerHTML = button.render();
}

const os = 'Windows';
const factory2 = os === 'Windows' ? new WindowsFactory() : new MacFactory();
renderUI(factory2);

// BUILDER PATTERN
// Construct complex objects step by step

class QueryBuilder {
  constructor() {
    this.query = {
      select: [],
      from: '',
      where: [],
      orderBy: [],
      limit: null
    };
  }
  
  select(...fields) {
    this.query.select.push(...fields);
    return this; // Chaining
  }
  
  from(table) {
    this.query.from = table;
    return this;
  }
  
  where(condition) {
    this.query.where.push(condition);
    return this;
  }
  
  orderBy(field, direction = 'ASC') {
    this.query.orderBy.push({ field, direction });
    return this;
  }
  
  limit(count) {
    this.query.limit = count;
    return this;
  }
  
  build() {
    let sql = `SELECT ${this.query.select.join(', ')} FROM ${this.query.from}`;
    
    if (this.query.where.length > 0) {
      sql += ` WHERE ${this.query.where.join(' AND ')}`;
    }
    
    if (this.query.orderBy.length > 0) {
      const orderClauses = this.query.orderBy.map(
        ({ field, direction }) => `${field} ${direction}`
      );
      sql += ` ORDER BY ${orderClauses.join(', ')}`;
    }
    
    if (this.query.limit) {
      sql += ` LIMIT ${this.query.limit}`;
    }
    
    return sql;
  }
}

// Usage
const query = new QueryBuilder()
  .select('id', 'name', 'email')
  .from('users')
  .where('age > 18')
  .where('active = true')
  .orderBy('name', 'ASC')
  .limit(10)
  .build();

console.log(query);
// SELECT id, name, email FROM users WHERE age > 18 AND active = true ORDER BY name ASC LIMIT 10

// BUILDER PATTERN: Complex object construction
class User {
  constructor(builder) {
    this.name = builder.name;
    this.email = builder.email;
    this.age = builder.age;
    this.address = builder.address;
    this.phone = builder.phone;
  }
}

class UserBuilder {
  setName(name) {
    this.name = name;
    return this;
  }
  
  setEmail(email) {
    this.email = email;
    return this;
  }
  
  setAge(age) {
    this.age = age;
    return this;
  }
  
  setAddress(address) {
    this.address = address;
    return this;
  }
  
  setPhone(phone) {
    this.phone = phone;
    return this;
  }
  
  build() {
    return new User(this);
  }
}

// Usage
const user = new UserBuilder()
  .setName('John Doe')
  .setEmail('john@example.com')
  .setAge(30)
  .setAddress('123 Main St')
  .build();
```

---

## <a id="security"></a>🔒 Security

### Q31: How do you prevent XSS attacks in JavaScript?

**Expected Answer:**
```javascript
// XSS (Cross-Site Scripting) - Injecting malicious scripts

// VULNERABLE CODE

// 1. innerHTML with user input
const userInput = '<img src=x onerror="alert(\'XSS\')">';
document.getElementById('output').innerHTML = userInput; // BAD!

// 2. eval() with user input
const code = prompt('Enter code');
eval(code); // BAD! User can execute arbitrary code

// 3. document.write()
document.write(userInput); // BAD!

// 4. location.href with user input
location.href = 'javascript:alert("XSS")'; // BAD!

// SAFE PRACTICES

// 1. Use textContent instead of innerHTML
document.getElementById('output').textContent = userInput; // GOOD!
// Treats everything as text, not HTML

// 2. Sanitize HTML if you must use innerHTML
function sanitizeHTML(html) {
  const div = document.createElement('div');
  div.textContent = html;
  return div.innerHTML;
}

document.getElementById('output').innerHTML = sanitizeHTML(userInput); // GOOD!

// 3. Use DOMPurify library for complex HTML
// import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userInput);
document.getElementById('output').innerHTML = clean;

// 4. Escape HTML special characters
function escapeHTML(str) {
  const div = document.createElement('div');
  div.appendChild(document.createTextNode(str));
  return div.innerHTML;
}

// Or manual escaping
function escapeHTML2(str) {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;');
}

// 5. Validate and sanitize URLs
function isSafeURL(url) {
  const allowedProtocols = ['http:', 'https:', 'mailto:'];
  try {
    const parsed = new URL(url);
    return allowedProtocols.includes(parsed.protocol);
  } catch {
    return false;
  }
}

const userURL = 'javascript:alert("XSS")';
if (isSafeURL(userURL)) {
  location.href = userURL;
}

// 6. Use CSP (Content Security Policy) headers
// Set in HTTP headers:
// Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.com

// 7. HTTPOnly cookies
// Set cookies with HTTPOnly flag (server-side)
// document.cookie cannot access these cookies

// 8. Validate and sanitize on server-side
// Client-side validation can be bypassed
// Always validate on server

// 9. Modern frameworks handle this automatically
// React example:
const element = <div>{userInput}</div>; // Automatically escaped
// To render HTML: dangerouslySetInnerHTML={{__html: sanitizedHTML}}

// 10. Template literals - be careful
const template = `<div>${userInput}</div>`; // Still needs escaping!
document.body.innerHTML = template; // Vulnerable

// CORRECT: Escape first
const template = `<div>${escapeHTML(userInput)}</div>`;
```

### Q32: How do you securely store sensitive data in JavaScript?

**Expected Answer:**
```javascript
// NEVER store sensitive data in:
// - LocalStorage
// - SessionStorage
// - Cookies (without security flags)
// - JavaScript variables (can be accessed via console)

// SECURE PRACTICES

// 1. Use HTTPOnly, Secure, SameSite cookies (server-side)
// Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Strict

// 2. Store tokens in memory only
class AuthService {
  #accessToken = null; // Private field
  #refreshToken = null;
  
  setTokens(access, refresh) {
    this.#accessToken = access;
    this.#refreshToken = refresh;
  }
  
  getAccessToken() {
    return this.#accessToken;
  }
  
  clearTokens() {
    this.#accessToken = null;
    this.#refreshToken = null;
  }
}

// 3. Encrypt sensitive data before storing
async function encryptData(data, password) {
  const encoder = new TextEncoder();
  const dataBuffer = encoder.encode(data);
  
  // Derive key from password
  const passwordKey = await crypto.subtle.importKey(
    'raw',
    encoder.encode(password),
    'PBKDF2',
    false,
    ['deriveKey']
  );
  
  const aesKey = await crypto.subtle.deriveKey(
    {
      name: 'PBKDF2',
      salt: encoder.encode('salt'),
      iterations: 100000,
      hash: 'SHA-256'
    },
    passwordKey,
    { name: 'AES-GCM', length: 256 },
    false,
    ['encrypt']
  );
  
  // Encrypt
  const iv = crypto.getRandomValues(new Uint8Array(12));
  const encrypted = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv },
    aesKey,
    dataBuffer
  );
  
  return { encrypted, iv };
}

// 4. Use environment variables for secrets (server-side)
// process.env.API_KEY
// Never hardcode: const API_KEY = '12345'; // BAD!

// 5. Implement proper session management
class SessionManager {
  constructor() {
    this.sessionTimeout = 30 * 60 * 1000; // 30 minutes
    this.lastActivity = Date.now();
    this.checkSession();
  }
  
  checkSession() {
    setInterval(() => {
      if (Date.now() - this.lastActivity > this.sessionTimeout) {
        this.logout();
      }
    }, 60000); // Check every minute
  }
  
  resetActivity() {
    this.lastActivity = Date.now();
  }
  
  logout() {
    // Clear all session data
    console.log('Session expired');
  }
}

// 6. Hash passwords (never store plain text)
async function hashPassword(password) {
  const encoder = new TextEncoder();
  const data = encoder.encode(password);
  const hash = await crypto.subtle.digest('SHA-256', data);
  return Array.from(new Uint8Array(hash))
    .map(b => b.toString(16).padStart(2, '0'))
    .join('');
}

// 7. Use secure randomness
const secureRandom = crypto.getRandomValues(new Uint8Array(32));
// NOT: Math.random() for security purposes

// 8. Implement rate limiting
class RateLimiter {
  constructor(maxAttempts, windowMs) {
    this.attempts = new Map();
    this.maxAttempts = maxAttempts;
    this.windowMs = windowMs;
  }
  
  checkLimit(identifier) {
    const now = Date.now();
    const attempts = this.attempts.get(identifier) || [];
    
    // Remove old attempts
    const recentAttempts = attempts.filter(
      time => now - time < this.windowMs
    );
    
    if (recentAttempts.length >= this.maxAttempts) {
      return false; // Rate limit exceeded
    }
    
    recentAttempts.push(now);
    this.attempts.set(identifier, recentAttempts);
    return true;
  }
}

// Usage
const limiter = new RateLimiter(5, 60000); // 5 attempts per minute

function login(username, password) {
  if (!limiter.checkLimit(username)) {
    throw new Error('Too many login attempts');
  }
  // Proceed with login
}
```

---

## <a id="scenarios"></a>🎯 Real-World Scenarios

### Q33: Implement deep clone of an object.

**Expected Answer:**
```javascript
// SHALLOW CLONE (doesn't copy nested objects)
const obj = { a: 1, b: { c: 2 } };
const shallow1 = { ...obj };
const shallow2 = Object.assign({}, obj);

shallow1.b.c = 3;
console.log(obj.b.c); // 3 - modified original!

// DEEP CLONE IMPLEMENTATIONS

// 1. JSON method (limitations: loses functions, undefined, Date, etc.)
function deepClone1(obj) {
  return JSON.parse(JSON.stringify(obj));
}

// 2. Recursive deep clone
function deepClone(obj, hash = new WeakMap()) {
  // Handle primitives and null
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  // Handle circular references
  if (hash.has(obj)) {
    return hash.get(obj);
  }
  
  // Handle Date
  if (obj instanceof Date) {
    return new Date(obj);
  }
  
  // Handle RegExp
  if (obj instanceof RegExp) {
    return new RegExp(obj);
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
  
  // Handle Object
  const objCopy = {};
  hash.set(obj, objCopy);
  Object.keys(obj).forEach(key => {
    objCopy[key] = deepClone(obj[key], hash);
  });
  
  return objCopy;
}

// 3. Using structuredClone (modern browsers)
const deep = structuredClone(obj);

// Test circular reference
const circular = { name: 'obj' };
circular.self = circular;

const cloned = deepClone(circular);
console.log(cloned.self === cloned); // true
console.log(cloned === circular); // false

// 4. Advanced: Clone with functions
function deepCloneWithFunctions(obj, hash = new WeakMap()) {
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  if (hash.has(obj)) {
    return hash.get(obj);
  }
  
  if (typeof obj === 'function') {
    return obj; // Functions are copied by reference
  }
  
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  if (obj instanceof Map) {
    const mapCopy = new Map();
    hash.set(obj, mapCopy);
    obj.forEach((value, key) => {
      mapCopy.set(key, deepCloneWithFunctions(value, hash));
    });
    return mapCopy;
  }
  if (obj instanceof Set) {
    const setCopy = new Set();
    hash.set(obj, setCopy);
    obj.forEach(value => {
      setCopy.add(deepCloneWithFunctions(value, hash));
    });
    return setCopy;
  }
  
  if (Array.isArray(obj)) {
    const arrCopy = [];
    hash.set(obj, arrCopy);
    obj.forEach((item, i) => {
      arrCopy[i] = deepCloneWithFunctions(item, hash);
    });
    return arrCopy;
  }
  
  const objCopy = Object.create(Object.getPrototypeOf(obj));
  hash.set(obj, objCopy);
  
  for (const key of Reflect.ownKeys(obj)) {
    objCopy[key] = deepCloneWithFunctions(obj[key], hash);
  }
  
  return objCopy;
}
```

### Q34: Implement a retry mechanism for failed async operations.

**Expected Answer:**
```javascript
// Basic retry with fixed delay
async function retry(fn, maxAttempts = 3, delay = 1000) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxAttempts) {
        throw error;
      }
      console.log(`Attempt ${attempt} failed, retrying in ${delay}ms...`);
      await sleep(delay);
    }
  }
}

// Usage
try {
  const data = await retry(
    () => fetch('/api/data').then(r => r.json()),
    3,
    2000
  );
} catch (error) {
  console.error('All attempts failed:', error);
}

// Exponential backoff
async function retryWithBackoff(fn, maxAttempts = 3, baseDelay = 1000) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxAttempts) {
        throw error;
      }
      
      const delay = baseDelay * Math.pow(2, attempt - 1);
      console.log(`Attempt ${attempt} failed, retrying in ${delay}ms...`);
      await sleep(delay);
    }
  }
}

// With jitter (random delay to prevent thundering herd)
async function retryWithJitter(fn, maxAttempts = 3, baseDelay = 1000) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxAttempts) {
        throw error;
      }
      
      const exponentialDelay = baseDelay * Math.pow(2, attempt - 1);
      const jitter = Math.random() * baseDelay;
      const delay = exponentialDelay + jitter;
      
      console.log(`Attempt ${attempt} failed, retrying in ${delay.toFixed(0)}ms...`);
      await sleep(delay);
    }
  }
}

// Conditional retry (only for specific errors)
async function retryOnCondition(
  fn,
  shouldRetry,
  maxAttempts = 3,
  delay = 1000
) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxAttempts || !shouldRetry(error)) {
        throw error;
      }
      
      console.log(`Retryable error, attempt ${attempt}/${maxAttempts}`);
      await sleep(delay);
    }
  }
}

// Usage: Only retry on network errors
try {
  const data = await retryOnCondition(
    () => fetch('/api/data').then(r => r.json()),
    (error) => error.name === 'TypeError', // Network error
    3,
    2000
  );
} catch (error) {
  console.error('Failed:', error);
}

// Advanced: Circuit breaker pattern
class CircuitBreaker {
  constructor(fn, { threshold = 5, timeout = 60000, resetTimeout = 30000 }) {
    this.fn = fn;
    this.threshold = threshold;
    this.timeout = timeout;
    this.resetTimeout = resetTimeout;
    
    this.failures = 0;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.nextAttempt = Date.now();
  }
  
  async execute(...args) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      this.state = 'HALF_OPEN';
    }
    
    try {
      const result = await this.fn(...args);
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }
  
  onFailure() {
    this.failures++;
    if (this.failures >= this.threshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.resetTimeout;
    }
  }
}

// Usage
const breaker = new CircuitBreaker(
  () => fetch('/api/data'),
  { threshold: 3, resetTimeout: 30000 }
);

try {
  const data = await breaker.execute();
} catch (error) {
  console.error('Circuit breaker prevented call or request failed');
}
```

### Q35: Create a custom Promise-based queue system.

**Expected Answer:**
```javascript
// Task queue with concurrency control
class TaskQueue {
  constructor(concurrency = 1) {
    this.concurrency = concurrency;
    this.running = 0;
    this.queue = [];
  }
  
  async add(task) {
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