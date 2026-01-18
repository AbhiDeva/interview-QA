# JavaScript Coding Styles for Large Dataset Arrays

A comprehensive guide to efficiently handling large arrays with different coding approaches.

---

## 1. Sorting Large Arrays

### Style 1: Built-in Sort (Simple)

```javascript
// Basic numeric sort
const numbers = [64, 34, 25, 12, 22, 11, 90];
const sorted = [...numbers].sort((a, b) => a - b);

console.log(sorted); // [11, 12, 22, 25, 34, 64, 90]
```

### Style 2: Custom Comparator (Complex Objects)

```javascript
const users = [
  { id: 3, name: 'Charlie', age: 35, salary: 75000 },
  { id: 1, name: 'Alice', age: 28, salary: 65000 },
  { id: 2, name: 'Bob', age: 32, salary: 70000 }
];

// Sort by multiple criteria
const sortedUsers = [...users].sort((a, b) => {
  // Primary: age descending
  if (b.age !== a.age) return b.age - a.age;
  // Secondary: name ascending
  return a.name.localeCompare(b.name);
});

console.log(sortedUsers);
// [Charlie(35), Bob(32), Alice(28)]
```

### Style 3: Optimized Sort for Large Datasets

```javascript
// For very large arrays, use typed arrays when possible
function efficientSort(largeArray) {
  // Convert to typed array if all numbers
  const typedArray = new Float64Array(largeArray);
  
  // Typed arrays sort faster
  typedArray.sort();
  
  return Array.from(typedArray);
}

// Quick sort implementation for better control
function quickSort(arr, left = 0, right = arr.length - 1) {
  if (left < right) {
    const pivotIndex = partition(arr, left, right);
    quickSort(arr, left, pivotIndex - 1);
    quickSort(arr, pivotIndex + 1, right);
  }
  return arr;
}

function partition(arr, left, right) {
  const pivot = arr[right];
  let i = left - 1;
  
  for (let j = left; j < right; j++) {
    if (arr[j] < pivot) {
      i++;
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
  }
  
  [arr[i + 1], arr[right]] = [arr[right], arr[i + 1]];
  return i + 1;
}
```

---

## 2. Filtering Large Arrays

### Style 1: Traditional Filter

```javascript
const numbers = Array.from({ length: 1000000 }, (_, i) => i);

// Simple filter
const evens = numbers.filter(n => n % 2 === 0);

// Multiple conditions
const filtered = users.filter(user => 
  user.age > 25 && user.salary > 60000
);
```

### Style 2: Optimized Filter with Early Exit

```javascript
// For finding first N matches (more efficient)
function filterWithLimit(arr, predicate, limit) {
  const result = [];
  
  for (let i = 0; i < arr.length && result.length < limit; i++) {
    if (predicate(arr[i])) {
      result.push(arr[i]);
    }
  }
  
  return result;
}

// Usage
const first100Evens = filterWithLimit(
  numbers, 
  n => n % 2 === 0, 
  100
);
```

### Style 3: Parallel Filtering (for very large datasets)

```javascript
function parallelFilter(arr, predicate, chunkSize = 10000) {
  const chunks = [];
  
  for (let i = 0; i < arr.length; i += chunkSize) {
    chunks.push(arr.slice(i, i + chunkSize));
  }
  
  // Process chunks (can be parallelized with Web Workers)
  return chunks
    .map(chunk => chunk.filter(predicate))
    .flat();
}
```

---

## 3. Searching in Large Arrays

### Style 1: Linear Search (Unsorted)

```javascript
function linearSearch(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) return i;
  }
  return -1;
}

// Built-in methods
const index = arr.indexOf(target);
const found = arr.find(item => item.id === 5);
const foundIndex = arr.findIndex(item => item.id === 5);
```

### Style 2: Binary Search (Sorted Arrays)

```javascript
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    
    if (arr[mid] === target) return mid;
    if (arr[mid] < target) left = mid + 1;
    else right = mid - 1;
  }
  
  return -1;
}

// Usage
const sortedNumbers = [1, 3, 5, 7, 9, 11, 13, 15];
const position = binarySearch(sortedNumbers, 7); // Returns 3
```

### Style 3: Hash Map Search (O(1) lookup)

```javascript
// Best for multiple searches on same dataset
function createSearchIndex(arr, keyField = 'id') {
  const map = new Map();
  
  arr.forEach((item, index) => {
    const key = typeof item === 'object' ? item[keyField] : item;
    map.set(key, { item, index });
  });
  
  return map;
}

// Usage
const users = [
  { id: 101, name: 'Alice' },
  { id: 102, name: 'Bob' },
  { id: 103, name: 'Charlie' }
];

const userIndex = createSearchIndex(users, 'id');

// O(1) lookup
const user = userIndex.get(102); // { item: {...}, index: 1 }
console.log(user.item.name); // 'Bob'
```

### Style 4: Advanced Multi-field Search

```javascript
function multiFieldSearch(arr, searchTerms) {
  return arr.filter(item => {
    return Object.entries(searchTerms).every(([key, value]) => {
      if (typeof value === 'string') {
        return item[key]?.toString().toLowerCase().includes(value.toLowerCase());
      }
      return item[key] === value;
    });
  });
}

// Usage
const results = multiFieldSearch(users, {
  age: 28,
  name: 'ali'
});
```

---

## 4. Identifying Duplicates

### Style 1: Using Set (Simple Primitives)

```javascript
const numbers = [1, 2, 3, 2, 4, 5, 1, 6, 3];

// Find duplicates
function findDuplicates(arr) {
  const seen = new Set();
  const duplicates = new Set();
  
  arr.forEach(item => {
    if (seen.has(item)) {
      duplicates.add(item);
    } else {
      seen.add(item);
    }
  });
  
  return Array.from(duplicates);
}

console.log(findDuplicates(numbers)); // [2, 1, 3]
```

### Style 2: Frequency Counter

```javascript
function getDuplicatesWithCount(arr) {
  const frequency = new Map();
  
  // Count occurrences
  arr.forEach(item => {
    frequency.set(item, (frequency.get(item) || 0) + 1);
  });
  
  // Filter duplicates
  const duplicates = [];
  frequency.forEach((count, item) => {
    if (count > 1) {
      duplicates.push({ value: item, count });
    }
  });
  
  return duplicates;
}

console.log(getDuplicatesWithCount(numbers));
// [{ value: 2, count: 2 }, { value: 1, count: 2 }, { value: 3, count: 2 }]
```

### Style 3: Object Duplicates (by specific field)

```javascript
function findDuplicateObjects(arr, key) {
  const seen = new Map();
  const duplicates = [];
  
  arr.forEach((item, index) => {
    const keyValue = item[key];
    
    if (seen.has(keyValue)) {
      duplicates.push({
        original: seen.get(keyValue),
        duplicate: { item, index }
      });
    } else {
      seen.set(keyValue, { item, index });
    }
  });
  
  return duplicates;
}

// Usage
const products = [
  { id: 1, name: 'Apple', sku: 'A123' },
  { id: 2, name: 'Banana', sku: 'B456' },
  { id: 3, name: 'Cherry', sku: 'A123' }, // Duplicate SKU
];

console.log(findDuplicateObjects(products, 'sku'));
```

### Style 4: Remove All Duplicates (Keep First Occurrence)

```javascript
function removeDuplicates(arr, key = null) {
  if (key) {
    // For objects
    const seen = new Set();
    return arr.filter(item => {
      const keyValue = item[key];
      if (seen.has(keyValue)) return false;
      seen.add(keyValue);
      return true;
    });
  }
  
  // For primitives
  return [...new Set(arr)];
}

// Primitives
console.log(removeDuplicates([1, 2, 2, 3, 4, 4, 5])); 
// [1, 2, 3, 4, 5]

// Objects
console.log(removeDuplicates(products, 'sku'));
// Removes objects with duplicate SKUs
```

---

## 5. Nested Arrays - Unique Identification

### Style 1: Flatten and Find Unique (Simple)

```javascript
const nestedArray = [
  [1, 2, 3],
  [3, 4, 5],
  [5, 6, 7],
  [7, 8, 9]
];

// Flatten and get unique
const uniqueFlat = [...new Set(nestedArray.flat())];
console.log(uniqueFlat); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### Style 2: Deep Flatten with Multiple Levels

```javascript
const deepNested = [1, [2, [3, [4, 5]]], 6, [7, 8]];

// Recursive flatten
function deepFlatten(arr) {
  return arr.reduce((acc, val) => {
    return acc.concat(Array.isArray(val) ? deepFlatten(val) : val);
  }, []);
}

// Or use built-in with Infinity
const flattened = deepNested.flat(Infinity);
const unique = [...new Set(flattened)];

console.log(unique); // [1, 2, 3, 4, 5, 6, 7, 8]
```

### Style 3: Nested Objects - Find Unique by Property

```javascript
const nestedUsers = [
  [
    { id: 1, name: 'Alice', dept: 'IT' },
    { id: 2, name: 'Bob', dept: 'HR' }
  ],
  [
    { id: 3, name: 'Charlie', dept: 'IT' },
    { id: 1, name: 'Alice', dept: 'IT' } // Duplicate
  ],
  [
    { id: 4, name: 'David', dept: 'Finance' }
  ]
];

function getUniqueNestedObjects(nestedArr, uniqueKey) {
  const seen = new Set();
  const unique = [];
  
  nestedArr.flat().forEach(item => {
    const key = item[uniqueKey];
    if (!seen.has(key)) {
      seen.add(key);
      unique.push(item);
    }
  });
  
  return unique;
}

const uniqueUsers = getUniqueNestedObjects(nestedUsers, 'id');
console.log(uniqueUsers);
// [Alice, Bob, Charlie, David] (4 unique users)
```

### Style 4: Complex Nested Structure - Multi-level Uniqueness

```javascript
const complexData = [
  {
    department: 'IT',
    teams: [
      { teamId: 'T1', members: [1, 2, 3] },
      { teamId: 'T2', members: [3, 4, 5] }
    ]
  },
  {
    department: 'HR',
    teams: [
      { teamId: 'T3', members: [5, 6, 7] },
      { teamId: 'T1', members: [8, 9] } // Duplicate teamId
    ]
  }
];

// Extract all unique team IDs
function extractUniqueTeamIds(data) {
  const teamIds = new Set();
  
  data.forEach(dept => {
    dept.teams.forEach(team => {
      teamIds.add(team.teamId);
    });
  });
  
  return Array.from(teamIds);
}

// Extract all unique member IDs
function extractUniqueMemberIds(data) {
  const members = new Set();
  
  data.forEach(dept => {
    dept.teams.forEach(team => {
      team.members.forEach(memberId => {
        members.add(memberId);
      });
    });
  });
  
  return Array.from(members);
}

console.log(extractUniqueTeamIds(complexData)); 
// ['T1', 'T2', 'T3']

console.log(extractUniqueMemberIds(complexData)); 
// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### Style 5: Optimized - Map-based Nested Unique Extraction

```javascript
function extractNestedUnique(nestedArr, path) {
  const unique = new Map();
  
  function traverse(obj, currentPath = []) {
    if (Array.isArray(obj)) {
      obj.forEach(item => traverse(item, currentPath));
    } else if (typeof obj === 'object' && obj !== null) {
      // Check if we've reached the target path
      if (currentPath.join('.') === path.slice(0, -1).join('.')) {
        const key = path[path.length - 1];
        if (obj[key] !== undefined) {
          const value = obj[key];
          unique.set(JSON.stringify(value), value);
        }
      }
      
      // Continue traversing
      Object.entries(obj).forEach(([key, value]) => {
        traverse(value, [...currentPath, key]);
      });
    }
  }
  
  traverse(nestedArr);
  return Array.from(unique.values());
}

// Usage
const data = [
  { users: [{ id: 1, name: 'A' }, { id: 2, name: 'B' }] },
  { users: [{ id: 1, name: 'A' }, { id: 3, name: 'C' }] }
];

const uniqueIds = extractNestedUnique(data, ['users', 'id']);
console.log(uniqueIds); // [1, 2, 3]
```

---

## 6. Performance Comparison for Large Datasets

```javascript
// Generate large dataset
const largeArray = Array.from({ length: 1000000 }, (_, i) => i);

// Benchmark helper
function benchmark(fn, label) {
  const start = performance.now();
  const result = fn();
  const end = performance.now();
  console.log(`${label}: ${(end - start).toFixed(2)}ms`);
  return result;
}

// Compare different approaches
console.log('=== Filtering 1M elements ===');

benchmark(
  () => largeArray.filter(n => n % 2 === 0),
  'Traditional filter'
);

benchmark(
  () => {
    const result = [];
    for (let i = 0; i < largeArray.length; i++) {
      if (largeArray[i] % 2 === 0) result.push(largeArray[i]);
    }
    return result;
  },
  'For loop'
);

console.log('\n=== Finding duplicates ===');

const withDups = [...largeArray, ...largeArray.slice(0, 100000)];

benchmark(
  () => {
    const seen = new Set();
    const dups = new Set();
    withDups.forEach(n => {
      if (seen.has(n)) dups.add(n);
      else seen.add(n);
    });
    return Array.from(dups);
  },
  'Set-based approach'
);

benchmark(
  () => {
    const map = new Map();
    withDups.forEach(n => map.set(n, (map.get(n) || 0) + 1));
    return Array.from(map.entries())
      .filter(([_, count]) => count > 1)
      .map(([val]) => val);
  },
  'Map-based approach'
);
```

---

## 7. Best Practices Summary

### ✅ DO's

1. **Use Set for uniqueness** - O(1) lookups vs O(n) for arrays
2. **Use Map for key-value searches** - Faster than object property access
3. **Use binary search on sorted data** - O(log n) vs O(n)
4. **Minimize array copies** - Use in-place operations when possible
5. **Use typed arrays for numeric data** - Better performance and memory
6. **Consider indexing** - Pre-process data for multiple operations
7. **Batch operations** - Combine filter, map, reduce when possible

### ❌ DON'Ts

1. **Avoid nested loops** - O(n²) complexity kills performance
2. **Don't repeatedly search unsorted arrays** - Sort first or use hash maps
3. **Don't use `includes()` in loops** - Use Set instead
4. **Avoid excessive array spreading** - `[...arr]` creates copies
5. **Don't ignore memory constraints** - Process in chunks if needed

---

## 8. Real-World Example: Complete Data Processing Pipeline

```javascript
class DataProcessor {
  constructor(data) {
    this.data = data;
    this.index = null;
  }
  
  // Build index for fast lookups
  buildIndex(key) {
    this.index = new Map();
    this.data.forEach((item, idx) => {
      this.index.set(item[key], { item, idx });
    });
    return this;
  }
  
  // Search using index
  search(key, value) {
    return this.index?.get(value) || null;
  }
  
  // Remove duplicates
  deduplicate(key) {
    const seen = new Set();
    this.data = this.data.filter(item => {
      const val = item[key];
      if (seen.has(val)) return false;
      seen.add(val);
      return true;
    });
    return this;
  }
  
  // Sort with multiple criteria
  sort(criteria) {
    this.data.sort((a, b) => {
      for (let { key, order = 'asc' } of criteria) {
        const aVal = a[key];
        const bVal = b[key];
        
        if (aVal !== bVal) {
          const comparison = aVal < bVal ? -1 : 1;
          return order === 'asc' ? comparison : -comparison;
        }
      }
      return 0;
    });
    return this;
  }
  
  // Filter with multiple conditions
  filter(conditions) {
    this.data = this.data.filter(item => {
      return conditions.every(({ key, operator, value }) => {
        switch (operator) {
          case '=': return item[key] === value;
          case '>': return item[key] > value;
          case '<': return item[key] < value;
          case 'contains': return item[key]?.includes(value);
          default: return true;
        }
      });
    });
    return this;
  }
  
  getData() {
    return this.data;
  }
}

// Usage
const users = [
  { id: 1, name: 'Alice', age: 30, dept: 'IT' },
  { id: 2, name: 'Bob', age: 25, dept: 'HR' },
  { id: 1, name: 'Alice', age: 30, dept: 'IT' }, // Duplicate
  { id: 3, name: 'Charlie', age: 35, dept: 'IT' },
];

const processor = new DataProcessor(users);

const result = processor
  .deduplicate('id')
  .filter([
    { key: 'dept', operator: '=', value: 'IT' },
    { key: 'age', operator: '>', value: 25 }
  ])
  .sort([
    { key: 'age', order: 'desc' },
    { key: 'name', order: 'asc' }
  ])
  .buildIndex('id')
  .getData();

console.log(result);
// [{ Charlie, 35 }, { Alice, 30 }]

// Fast lookup
console.log(processor.search('id', 3));
// { item: { id: 3, ... }, idx: 0 }
```

---

## Conclusion

Different coding styles have different performance characteristics:

- **Simple operations**: Use built-in methods (readable, optimized)
- **Large datasets**: Use Set/Map, avoid nested loops
- **Complex queries**: Build indexes, use hash maps
- **Memory constraints**: Process in chunks, use generators
- **Multiple operations**: Chain methods, minimize iterations

Choose the style that best fits your data size, operation frequency, and performance requirements.