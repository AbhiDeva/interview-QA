# JavaScript Debugging Techniques in Chrome DevTools

A comprehensive guide to debugging JavaScript using Chrome DevTools.

---

## Table of Contents

1. [Console Debugging](#1-console-debugging)
2. [Breakpoints](#2-breakpoints)
3. [Sources Panel](#3-sources-panel)
4. [Network Debugging](#4-network-debugging)
5. [Performance Profiling](#5-performance-profiling)
6. [Memory Debugging](#6-memory-debugging)
7. [Advanced Techniques](#7-advanced-techniques)
8. [Useful Shortcuts](#8-useful-shortcuts)

---

## 1. Console Debugging

### 1.1 Basic Console Methods

```javascript
// Basic logging
console.log('Simple message');
console.log('Multiple', 'arguments', 123, { key: 'value' });

// String substitution
console.log('User %s is %d years old', 'John', 25);
console.log('Object: %o', { name: 'John', age: 25 });

// Different log levels
console.info('ℹ️ Information message');
console.warn('⚠️ Warning message');
console.error('❌ Error message');
console.debug('🐛 Debug message');
```

### 1.2 Styled Console Output

```javascript
// Custom styling
console.log(
  '%c Styled Text',
  'color: white; background: blue; font-size: 20px; padding: 10px;'
);

// Multiple styles
console.log(
  '%cError: %cSomething went wrong',
  'color: red; font-weight: bold;',
  'color: orange;'
);

// Create custom logger
const logger = {
  success: (msg) => console.log(`%c✓ ${msg}`, 'color: green; font-weight: bold'),
  error: (msg) => console.log(`%c✗ ${msg}`, 'color: red; font-weight: bold'),
  info: (msg) => console.log(`%cℹ ${msg}`, 'color: blue; font-weight: bold')
};

logger.success('Operation completed');
logger.error('Failed to load data');
```

### 1.3 Console Table

```javascript
// Display arrays/objects as tables
const users = [
  { id: 1, name: 'Alice', age: 28, city: 'NYC' },
  { id: 2, name: 'Bob', age: 32, city: 'LA' },
  { id: 3, name: 'Charlie', age: 25, city: 'Chicago' }
];

console.table(users);

// Display specific columns
console.table(users, ['name', 'age']);

// Objects as table
const obj = {
  name: 'John',
  age: 30,
  occupation: 'Developer'
};
console.table(obj);
```

### 1.4 Console Grouping

```javascript
// Group related logs
console.group('User Details');
console.log('Name: John Doe');
console.log('Age: 30');
console.log('Role: Admin');
console.groupEnd();

// Collapsed group
console.groupCollapsed('API Response');
console.log('Status: 200');
console.log('Data:', { users: [] });
console.groupEnd();

// Nested groups
console.group('Level 1');
console.log('Message 1');
console.group('Level 2');
console.log('Message 2');
console.groupEnd();
console.groupEnd();
```

### 1.5 Console Assertions

```javascript
// Assert conditions
const age = 15;
console.assert(age >= 18, 'User must be 18 or older');

// Assert with object
const user = { name: 'John', age: 15 };
console.assert(user.age >= 18, 'Invalid user age:', user);

// Multiple assertions
function validateUser(user) {
  console.assert(user.name, 'Name is required');
  console.assert(user.email, 'Email is required');
  console.assert(user.age > 0, 'Age must be positive');
}
```

### 1.6 Console Timing

```javascript
// Measure execution time
console.time('Data Processing');

// Simulate processing
for (let i = 0; i < 1000000; i++) {
  // Some operation
}

console.timeEnd('Data Processing');
// Output: Data Processing: 15.234ms

// Multiple timers
console.time('Database Query');
console.time('API Call');

// ... operations ...

console.timeEnd('Database Query');
console.timeEnd('API Call');

// Time log (intermediate timing)
console.time('Total Operation');
// ... some work ...
console.timeLog('Total Operation', 'Checkpoint 1');
// ... more work ...
console.timeLog('Total Operation', 'Checkpoint 2');
console.timeEnd('Total Operation');
```

### 1.7 Console Counting

```javascript
// Count function calls
function processItem(item) {
  console.count('processItem called');
  // Process item
}

processItem(1); // processItem called: 1
processItem(2); // processItem called: 2
processItem(3); // processItem called: 3

// Count with labels
function handleClick(button) {
  console.count(`Button ${button} clicked`);
}

handleClick('A'); // Button A clicked: 1
handleClick('B'); // Button B clicked: 1
handleClick('A'); // Button A clicked: 2

// Reset counter
console.countReset('processItem called');
```

### 1.8 Console Trace

```javascript
// Stack trace
function outer() {
  inner();
}

function inner() {
  deepest();
}

function deepest() {
  console.trace('Function call stack');
}

outer();
// Shows: deepest() -> inner() -> outer()
```

---

## 2. Breakpoints

### 2.1 Line-of-Code Breakpoints

**How to set:**
1. Open DevTools (F12)
2. Go to **Sources** tab
3. Click on line number in source code
4. Blue marker appears

**Keyboard shortcut:** `Ctrl+B` (Windows) or `Cmd+B` (Mac)

```javascript
function calculateTotal(items) {
  let total = 0;
  // Set breakpoint on next line
  items.forEach(item => {
    total += item.price * item.quantity;
  });
  return total;
}
```

### 2.2 Conditional Breakpoints

**How to set:**
1. Right-click on line number
2. Select "Add conditional breakpoint"
3. Enter condition

```javascript
// Example: Break only when total exceeds 100
function calculateTotal(items) {
  let total = 0;
  items.forEach(item => {
    total += item.price * item.quantity;
    // Conditional breakpoint: total > 100
  });
  return total;
}
```

**Common conditions:**
```javascript
// Break on specific value
userId === 123

// Break on specific index
i === 50

// Break on error condition
response.status !== 200

// Break on null/undefined
data === null || data === undefined

// Break on array length
arr.length > 10
```

### 2.3 Logpoints (Console.log without modifying code)

**How to set:**
1. Right-click on line number
2. Select "Add logpoint"
3. Enter message

```javascript
function processUser(user) {
  // Logpoint: User ID: {user.id}, Name: {user.name}
  const processed = doSomething(user);
  return processed;
}
```

### 2.4 DOM Breakpoints

**Types:**
- **Subtree modifications** - Child elements added/removed
- **Attribute modifications** - Attributes changed
- **Node removal** - Element removed from DOM

**How to set:**
1. Right-click element in **Elements** tab
2. Select "Break on" → Choose type

```javascript
// Example: Find what's changing an element
const element = document.getElementById('status');

// Set breakpoint in Elements tab
// Code that triggers breakpoint:
element.textContent = 'Updated'; // Breaks here
element.classList.add('active'); // Breaks here (attribute modification)
```

### 2.5 Event Listener Breakpoints

**Location:** Sources → Event Listener Breakpoints panel

**Common events to debug:**
- Mouse: click, dblclick, mousedown, mouseup
- Keyboard: keydown, keyup, keypress
- Form: submit, change, input
- Touch: touchstart, touchend, touchmove
- Timer: setInterval, setTimeout

```javascript
// Debug click events
document.getElementById('button').addEventListener('click', (e) => {
  // Execution pauses here when event listener breakpoint is active
  console.log('Button clicked');
});
```

### 2.6 Exception Breakpoints

**How to enable:**
1. Sources tab → Breakpoints panel
2. Check "Pause on caught exceptions" or "Pause on uncaught exceptions"

```javascript
try {
  // Execution pauses here if "Pause on caught exceptions" is enabled
  throw new Error('Something went wrong');
} catch (error) {
  console.error(error);
}

// Execution pauses here if "Pause on uncaught exceptions" is enabled
throw new Error('Uncaught error');
```

### 2.7 XHR/Fetch Breakpoints

**How to set:**
1. Sources → XHR/fetch Breakpoints
2. Click + and enter URL pattern

```javascript
// Break on specific API calls
fetch('/api/users')
  .then(response => response.json())
  .then(data => console.log(data));

// Pattern: /api/users
// Pauses when this fetch executes
```

---

## 3. Sources Panel

### 3.1 Debugger Statement

```javascript
function calculatePrice(items) {
  let total = 0;
  
  // Programmatic breakpoint
  debugger;
  
  items.forEach(item => {
    total += item.price * item.quantity;
  });
  
  return total;
}
```

### 3.2 Call Stack Navigation

**What it shows:** Function call hierarchy

```javascript
function level1() {
  level2();
}

function level2() {
  level3();
}

function level3() {
  debugger; // Pause here
  // Call stack shows: level3 → level2 → level1
}

level1();
```

**Navigation:**
- Click on function in call stack to jump to that frame
- View local variables for each frame

### 3.3 Scope Variables

**Panels available:**
- **Local** - Variables in current function
- **Closure** - Variables from outer scopes
- **Global** - Window object

```javascript
const globalVar = 'I am global';

function outer(outerParam) {
  const outerVar = 'I am outer';
  
  function inner(innerParam) {
    const innerVar = 'I am inner';
    debugger;
    
    // Scope shows:
    // Local: innerParam, innerVar
    // Closure: outerParam, outerVar
    // Global: globalVar, window, document, etc.
  }
  
  inner('inner value');
}

outer('outer value');
```

### 3.4 Watch Expressions

**How to use:**
1. Sources → Watch panel
2. Click + and enter expression
3. Updates automatically during debugging

```javascript
function processOrder(order) {
  const tax = order.subtotal * 0.1;
  const shipping = order.weight * 2;
  const total = order.subtotal + tax + shipping;
  
  debugger;
  
  // Add to Watch:
  // - order.subtotal
  // - tax
  // - total
  // - total > 100
}
```

### 3.5 Step Controls

**Controls:**
- **Resume (F8)** - Continue execution
- **Step Over (F10)** - Execute current line, don't enter functions
- **Step Into (F11)** - Enter function call
- **Step Out (Shift+F11)** - Exit current function
- **Step (F9)** - Same as Step Into

```javascript
function main() {
  const data = getData(); // F10: Execute without entering
  processData(data);      // F11: Enter this function
  saveData(data);         // Shift+F11 from inside: Return here
  // F8: Continue to next breakpoint
}

function processData(data) {
  // Stepped into this function
  return data.map(item => item * 2);
}
```

### 3.6 Blackboxing Scripts

**Purpose:** Skip library code during debugging

**How to blackbox:**
1. Right-click on script in Call Stack
2. Select "Blackbox script"

```javascript
// Your code
function myFunction() {
  debugger;
  $(document).ready(() => { // Won't step into jQuery
    // Your code
  });
}
```

**Settings:**
- Settings → Blackboxing
- Add patterns: `node_modules`, `jquery*.js`, etc.

---

## 4. Network Debugging

### 4.1 Network Panel Overview

**Key columns:**
- Name
- Status
- Type
- Initiator
- Size
- Time
- Waterfall

**Filter by type:**
- XHR
- JS
- CSS
- Img
- Media
- Font
- Doc
- WS (WebSocket)

### 4.2 Request Inspection

```javascript
// Example fetch request
fetch('/api/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123'
  },
  body: JSON.stringify({ name: 'John' })
})
.then(response => response.json())
.then(data => console.log(data));
```

**Inspect in Network tab:**
1. Headers tab: Request/response headers
2. Preview tab: Formatted response
3. Response tab: Raw response
4. Timing tab: Request breakdown
5. Initiator tab: What triggered request

### 4.3 Copy as Fetch/cURL

**How to:**
1. Right-click on request
2. Copy → Copy as fetch / Copy as cURL

```javascript
// Copy as fetch generates:
fetch("https://api.example.com/users", {
  "headers": {
    "accept": "application/json",
    "authorization": "Bearer token123"
  },
  "method": "GET"
});
```

### 4.4 Throttling Network

**Presets:**
- Slow 3G
- Fast 3G
- Offline

**Custom profile:**
1. Network tab → Throttling dropdown
2. Add custom profile
3. Set download/upload speeds

### 4.5 Block Request URL

**How to:**
1. Right-click request
2. Block request URL
3. Reload page

**Use case:** Test error handling when API fails

---

## 5. Performance Profiling

### 5.1 Performance.measure API

```javascript
// Mark points in time
performance.mark('start-data-fetch');

fetch('/api/data')
  .then(response => response.json())
  .then(data => {
    performance.mark('end-data-fetch');
    
    // Measure between marks
    performance.measure(
      'data-fetch-duration',
      'start-data-fetch',
      'end-data-fetch'
    );
    
    // Get measurements
    const measures = performance.getEntriesByName('data-fetch-duration');
    console.log(`Fetch took: ${measures[0].duration}ms`);
  });
```

### 5.2 Performance Panel Recording

**How to record:**
1. Performance tab
2. Click Record (●)
3. Perform actions
4. Stop recording

**What to analyze:**
- FPS (Frames Per Second)
- CPU usage
- Network activity
- Screenshots

### 5.3 Console.profile()

```javascript
// Start profiling
console.profile('My Profile');

// Code to profile
for (let i = 0; i < 1000000; i++) {
  // Some operation
}

// Stop profiling
console.profileEnd('My Profile');

// View in Performance → Profiler tab
```

### 5.4 Performance Observer

```javascript
// Observe long tasks
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log('Long task detected:', entry);
  }
});

observer.observe({ entryTypes: ['longtask'] });

// Observe navigation timing
const navObserver = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log('Navigation:', entry.toJSON());
  }
});

navObserver.observe({ entryTypes: ['navigation'] });
```

---

## 6. Memory Debugging

### 6.1 Heap Snapshots

**How to take snapshot:**
1. Memory tab
2. Select "Heap snapshot"
3. Click "Take snapshot"

**What to look for:**
- Detached DOM nodes
- Large objects
- Memory leaks

```javascript
// Example memory leak
let leakedData = [];

function createLeak() {
  const bigArray = new Array(1000000).fill('data');
  leakedData.push(bigArray); // Never released
}

// Take snapshot before
// Call createLeak() multiple times
// Take snapshot after
// Compare snapshots to find leak
```

### 6.2 Allocation Timeline

**How to use:**
1. Memory → Allocation instrumentation on timeline
2. Start recording
3. Perform actions
4. Stop recording

**Identifies:**
- Objects created over time
- Memory allocation patterns
- Garbage collection events

### 6.3 Allocation Sampling

**Purpose:** Find memory allocations with minimal overhead

```javascript
function processLargeData() {
  const data = [];
  
  for (let i = 0; i < 100000; i++) {
    data.push({
      id: i,
      name: `Item ${i}`,
      description: 'Lorem ipsum '.repeat(10)
    });
  }
  
  return data;
}

// Profile with Allocation sampling to see where memory is allocated
```

### 6.4 Memory Leak Detection

**Common patterns:**

```javascript
// BAD: Event listener leak
function badExample() {
  const element = document.getElementById('button');
  element.addEventListener('click', function handler() {
    console.log('Clicked');
    // Handler never removed
  });
}

// GOOD: Proper cleanup
function goodExample() {
  const element = document.getElementById('button');
  
  function handler() {
    console.log('Clicked');
  }
  
  element.addEventListener('click', handler);
  
  // Cleanup function
  return () => element.removeEventListener('click', handler);
}

// BAD: Timer leak
function badTimer() {
  setInterval(() => {
    console.log('Running forever');
  }, 1000);
}

// GOOD: Clear timer
function goodTimer() {
  const timer = setInterval(() => {
    console.log('Running');
  }, 1000);
  
  return () => clearInterval(timer);
}

// BAD: Closure leak
function createClosure() {
  const largeData = new Array(1000000).fill('data');
  
  return function() {
    console.log(largeData.length); // Keeps largeData in memory
  };
}

// GOOD: Release reference
function createClosure() {
  const largeData = new Array(1000000).fill('data');
  const length = largeData.length;
  
  return function() {
    console.log(length); // Only keeps length, not entire array
  };
}
```

---

## 7. Advanced Techniques

### 7.1 Command Line API

**Available only in DevTools Console:**

```javascript
// $ - document.querySelector shorthand
$('#myId')        // Same as document.getElementById('myId')
$('.myClass')     // Same as document.querySelector('.myClass')

// $$ - document.querySelectorAll shorthand
$$('div')         // Same as document.querySelectorAll('div')
$$('.item')       // Returns array, not NodeList

// $_ - Last evaluated expression
2 + 2             // 4
$_                // 4
$_ * 2            // 8

// $0-$4 - Recently inspected elements
// $0 = most recent, $4 = 5th most recent
$0.classList.add('highlight')

// dir() - Display object properties
dir(document.body)

// keys() / values()
const obj = { a: 1, b: 2, c: 3 };
keys(obj)         // ['a', 'b', 'c']
values(obj)       // [1, 2, 3]

// copy() - Copy to clipboard
copy(document.body.innerHTML)

// monitor() / unmonitor() - Monitor function calls
function myFunc(a, b) {
  return a + b;
}
monitor(myFunc)
myFunc(1, 2)      // Console shows: function myFunc called with arguments: 1, 2
unmonitor(myFunc)

// monitorEvents() / unmonitorEvents()
monitorEvents(document.body, 'click')
// Click anywhere to see events in console
unmonitorEvents(document.body)

// getEventListeners()
getEventListeners(document.getElementById('button'))
```

### 7.2 Live Expressions

**How to add:**
1. Console → Click eye icon
2. Enter expression
3. Updates in real-time

```javascript
// Useful live expressions:
document.body.scrollTop
window.innerWidth + 'x' + window.innerHeight
new Date().toLocaleTimeString()
performance.memory.usedJSHeapSize
navigator.onLine
```

### 7.3 Snippets

**How to create:**
1. Sources → Snippets tab
2. New snippet
3. Write code
4. Ctrl+Enter to run

**Example snippets:**

```javascript
// Snippet 1: Clear all localStorage
(function() {
  localStorage.clear();
  sessionStorage.clear();
  console.log('Storage cleared!');
})();

// Snippet 2: List all global variables
(function() {
  const globals = Object.keys(window).filter(key => {
    return !key.includes('webkit') && !key.includes('moz');
  });
  console.table(globals);
})();

// Snippet 3: Performance metrics
(function() {
  const perf = performance.getEntriesByType('navigation')[0];
  console.table({
    'DNS Lookup': perf.domainLookupEnd - perf.domainLookupStart,
    'TCP Connection': perf.connectEnd - perf.connectStart,
    'Request Time': perf.responseStart - perf.requestStart,
    'Response Time': perf.responseEnd - perf.responseStart,
    'DOM Processing': perf.domComplete - perf.domLoading,
    'Total Load Time': perf.loadEventEnd - perf.fetchStart
  });
})();

// Snippet 4: Find memory leaks (detached DOM)
(function() {
  const snapshot = performance.memory;
  console.table({
    'Total JS Heap': (snapshot.totalJSHeapSize / 1048576).toFixed(2) + ' MB',
    'Used JS Heap': (snapshot.usedJSHeapSize / 1048576).toFixed(2) + ' MB',
    'JS Heap Limit': (snapshot.jsHeapSizeLimit / 1048576).toFixed(2) + ' MB'
  });
})();
```

### 7.4 Local Overrides

**Purpose:** Edit files and persist changes across page loads

**How to set up:**
1. Sources → Overrides tab
2. Select folder
3. Allow access
4. Edit any file in Sources
5. Changes persist

**Use cases:**
- Test CSS changes
- Debug production code
- Test API response modifications

### 7.5 Workspace

**Purpose:** Edit files directly in DevTools and save to filesystem

**How to set up:**
1. Sources → Filesystem tab
2. Add folder to workspace
3. Allow access
4. Edit files in DevTools = Edit actual files

### 7.6 Remote Debugging

**For mobile devices:**

**Android:**
1. Enable USB debugging on device
2. Connect via USB
3. Chrome → chrome://inspect
4. Select device

**iOS (Safari):**
1. Enable Web Inspector on device
2. Connect via USB
3. Safari → Develop → [Device Name]

---

## 8. Useful Shortcuts

### 8.1 General Shortcuts

| Action | Windows/Linux | Mac |
|--------|---------------|-----|
| Open DevTools | F12 or Ctrl+Shift+I | Cmd+Option+I |
| Open Console | Ctrl+Shift+J | Cmd+Option+J |
| Open Elements | Ctrl+Shift+C | Cmd+Shift+C |
| Command Menu | Ctrl+Shift+P | Cmd+Shift+P |
| Next Panel | Ctrl+] | Cmd+] |
| Previous Panel | Ctrl+[ | Cmd+[ |

### 8.2 Sources Panel Shortcuts

| Action | Windows/Linux | Mac |
|--------|---------------|-----|
| Resume/Pause | F8 | F8 |
| Step Over | F10 | F10 |
| Step Into | F11 | F11 |
| Step Out | Shift+F11 | Shift+F11 |
| Toggle Breakpoint | Ctrl+B | Cmd+B |
| Run Snippet | Ctrl+Enter | Cmd+Enter |

### 8.3 Console Shortcuts

| Action | Windows/Linux | Mac |
|--------|---------------|-----|
| Clear Console | Ctrl+L | Cmd+K |
| Focus Console | Ctrl+` | Ctrl+` |
| Multi-line Entry | Shift+Enter | Shift+Enter |
| Previous Command | Up Arrow | Up Arrow |
| Search in Console | Ctrl+F | Cmd+F |

### 8.4 Elements Panel Shortcuts

| Action | Windows/Linux | Mac |
|--------|---------------|-----|
| Edit as HTML | F2 | F2 |
| Hide Element | H | H |
| Toggle Class | .cls button | .cls button |
| Undo | Ctrl+Z | Cmd+Z |
| Redo | Ctrl+Y | Cmd+Shift+Z |

---

## 9. Debugging Patterns & Best Practices

### 9.1 Defensive Logging

```javascript
function processData(data) {
  console.log('processData called with:', data);
  
  if (!data) {
    console.error('Data is null or undefined');
    return null;
  }
  
  console.log('Data validation passed');
  
  try {
    const result = transform(data);
    console.log('Transform result:', result);
    return result;
  } catch (error) {
    console.error('Transform failed:', error);
    throw error;
  }
}
```

### 9.2 Debug Wrapper Functions

```javascript
// Create debug wrapper for any function
function debugWrapper(fn, name) {
  return function(...args) {
    console.group(`Calling ${name}`);
    console.log('Arguments:', args);
    console.time(name);
    
    try {
      const result = fn.apply(this, args);
      console.log('Result:', result);
      return result;
    } catch (error) {
      console.error('Error:', error);
      throw error;
    } finally {
      console.timeEnd(name);
      console.groupEnd();
    }
  };
}

// Usage
const debuggedFetch = debugWrapper(fetch, 'fetch');
debuggedFetch('/api/data');
```

### 9.3 Conditional Debugging

```javascript
// Enable debugging based on environment
const DEBUG = localStorage.getItem('debug') === 'true';

function debug(...args) {
  if (DEBUG) {
    console.log('[DEBUG]', ...args);
  }
}

// Enable: localStorage.setItem('debug', 'true')
// Disable: localStorage.removeItem('debug')

debug('This only logs when debugging is enabled');
```

### 9.4 Request Logging Interceptor

```javascript
// Intercept all fetch requests
const originalFetch = window.fetch;

window.fetch = function(...args) {
  console.group('Fetch Request');
  console.log('URL:', args[0]);
  console.log('Options:', args[1]);
  console.time('Request Duration');
  
  return originalFetch.apply(this, args)
    .then(response => {
      console.log('Status:', response.status);
      console.timeEnd('Request Duration');
      console.groupEnd();
      return response;
    })
    .catch(error => {
      console.error('Fetch Error:', error);
      console.groupEnd();
      throw error;
    });
};
```

---

## 10. Common Debugging Scenarios

### 10.1 Async/Await Debugging

```javascript
async function fetchUserData(userId) {
  try {
    debugger; // Pause before fetch
    
    const response = await fetch(`/api/users/${userId}`);
    
    debugger; // Pause after fetch
    
    const data = await response.json();
    
    debugger; // Pause after parsing
    
    return data;
  } catch (error) {
    console.error('Error fetching user:', error);
    debugger; // Pause on error
  }
}
```

### 10.2 Event Handling Debugging

```javascript
// Debug event propagation
document.addEventListener('click', (e) => {
  console.log('Document click');
  console.log('Target:', e.target);
  console.log('CurrentTarget:', e.currentTarget);
  console.log('Phase:', e.eventPhase);
}, true); // Capture phase

document.getElementById('button').addEventListener('click', (e) => {
  debugger;
  console.log('Button click');
  // e.stopPropagation(); // Test propagation
});
```

### 10.3 Promise Chain Debugging

```javascript
fetch('/api/data')
  .then(response => {
    debugger; // Check response
    return response.json();
  })
  .then(data => {
    debugger; // Check parsed data
    return processData(data);
  })
  .then(result => {
    debugger; // Check final result
    console.log(result);
  })
  .catch(error => {
    debugger; // Debug errors
    console.error(error);
  });
```

---

## Summary

**Quick Debug Checklist:**

1. ✅ Use `console.log()` for quick checks
2. ✅ Use `debugger` or breakpoints for step-through debugging
3. ✅ Use conditional breakpoints to isolate specific scenarios
4. ✅ Use Network tab to debug API calls
5. ✅ Use Performance tab for slow operations
6. ✅ Use Memory tab for memory leaks
7. ✅ Use Command Line API for quick DOM/object inspection
8. ✅ Create snippets for repetitive debug tasks
9. ✅ Use source maps for production debugging
10. ✅ Master keyboard shortcuts for efficiency

**Remember:** The best debugging technique is the one that helps you find the bug fastest!