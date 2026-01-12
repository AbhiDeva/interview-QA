# JavaScript Code Challenge: Array Manipulation

## Problem Statement

Write a function called `groupByProperty` that takes an array of objects and a property name, then returns an object where the keys are unique values of that property and the values are arrays of objects that share that property value.

### Requirements

- The function should accept two parameters: an array of objects and a string representing the property name
- If the property doesn't exist on an object, skip that object
- If the array is empty, return an empty object
- The function should handle edge cases gracefully

### Example

```javascript
const users = [
  { name: 'Alice', role: 'admin', age: 30 },
  { name: 'Bob', role: 'user', age: 25 },
  { name: 'Charlie', role: 'admin', age: 35 },
  { name: 'David', role: 'user', age: 28 },
  { name: 'Eve', role: 'moderator', age: 32 }
];

console.log(groupByProperty(users, 'role'));
```

### Expected Output

```javascript
{
  admin: [
    { name: 'Alice', role: 'admin', age: 30 },
    { name: 'Charlie', role: 'admin', age: 35 }
  ],
  user: [
    { name: 'Bob', role: 'user', age: 25 },
    { name: 'David', role: 'user', age: 28 }
  ],
  moderator: [
    { name: 'Eve', role: 'moderator', age: 32 }
  ]
}
```

---

## Solution

```javascript
function groupByProperty(array, property) {
  // Handle edge cases
  if (!Array.isArray(array) || array.length === 0) {
    return {};
  }
  
  // Use reduce to build the grouped object
  return array.reduce((grouped, item) => {
    // Check if the property exists on the current item
    if (item.hasOwnProperty(property)) {
      const key = item[property];
      
      // Initialize the array for this key if it doesn't exist
      if (!grouped[key]) {
        grouped[key] = [];
      }
      
      // Add the item to the appropriate group
      grouped[key].push(item);
    }
    
    return grouped;
  }, {});
}

// Test the function
const users = [
  { name: 'Alice', role: 'admin', age: 30 },
  { name: 'Bob', role: 'user', age: 25 },
  { name: 'Charlie', role: 'admin', age: 35 },
  { name: 'David', role: 'user', age: 28 },
  { name: 'Eve', role: 'moderator', age: 32 }
];

console.log(groupByProperty(users, 'role'));
```

### Alternative Solution (More Concise)

```javascript
function groupByProperty(array, property) {
  if (!Array.isArray(array)) return {};
  
  return array.reduce((grouped, item) => {
    const key = item?.[property];
    if (key !== undefined) {
      (grouped[key] = grouped[key] || []).push(item);
    }
    return grouped;
  }, {});
}
```

---

## Explanation

The solution uses the `reduce()` method to iterate through the array and build up a grouped object. Here's how it works:

1. **Edge Case Handling**: First, we check if the input is a valid array and return an empty object if not.

2. **Reduce Method**: We use `reduce()` with an empty object as the initial accumulator value.

3. **Property Check**: For each item, we check if it has the specified property using `hasOwnProperty()`.

4. **Group Initialization**: If the key doesn't exist in our grouped object yet, we create an empty array for it.

5. **Item Addition**: We push the current item into the appropriate group array.

6. **Return Accumulator**: The accumulator is returned for the next iteration.

### Time Complexity
- **O(n)** where n is the number of elements in the array

### Space Complexity
- **O(n)** for storing all elements in the grouped object

---

## Additional Test Cases

```javascript
// Test with different property
console.log(groupByProperty(users, 'age'));
// Groups by age values

// Test with empty array
console.log(groupByProperty([], 'role'));
// Returns: {}

// Test with non-existent property
console.log(groupByProperty(users, 'department'));
// Returns: {}

// Test with mixed data
const mixedData = [
  { name: 'Item1', category: 'A' },
  { name: 'Item2' }, // Missing category
  { name: 'Item3', category: 'A' },
  { name: 'Item4', category: 'B' }
];
console.log(groupByProperty(mixedData, 'category'));
// Skips Item2, groups others by category
```