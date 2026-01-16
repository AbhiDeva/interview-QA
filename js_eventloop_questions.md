# 25 JavaScript Event Loop Interview Questions with Code

## Question 1: Basic Event Loop - What will be the output?

**Code:**
```javascript
console.log('Start');

setTimeout(() => {
  console.log('Timeout');
}, 0);

console.log('End');
```

**Answer:**
```
Start
End
Timeout
```

**Explanation:**
- `console.log('Start')` executes immediately (synchronous)
- `setTimeout` is sent to Web APIs, callback goes to Task Queue
- `console.log('End')` executes immediately
- Event loop checks: call stack is empty, moves setTimeout callback to call stack
- `console.log('Timeout')` executes

---

## Question 2: Promises vs setTimeout - What's the order?

**Code:**
```javascript
console.log('1');

setTimeout(() => {
  console.log('2');
}, 0);

Promise.resolve().then(() => {
  console.log('3');
});

console.log('4');
```

**Answer:**
```
1
4
3
2
```

**Explanation:**
- Synchronous code executes first: 1, 4
- Promises use the **Microtask Queue** (higher priority)
- setTimeout uses the **Macrotask Queue** (lower priority)
- Microtasks execute before macrotasks
- Order: 3 (promise), then 2 (setTimeout)

---

## Question 3: Multiple Promises and setTimeout

**Code:**
```javascript
setTimeout(() => console.log('1'), 0);

Promise.resolve()
  .then(() => console.log('2'))
  .then(() => console.log('3'));

Promise.resolve()
  .then(() => console.log('4'));

setTimeout(() => console.log('5'), 0);

console.log('6');
```

**Answer:**
```
6
2
4
3
1
5
```

**Explanation:**
- Synchronous: 6
- Microtasks (all promises): 2, 4, 3
- Macrotasks (setTimeouts): 1, 5

---

## Question 4: Nested setTimeout

**Code:**
```javascript
console.log('Start');

setTimeout(() => {
  console.log('Timeout 1');
  
  setTimeout(() => {
    console.log('Timeout 2');
  }, 0);
  
  console.log('Timeout 1 End');
}, 0);

console.log('End');
```

**Answer:**
```
Start
End
Timeout 1
Timeout 1 End
Timeout 2
```

**Explanation:**
- Synchronous code first
- First setTimeout callback executes
- Nested setTimeout is queued
- 'Timeout 1 End' executes (synchronous in callback)
- Nested setTimeout callback executes

---

## Question 5: Promise in setTimeout

**Code:**
```javascript
setTimeout(() => {
  console.log('1');
  Promise.resolve().then(() => console.log('2'));
}, 0);

setTimeout(() => {
  console.log('3');
}, 0);

Promise.resolve().then(() => console.log('4'));
```

**Answer:**
```
4
1
2
3
```

**Explanation:**
- First promise (4) executes - microtask queue
- First setTimeout runs: logs 1, queues promise (2)
- Promise (2) executes before next setTimeout
- Second setTimeout logs 3

---

## Question 6: async/await with setTimeout

**Code:**
```javascript
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}

async function async2() {
  console.log('async2');
}

console.log('script start');

setTimeout(() => {
  console.log('setTimeout');
}, 0);

async1();

Promise.resolve().then(() => {
  console.log('promise1');
});

console.log('script end');
```

**Answer:**
```
script start
async1 start
async2
script end
async1 end
promise1
setTimeout
```

**Explanation:**
- Synchronous: script start, async1 start, async2, script end
- await creates a microtask for code after it
- Microtasks: async1 end, promise1
- Macrotask: setTimeout

---

## Question 7: setImmediate vs setTimeout (Node.js)

**Code:**
```javascript
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
```

**Answer:**
```
Output can vary, but typically:
immediate
timeout
```

**Explanation:**
- In Node.js, `setImmediate` is designed to execute after I/O events
- `setTimeout(fn, 0)` has a minimum delay of ~1ms
- Order can vary depending on timing

---

## Question 8: Microtask vs Macrotask Queue

**Code:**
```javascript
console.log('1');

setTimeout(() => {
  console.log('2');
  Promise.resolve().then(() => console.log('3'));
}, 0);

Promise.resolve()
  .then(() => console.log('4'))
  .then(() => console.log('5'));

console.log('6');
```

**Answer:**
```
1
6
4
5
2
3
```

**Explanation:**
- Synchronous: 1, 6
- All microtasks complete: 4, 5
- Macrotask starts: 2
- Microtask from inside macrotask: 3

---

## Question 9: Multiple Promise Chains

**Code:**
```javascript
Promise.resolve()
  .then(() => {
    console.log('1');
    return Promise.resolve('2');
  })
  .then((res) => console.log(res));

Promise.resolve()
  .then(() => {
    console.log('3');
  })
  .then(() => {
    console.log('4');
  });

console.log('5');
```

**Answer:**
```
5
1
3
4
2
```

**Explanation:**
- Synchronous: 5
- First .then(): 1
- Second promise chain .then(): 3
- Second promise chain .then(): 4
- First chain continues: 2 (returned promise adds extra microtask)

---

## Question 10: Process.nextTick (Node.js)

**Code:**
```javascript
console.log('start');

setTimeout(() => console.log('timeout'), 0);

Promise.resolve().then(() => console.log('promise'));

process.nextTick(() => console.log('nextTick'));

console.log('end');
```

**Answer:**
```
start
end
nextTick
promise
timeout
```

**Explanation:**
- `process.nextTick` has higher priority than promises
- Order: Synchronous → nextTick → Microtasks → Macrotasks

---

## Question 11: Chained setTimeout

**Code:**
```javascript
setTimeout(() => {
  console.log('A');
  setTimeout(() => console.log('B'), 0);
}, 100);

setTimeout(() => {
  console.log('C');
}, 50);

setTimeout(() => {
  console.log('D');
}, 0);
```

**Answer:**
```
D
C
A
B
```

**Explanation:**
- Timers execute based on delay
- D (0ms), C (50ms), A (100ms), B (100ms + 0ms)

---

## Question 12: Promise.all with setTimeout

**Code:**
```javascript
console.log('start');

const promise1 = new Promise((resolve) => {
  setTimeout(() => {
    console.log('promise1');
    resolve('1');
  }, 100);
});

const promise2 = new Promise((resolve) => {
  setTimeout(() => {
    console.log('promise2');
    resolve('2');
  }, 50);
});

Promise.all([promise1, promise2]).then((values) => {
  console.log('all:', values);
});

console.log('end');
```

**Answer:**
```
start
end
promise2
promise1
all: ['1', '2']
```

**Explanation:**
- Synchronous code first
- promise2 resolves at 50ms
- promise1 resolves at 100ms
- Promise.all waits for both, then executes

---

## Question 13: Async Function Return Value

**Code:**
```javascript
async function getData() {
  return 'data';
}

console.log('1');

getData().then(result => console.log(result));

console.log('2');
```

**Answer:**
```
1
2
data
```

**Explanation:**
- async functions always return a Promise
- .then() callback is a microtask
- Executes after synchronous code

---

## Question 14: await in Loop

**Code:**
```javascript
async function processArray() {
  console.log('start');
  
  const arr = [1, 2, 3];
  
  for (const num of arr) {
    await new Promise(resolve => {
      setTimeout(() => {
        console.log(num);
        resolve();
      }, 100);
    });
  }
  
  console.log('end');
}

processArray();
console.log('after call');
```

**Answer:**
```
start
after call
1 (after 100ms)
2 (after 200ms)
3 (after 300ms)
end
```

**Explanation:**
- Function starts, logs 'start'
- await pauses execution, control returns
- 'after call' logs
- Each iteration waits 100ms

---

## Question 15: Promise Constructor Execution

**Code:**
```javascript
console.log('1');

new Promise((resolve) => {
  console.log('2');
  resolve();
  console.log('3');
}).then(() => {
  console.log('4');
});

console.log('5');
```

**Answer:**
```
1
2
3
5
4
```

**Explanation:**
- Promise executor function runs synchronously
- Code after resolve() still executes
- .then() callback is a microtask

---

## Question 16: Multiple async/await

**Code:**
```javascript
async function func1() {
  console.log('A');
  await func2();
  console.log('B');
}

async function func2() {
  console.log('C');
}

console.log('1');
func1();
console.log('2');
```

**Answer:**
```
1
A
C
2
B
```

**Explanation:**
- Synchronous until await
- await suspends execution
- Remaining sync code runs
- Microtask completes

---

## Question 17: setTimeout with Promises in Loop

**Code:**
```javascript
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log('setTimeout:', i);
  }, 0);
  
  Promise.resolve().then(() => {
    console.log('Promise:', i);
  });
}

console.log('Done');
```

**Answer:**
```
Done
Promise: 0
Promise: 1
Promise: 2
setTimeout: 0
setTimeout: 1
setTimeout: 2
```

**Explanation:**
- Loop executes synchronously, queuing callbacks
- Synchronous: Done
- All microtasks (Promises): 0, 1, 2
- All macrotasks (setTimeouts): 0, 1, 2

---

## Question 18: Nested Promises

**Code:**
```javascript
Promise.resolve()
  .then(() => {
    console.log('1');
    Promise.resolve().then(() => {
      console.log('2');
    });
    console.log('3');
  })
  .then(() => {
    console.log('4');
  });
```

**Answer:**
```
1
3
2
4
```

**Explanation:**
- First .then() executes: logs 1
- Nested promise queued
- Synchronous in first .then(): 3
- Second .then() queued
- Nested promise: 2
- Second .then(): 4

---

## Question 19: MutationObserver vs Promise

**Code:**
```javascript
console.log('start');

setTimeout(() => console.log('timeout'), 0);

Promise.resolve().then(() => console.log('promise'));

const observer = new MutationObserver(() => {
  console.log('mutation');
});

const div = document.createElement('div');
observer.observe(div, { attributes: true });
div.setAttribute('data-test', 'value');

console.log('end');
```

**Answer:**
```
start
end
promise
mutation
timeout
```

**Explanation:**
- MutationObserver callbacks are microtasks
- Execute in order with promises
- Before macrotasks (setTimeout)

---

## Question 20: Error Handling in Event Loop

**Code:**
```javascript
console.log('1');

setTimeout(() => {
  console.log('2');
  throw new Error('Error in setTimeout');
}, 0);

Promise.resolve()
  .then(() => {
    console.log('3');
    throw new Error('Error in Promise');
  })
  .catch(err => console.log('Caught:', err.message));

console.log('4');
```

**Answer:**
```
1
4
3
Caught: Error in Promise
2
Uncaught Error: Error in setTimeout
```

**Explanation:**
- Promise errors can be caught with .catch()
- setTimeout errors are uncaught (different execution context)
- Each task has its own error handling

---

## Question 21: Async/Await Error Handling

**Code:**
```javascript
async function test() {
  console.log('1');
  
  try {
    await Promise.reject('Error');
    console.log('2'); // Won't execute
  } catch (err) {
    console.log('Caught:', err);
  }
  
  console.log('3');
}

console.log('Start');
test();
console.log('End');
```

**Answer:**
```
Start
1
End
Caught: Error
3
```

**Explanation:**
- Function starts synchronously
- await creates microtask
- 'End' logs (sync code continues)
- Error caught, execution resumes
- '3' logs

---

## Question 22: RequestAnimationFrame

**Code:**
```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

requestAnimationFrame(() => console.log('3'));

Promise.resolve().then(() => console.log('4'));

console.log('5');
```

**Answer:**
```
1
5
4
3
2
```

**Explanation:**
- Synchronous: 1, 5
- Microtasks: 4
- rAF runs before next paint (before macrotasks)
- Macrotasks: 2

---

## Question 23: Immediate Promise Resolution

**Code:**
```javascript
const promise = Promise.resolve();

promise.then(() => console.log('1'));
console.log('2');
promise.then(() => console.log('3'));
console.log('4');
```

**Answer:**
```
2
4
1
3
```

**Explanation:**
- Promise is already resolved
- Both .then() callbacks are queued as microtasks
- Synchronous code executes first
- Microtasks execute in order

---

## Question 24: Complex Event Loop Scenario

**Code:**
```javascript
console.log('Script start');

setTimeout(() => {
  console.log('setTimeout 1');
  Promise.resolve().then(() => {
    console.log('Promise in setTimeout 1');
  });
}, 0);

Promise.resolve()
  .then(() => {
    console.log('Promise 1');
    setTimeout(() => {
      console.log('setTimeout in Promise 1');
    }, 0);
  })
  .then(() => {
    console.log('Promise 2');
  });

setTimeout(() => {
  console.log('setTimeout 2');
}, 0);

console.log('Script end');
```

**Answer:**
```
Script start
Script end
Promise 1
Promise 2
setTimeout 1
Promise in setTimeout 1
setTimeout 2
setTimeout in Promise 1
```

**Explanation:**
1. Synchronous: Script start, Script end
2. Microtasks: Promise 1, Promise 2
3. Macrotask: setTimeout 1
4. Microtask from setTimeout 1: Promise in setTimeout 1
5. Macrotask: setTimeout 2
6. Macrotask: setTimeout in Promise 1

---

## Question 25: The Ultimate Event Loop Challenge

**Code:**
```javascript
console.log('1');

async function async1() {
  console.log('2');
  await async2();
  console.log('3');
}

async function async2() {
  console.log('4');
}

setTimeout(() => {
  console.log('5');
  new Promise(resolve => {
    console.log('6');
    resolve();
  }).then(() => {
    console.log('7');
  });
}, 0);

const promise = new Promise(resolve => {
  console.log('8');
  resolve();
});

promise.then(() => {
  console.log('9');
});

async1();

console.log('10');

promise.then(() => {
  console.log('11');
});
```

**Answer:**
```
1
8
2
4
10
9
3
11
5
6
7
```

**Explanation:**
**Synchronous execution:**
1. Log '1'
2. Promise executor: '8'
3. async1 called: '2'
4. async2 called: '4' (await suspends async1)
5. Log '10'

**Microtasks (first batch):**
6. First promise.then(): '9'
7. Code after await: '3'
8. Second promise.then(): '11'

**Macrotask (setTimeout):**
9. Log '5'
10. Promise executor (sync): '6'
11. Promise .then() (microtask): '7'

---

## Key Concepts Summary

### Execution Order:
1. **Synchronous Code** - Executes immediately
2. **Microtasks** - Promises, async/await, process.nextTick, MutationObserver
3. **Macrotasks** - setTimeout, setInterval, setImmediate, I/O operations

### Important Rules:
- All microtasks execute before any macrotask
- Microtasks can queue more microtasks
- Promise executors run synchronously
- async functions return Promises
- await suspends async function execution
- Each macrotask can queue new microtasks

### Common Pitfalls:
- setTimeout(fn, 0) is not immediate
- Promise constructor is synchronous
- Code after resolve() still executes
- await doesn't block other code
- Microtask queue must be empty before next macrotask

---

**End of Event Loop Questions**