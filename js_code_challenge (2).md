# JavaScript Code Challenges Collection

## Challenge 1: Group By Property

### Problem
Write a function that groups an array of objects by a specified property.

```javascript
const users = [
  { name: 'Alice', role: 'admin' },
  { name: 'Bob', role: 'user' },
  { name: 'Charlie', role: 'admin' }
];
groupByProperty(users, 'role');
// { admin: [{...}, {...}], user: [{...}] }
```

### Solution
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

## Challenge 2: Deep Clone Object

### Problem
Create a function that deep clones an object, including nested objects and arrays.

### Solution
```javascript
function deepClone(obj) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof Array) return obj.map(item => deepClone(item));
  
  const cloned = {};
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      cloned[key] = deepClone(obj[key]);
    }
  }
  return cloned;
}
```

---

## Challenge 3: Debounce Function

### Problem
Implement a debounce function that delays execution until after a specified wait time.

### Solution
```javascript
function debounce(func, wait) {
  let timeout;
  return function executedFunction(...args) {
    const later = () => {
      clearTimeout(timeout);
      func(...args);
    };
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
  };
}
```

---

## Challenge 4: Flatten Nested Array

### Problem
Flatten a deeply nested array into a single-level array.

```javascript
flatten([1, [2, [3, [4]], 5]]); // [1, 2, 3, 4, 5]
```

### Solution
```javascript
function flatten(arr) {
  return arr.reduce((flat, item) => {
    return flat.concat(Array.isArray(item) ? flatten(item) : item);
  }, []);
}

// Alternative using flat()
const flatten = arr => arr.flat(Infinity);
```

---

## Challenge 5: Throttle Function

### Problem
Create a throttle function that limits how often a function can be called.

### Solution
```javascript
function throttle(func, limit) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}
```

---

## Challenge 6: Find Duplicates

### Problem
Find all duplicate values in an array.

### Solution
```javascript
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
```

---

## Challenge 7: Chunk Array

### Problem
Split an array into chunks of a specified size.

```javascript
chunk([1, 2, 3, 4, 5], 2); // [[1, 2], [3, 4], [5]]
```

### Solution
```javascript
function chunk(array, size) {
  const chunks = [];
  for (let i = 0; i < array.length; i += size) {
    chunks.push(array.slice(i, i + size));
  }
  return chunks;
}
```

---

## Challenge 8: Memoization

### Problem
Implement a memoization function to cache expensive function results.

### Solution
```javascript
function memoize(fn) {
  const cache = {};
  return function(...args) {
    const key = JSON.stringify(args);
    if (key in cache) {
      return cache[key];
    }
    const result = fn.apply(this, args);
    cache[key] = result;
    return result;
  };
}
```

---

## Challenge 9: Remove Duplicates

### Problem
Remove duplicate values from an array while preserving order.

### Solution
```javascript
function removeDuplicates(arr) {
  return [...new Set(arr)];
}

// Alternative with filter
function removeDuplicates(arr) {
  return arr.filter((item, index) => arr.indexOf(item) === index);
}
```

---

## Challenge 10: Curry Function

### Problem
Implement function currying for any number of arguments.

### Solution
```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function(...nextArgs) {
      return curried.apply(this, args.concat(nextArgs));
    };
  };
}

// Usage: const add = curry((a, b, c) => a + b + c);
// add(1)(2)(3) === add(1, 2)(3) === add(1, 2, 3)
```

---

## Challenge 11: Array Intersection

### Problem
Find the intersection of two or more arrays.

### Solution
```javascript
function intersection(...arrays) {
  if (arrays.length === 0) return [];
  return arrays.reduce((acc, arr) => 
    acc.filter(item => arr.includes(item))
  );
}
```

---

## Challenge 12: Object Deep Merge

### Problem
Deeply merge two or more objects.

### Solution
```javascript
function deepMerge(target, ...sources) {
  if (!sources.length) return target;
  const source = sources.shift();
  
  if (typeof target === 'object' && typeof source === 'object') {
    for (const key in source) {
      if (typeof source[key] === 'object' && !Array.isArray(source[key])) {
        if (!target[key]) Object.assign(target, { [key]: {} });
        deepMerge(target[key], source[key]);
      } else {
        Object.assign(target, { [key]: source[key] });
      }
    }
  }
  
  return deepMerge(target, ...sources);
}
```

---

## Challenge 13: Palindrome Checker

### Problem
Check if a string is a palindrome (reads same forwards and backwards).

### Solution
```javascript
function isPalindrome(str) {
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  return cleaned === cleaned.split('').reverse().join('');
}
```

---

## Challenge 14: Binary Search

### Problem
Implement binary search on a sorted array.

### Solution
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
```

---

## Challenge 15: FizzBuzz

### Problem
Print numbers 1 to n, replacing multiples of 3 with "Fizz", 5 with "Buzz", and both with "FizzBuzz".

### Solution
```javascript
function fizzBuzz(n) {
  const result = [];
  for (let i = 1; i <= n; i++) {
    if (i % 15 === 0) result.push('FizzBuzz');
    else if (i % 3 === 0) result.push('Fizz');
    else if (i % 5 === 0) result.push('Buzz');
    else result.push(i);
  }
  return result;
}
```

---

## Challenge 16: Max Subarray Sum (Kadane's Algorithm)

### Problem
Find the maximum sum of a contiguous subarray.

### Solution
```javascript
function maxSubarraySum(arr) {
  let maxSum = arr[0];
  let currentSum = arr[0];
  
  for (let i = 1; i < arr.length; i++) {
    currentSum = Math.max(arr[i], currentSum + arr[i]);
    maxSum = Math.max(maxSum, currentSum);
  }
  
  return maxSum;
}
```

---

## Challenge 17: Anagram Checker

### Problem
Check if two strings are anagrams of each other.

### Solution
```javascript
function areAnagrams(str1, str2) {
  const normalize = str => str.toLowerCase().replace(/[^a-z]/g, '').split('').sort().join('');
  return normalize(str1) === normalize(str2);
}
```

---

## Challenge 18: Promise.all Implementation

### Problem
Implement your own version of Promise.all.

### Solution
```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let completed = 0;
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = value;
          completed++;
          if (completed === promises.length) {
            resolve(results);
          }
        })
        .catch(reject);
    });
  });
}
```

---

## Challenge 19: Title Case

### Problem
Convert a string to title case (first letter of each word capitalized).

### Solution
```javascript
function titleCase(str) {
  return str
    .toLowerCase()
    .split(' ')
    .map(word => word.charAt(0).toUpperCase() + word.slice(1))
    .join(' ');
}
```

---

## Challenge 20: Array Difference

### Problem
Find elements in the first array that are not in the second array.

### Solution
```javascript
function arrayDifference(arr1, arr2) {
  const set2 = new Set(arr2);
  return arr1.filter(item => !set2.has(item));
}
```

---

## Challenge 21: LRU Cache

### Problem
Implement a Least Recently Used (LRU) cache.

### Solution
```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }
  
  get(key) {
    if (!this.cache.has(key)) return -1;
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }
  
  put(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }
    this.cache.set(key, value);
    if (this.cache.size > this.capacity) {
      this.cache.delete(this.cache.keys().next().value);
    }
  }
}
```

---

## Challenge 22: Event Emitter

### Problem
Create a simple event emitter class.

### Solution
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
  }
  
  emit(event, ...args) {
    if (this.events[event]) {
      this.events[event].forEach(listener => listener(...args));
    }
  }
  
  off(event, listenerToRemove) {
    if (this.events[event]) {
      this.events[event] = this.events[event].filter(
        listener => listener !== listenerToRemove
      );
    }
  }
}
```

---

## Challenge 23: Fibonacci Sequence

### Problem
Generate the first n numbers of the Fibonacci sequence.

### Solution
```javascript
function fibonacci(n) {
  if (n <= 0) return [];
  if (n === 1) return [0];
  
  const fib = [0, 1];
  for (let i = 2; i < n; i++) {
    fib[i] = fib[i - 1] + fib[i - 2];
  }
  return fib;
}

// Recursive with memoization
const fibMemo = (n, memo = {}) => {
  if (n in memo) return memo[n];
  if (n <= 1) return n;
  memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
  return memo[n];
};
```

---

## Challenge 24: Sum of Digits

### Problem
Calculate the sum of all digits in a number.

### Solution
```javascript
function sumDigits(num) {
  return Math.abs(num)
    .toString()
    .split('')
    .reduce((sum, digit) => sum + parseInt(digit), 0);
}
```

---

## Challenge 25: Reverse Words

### Problem
Reverse the words in a sentence while maintaining word order.

### Solution
```javascript
function reverseWords(str) {
  return str
    .split(' ')
    .map(word => word.split('').reverse().join(''))
    .join(' ');
}
```

---

## Challenge 26: Valid Parentheses

### Problem
Check if a string of parentheses is valid (properly opened and closed).

### Solution
```javascript
function isValidParentheses(str) {
  const stack = [];
  const pairs = { '(': ')', '[': ']', '{': '}' };
  
  for (let char of str) {
    if (pairs[char]) {
      stack.push(char);
    } else {
      const last = stack.pop();
      if (pairs[last] !== char) return false;
    }
  }
  
  return stack.length === 0;
}
```

---

## Challenge 27: Unique Permutations

### Problem
Generate all unique permutations of an array.

### Solution
```javascript
function permutations(arr) {
  if (arr.length <= 1) return [arr];
  
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    const current = arr[i];
    const remaining = arr.slice(0, i).concat(arr.slice(i + 1));
    const perms = permutations(remaining);
    
    for (let perm of perms) {
      result.push([current, ...perm]);
    }
  }
  
  return result;
}
```

---

## Challenge 28: Most Frequent Element

### Problem
Find the most frequently occurring element in an array.

### Solution
```javascript
function mostFrequent(arr) {
  const frequency = {};
  let maxCount = 0;
  let mostFrequentItem;
  
  arr.forEach(item => {
    frequency[item] = (frequency[item] || 0) + 1;
    if (frequency[item] > maxCount) {
      maxCount = frequency[item];
      mostFrequentItem = item;
    }
  });
  
  return mostFrequentItem;
}
```

---

## Challenge 29: Rotate Array

### Problem
Rotate an array to the right by k steps.

### Solution
```javascript
function rotateArray(arr, k) {
  k = k % arr.length;
  return arr.slice(-k).concat(arr.slice(0, -k));
}

// In-place rotation
function rotateInPlace(arr, k) {
  k = k % arr.length;
  reverse(arr, 0, arr.length - 1);
  reverse(arr, 0, k - 1);
  reverse(arr, k, arr.length - 1);
  return arr;
}

function reverse(arr, start, end) {
  while (start < end) {
    [arr[start], arr[end]] = [arr[end], arr[start]];
    start++;
    end--;
  }
}
```

---

## Challenge 30: Longest Substring Without Repeating Characters

### Problem
Find the length of the longest substring without repeating characters.

### Solution
```javascript
function lengthOfLongestSubstring(s) {
  const seen = new Map();
  let maxLength = 0;
  let start = 0;
  
  for (let i = 0; i < s.length; i++) {
    if (seen.has(s[i]) && seen.get(s[i]) >= start) {
      start = seen.get(s[i]) + 1;
    }
    seen.set(s[i], i);
    maxLength = Math.max(maxLength, i - start + 1);
  }
  
  return maxLength;
}
```

---

## Challenge 31: Two Sum

### Problem
Find two numbers in an array that add up to a target sum.

### Solution
```javascript
function twoSum(nums, target) {
  const map = new Map();
  
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    if (map.has(complement)) {
      return [map.get(complement), i];
    }
    map.set(nums[i], i);
  }
  
  return null;
}
```

---

## Challenge 32: Prime Number Checker

### Problem
Check if a number is prime.

### Solution
```javascript
function isPrime(n) {
  if (n <= 1) return false;
  if (n <= 3) return true;
  if (n % 2 === 0 || n % 3 === 0) return false;
  
  for (let i = 5; i * i <= n; i += 6) {
    if (n % i === 0 || n % (i + 2) === 0) return false;
  }
  
  return true;
}
```

---

## Challenge 33: Range Sum Query

### Problem
Implement a class to efficiently calculate sum of elements between indices.

### Solution
```javascript
class RangeSumQuery {
  constructor(nums) {
    this.prefixSum = [0];
    for (let i = 0; i < nums.length; i++) {
      this.prefixSum[i + 1] = this.prefixSum[i] + nums[i];
    }
  }
  
  sumRange(left, right) {
    return this.prefixSum[right + 1] - this.prefixSum[left];
  }
}
```

---

## Challenge 34: Merge Intervals

### Problem
Merge overlapping intervals.

### Solution
```javascript
function mergeIntervals(intervals) {
  if (intervals.length <= 1) return intervals;
  
  intervals.sort((a, b) => a[0] - b[0]);
  const merged = [intervals[0]];
  
  for (let i = 1; i < intervals.length; i++) {
    const last = merged[merged.length - 1];
    const current = intervals[i];
    
    if (current[0] <= last[1]) {
      last[1] = Math.max(last[1], current[1]);
    } else {
      merged.push(current);
    }
  }
  
  return merged;
}
```

---

## Challenge 35: String Compression

### Problem
Compress a string using character counts (e.g., "aaabbc" -> "a3b2c1").

### Solution
```javascript
function compressString(str) {
  if (!str) return '';
  
  let compressed = '';
  let count = 1;
  
  for (let i = 0; i < str.length; i++) {
    if (str[i] === str[i + 1]) {
      count++;
    } else {
      compressed += str[i] + count;
      count = 1;
    }
  }
  
  return compressed.length < str.length ? compressed : str;
}
```

---

## Challenge 36: Missing Number

### Problem
Find the missing number in an array containing n distinct numbers from 0 to n.

### Solution
```javascript
function findMissingNumber(nums) {
  const n = nums.length;
  const expectedSum = (n * (n + 1)) / 2;
  const actualSum = nums.reduce((sum, num) => sum + num, 0);
  return expectedSum - actualSum;
}

// XOR approach
function findMissingNumberXOR(nums) {
  let xor = nums.length;
  for (let i = 0; i < nums.length; i++) {
    xor ^= i ^ nums[i];
  }
  return xor;
}
```

---

## Challenge 37: Capitalize First Letter

### Problem
Capitalize the first letter of each word in a sentence.

### Solution
```javascript
function capitalizeFirstLetter(str) {
  return str.replace(/\b\w/g, char => char.toUpperCase());
}
```

---

## Challenge 38: Valid Email Checker

### Problem
Validate an email address format.

### Solution
```javascript
function isValidEmail(email) {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}
```

---

## Challenge 39: Array Shuffle

### Problem
Randomly shuffle an array (Fisher-Yates algorithm).

### Solution
```javascript
function shuffle(array) {
  const arr = [...array];
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  return arr;
}
```

---

## Challenge 40: Longest Common Prefix

### Problem
Find the longest common prefix among an array of strings.

### Solution
```javascript
function longestCommonPrefix(strs) {
  if (!strs.length) return '';
  
  let prefix = strs[0];
  
  for (let i = 1; i < strs.length; i++) {
    while (strs[i].indexOf(prefix) !== 0) {
      prefix = prefix.slice(0, -1);
      if (!prefix) return '';
    }
  }
  
  return prefix;
}
```

---

## Challenge 41: Binary Tree Traversal

### Problem
Implement in-order, pre-order, and post-order tree traversals.

### Solution
```javascript
class TreeNode {
  constructor(val) {
    this.val = val;
    this.left = this.right = null;
  }
}

function inOrderTraversal(root, result = []) {
  if (root) {
    inOrderTraversal(root.left, result);
    result.push(root.val);
    inOrderTraversal(root.right, result);
  }
  return result;
}

function preOrderTraversal(root, result = []) {
  if (root) {
    result.push(root.val);
    preOrderTraversal(root.left, result);
    preOrderTraversal(root.right, result);
  }
  return result;
}

function postOrderTraversal(root, result = []) {
  if (root) {
    postOrderTraversal(root.left, result);
    postOrderTraversal(root.right, result);
    result.push(root.val);
  }
  return result;
}
```

---

## Challenge 42: Count Vowels

### Problem
Count the number of vowels in a string.

### Solution
```javascript
function countVowels(str) {
  const matches = str.match(/[aeiou]/gi);
  return matches ? matches.length : 0;
}
```

---

## Challenge 43: Object Equality

### Problem
Deep compare two objects for equality.

### Solution
```javascript
function deepEqual(obj1, obj2) {
  if (obj1 === obj2) return true;
  
  if (typeof obj1 !== 'object' || typeof obj2 !== 'object' ||
      obj1 == null || obj2 == null) {
    return false;
  }
  
  const keys1 = Object.keys(obj1);
  const keys2 = Object.keys(obj2);
  
  if (keys1.length !== keys2.length) return false;
  
  for (let key of keys1) {
    if (!keys2.includes(key) || !deepEqual(obj1[key], obj2[key])) {
      return false;
    }
  }
  
  return true;
}
```

---

## Challenge 44: Moving Average

### Problem
Calculate the moving average of a stream of numbers.

### Solution
```javascript
class MovingAverage {
  constructor(size) {
    this.size = size;
    this.queue = [];
    this.sum = 0;
  }
  
  next(val) {
    this.queue.push(val);
    this.sum += val;
    
    if (this.queue.length > this.size) {
      this.sum -= this.queue.shift();
    }
    
    return this.sum / this.queue.length;
  }
}
```

---

## Challenge 45: Roman to Integer

### Problem
Convert a Roman numeral to an integer.

### Solution
```javascript
function romanToInt(s) {
  const roman = { I: 1, V: 5, X: 10, L: 50, C: 100, D: 500, M: 1000 };
  let result = 0;
  
  for (let i = 0; i < s.length; i++) {
    const current = roman[s[i]];
    const next = roman[s[i + 1]];
    
    if (next && current < next) {
      result -= current;
    } else {
      result += current;
    }
  }
  
  return result;
}
```

---

## Challenge 46: Integer to Roman

### Problem
Convert an integer to a Roman numeral.

### Solution
```javascript
function intToRoman(num) {
  const values = [1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1];
  const symbols = ['M', 'CM', 'D', 'CD', 'C', 'XC', 'L', 'XL', 'X', 'IX', 'V', 'IV', 'I'];
  
  let result = '';
  
  for (let i = 0; i < values.length && num > 0; i++) {
    while (num >= values[i]) {
      result += symbols[i];
      num -= values[i];
    }
  }
  
  return result;
}
```

---

## Challenge 47: Power of Two

### Problem
Check if a number is a power of two.

### Solution
```javascript
function isPowerOfTwo(n) {
  return n > 0 && (n & (n - 1)) === 0;
}

// Alternative
function isPowerOfTwo(n) {
  if (n <= 0) return false;
  while (n % 2 === 0) {
    n /= 2;
  }
  return n === 1;
}
```

---

## Challenge 48: Generate Parentheses

### Problem
Generate all combinations of n pairs of valid parentheses.

### Solution
```javascript
function generateParentheses(n) {
  const result = [];
  
  function backtrack(current, open, close) {
    if (current.length === 2 * n) {
      result.push(current);
      return;
    }
    
    if (open < n) {
      backtrack(current + '(', open + 1, close);
    }
    
    if (close < open) {
      backtrack(current + ')', open, close + 1);
    }
  }
  
  backtrack('', 0, 0);
  return result;
}
```

---

## Challenge 49: First Non-Repeating Character

### Problem
Find the first non-repeating character in a string.

### Solution
```javascript
function firstNonRepeating(str) {
  const charCount = {};
  
  for (let char of str) {
    charCount[char] = (charCount[char] || 0) + 1;
  }
  
  for (let char of str) {
    if (charCount[char] === 1) {
      return char;
    }
  }
  
  return null;
}
```

---

## Challenge 50: Reverse Integer

### Problem
Reverse the digits of an integer.

### Solution
```javascript
function reverseInteger(num) {
  const isNegative = num < 0;
  const reversed = parseInt(
    Math.abs(num).toString().split('').reverse().join('')
  );
  
  const result = isNegative ? -reversed : reversed;
  
  // Check for 32-bit integer overflow
  if (result < -2147483648 || result > 2147483647) {
    return 0;
  }
  
  return result;
}
```

---

## Challenge 51: Snake to Camel Case

### Problem
Convert snake_case strings to camelCase.

### Solution
```javascript
function snakeToCamel(str) {
  return str.replace(/_([a-z])/g, (_, letter) => letter.toUpperCase());
}
```

---

## Challenge 52: Camel to Snake Case

### Problem
Convert camelCase strings to snake_case.

### Solution
```javascript
function camelToSnake(str) {
  return str.replace(/[A-Z]/g, letter => `_${letter.toLowerCase()}`);
}
```

---

## Challenge 53: Find Kth Largest Element

### Problem
Find the kth largest element in an unsorted array.

### Solution
```javascript
function findKthLargest(nums, k) {
  return nums.sort((a, b) => b - a)[k - 1];
}

// Using QuickSelect (O(n) average)
function findKthLargestOptimized(nums, k) {
  const targetIndex = k - 1;
  
  function quickSelect(left, right) {
    const pivot = nums[right];
    let p = left;
    
    for (let i = left; i < right; i++) {
      if (nums[i] >= pivot) {
        [nums[i], nums[p]] = [nums[p], nums[i]];
        p++;
      }
    }
    
    [nums[p], nums[right]] = [nums[right], nums[p]];
    
    if (p === targetIndex) return nums[p];
    if (p < targetIndex) return quickSelect(p + 1, right);
    return quickSelect(left, p - 1);
  }
  
  return quickSelect(0, nums.length - 1);
}
```

---

## Challenge 54: Implement Stack with Queues

### Problem
Implement a stack using only queue operations.

### Solution
```javascript
class StackWithQueues {
  constructor() {
    this.q1 = [];
    this.q2 = [];
  }
  
  push(x) {
    this.q1.push(x);
  }
  
  pop() {
    while (this.q1.length > 1) {
      this.q2.push(this.q1.shift());
    }
    const popped = this.q1.shift();
    [this.q1, this.q2] = [this.q2, this.q1];
    return popped;
  }
  
  top() {
    while (this.q1.length > 1) {
      this.q2.push(this.q1.shift());
    }
    const top = this.q1[0];
    this.q2.push(this.q1.shift());
    [this.q1, this.q2] = [this.q2, this.q1];
    return top;
  }
  
  empty() {
    return this.q1.length === 0;
  }
}
```

---

## Challenge 55: Implement Queue with Stacks

### Problem
Implement a queue using only stack operations.

### Solution
```javascript
class QueueWithStacks {
  constructor() {
    this.stack1 = [];
    this.stack2 = [];
  }
  
  enqueue(x) {
    this.stack1.push(x);
  }
  
  dequeue() {
    if (this.stack2.length === 0) {
      while (this.stack1.length > 0) {
        this.stack2.push(this.stack1.pop());
      }
    }
    return this.stack2.pop();
  }
  
  peek() {
    if (this.stack2.length === 0) {
      while (this.stack1.length > 0) {
        this.stack2.push(this.stack1.pop());
      }
    }
    return this.stack2[this.stack2.length - 1];
  }
  
  empty() {
    return this.stack1.length === 0 && this.stack2.length === 0;
  }
}
```

---

## Challenge 56: Product of Array Except Self

### Problem
Return an array where each element is the product of all other elements.

### Solution
```javascript
function productExceptSelf(nums) {
  const n = nums.length;
  const result = new Array(n).fill(1);
  
  // Left pass
  let left = 1;
  for (let i = 0; i < n; i++) {
    result[i] = left;
    left *= nums[i];
  }
  
  // Right pass
  let right = 1;
  for (let i = n - 1; i >= 0; i--) {
    result[i] *= right;
    right *= nums[i];
  }
  
  return result;
}
```

---

## Challenge 57: Climbing Stairs

### Problem
Count the number of ways to climb n stairs (taking 1 or 2 steps at a time).

### Solution
```javascript
function climbStairs(n) {
  if (n <= 2) return n;
  
  let prev2 = 1;
  let prev1 = 2;
  
  for (let i = 3; i <= n; i++) {
    const current = prev1 + prev2;
    prev2 = prev1;
    prev1 = current;
  }
  
  return prev1;
}
```

---

## Challenge 58: Word Break

### Problem
Check if a string can be segmented into space-separated words from a dictionary.

### Solution
```javascript
function wordBreak(s, wordDict) {
  const wordSet = new Set(wordDict);
  const dp = new Array(s.length + 1).fill(false);
  dp[0] = true;
  
  for (let i = 1; i <= s.length; i++) {
    for (let j = 0; j < i; j++) {
      if (dp[j] && wordSet.has(s.substring(j, i))) {
        dp[i] = true;
        break;
      }
    }
  }
  
  return dp[s.length];
}
```

---

## Challenge 59: Unique Paths

### Problem
Find number of unique paths in a grid from top-left to bottom-right.

### Solution
```javascript
function uniquePaths(m, n) {
  const dp = Array(m).fill(0).map(() => Array(n).fill(1));
  
  for (let i = 1; i < m; i++) {
    for (let j = 1; j < n; j++) {
      dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
    }
  }
  
  return dp[m - 1][n - 1];
}

// Space-optimized version
function uniquePathsOptimized(m, n) {
  const dp = new Array(n).fill(1);
  
  for (let i = 1; i < m; i++) {
    for (let j = 1; j < n; j++) {
      dp[j] += dp[j - 1];
    }
  }
  
  return dp[n - 1];
}
```

---

## Challenge 60: Coin Change

### Problem
Find the minimum number of coins needed to make a given amount.

### Solution
```javascript
function coinChange(coins, amount) {
  const dp = new Array(amount + 1).fill(Infinity);
  dp[0] = 0;
  
  for (let i = 1; i <= amount; i++) {
    for (let coin of coins) {
      if (coin <= i) {
        dp[i] = Math.min(dp[i], dp[i - coin] + 1);
      }
    }
  }
  
  return dp[amount] === Infinity ? -1 : dp[amount];
}
```

---

## Challenge 61: Longest Increasing Subsequence

### Problem
Find the length of the longest increasing subsequence.

### Solution
```javascript
function lengthOfLIS(nums) {
  if (nums.length === 0) return 0;
  
  const dp = new Array(nums.length).fill(1);
  
  for (let i = 1; i < nums.length; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[i] > nums[j]) {
        dp[i] = Math.max(dp[i], dp[j] + 1);
      }
    }
  }
  
  return Math.max(...dp);
}
```

---

## Challenge 62: Container With Most Water

### Problem
Find two lines that form a container with the maximum water area.

### Solution
```javascript
function maxArea(height) {
  let left = 0;
  let right = height.length - 1;
  let maxArea = 0;
  
  while (left < right) {
    const width = right - left;
    const h = Math.min(height[left], height[right]);
    maxArea = Math.max(maxArea, width * h);
    
    if (height[left] < height[right]) {
      left++;
    } else {
      right--;
    }
  }
  
  return maxArea;
}
```

---

## Challenge 63: Trapping Rain Water

### Problem
Calculate how much water can be trapped between bars.

### Solution
```javascript
function trap(height) {
  let left = 0;
  let right = height.length - 1;
  let leftMax = 0;
  let rightMax = 0;
  let water = 0;
  
  while (left < right) {
    if (height[left] < height[right]) {
      if (height[left] >= leftMax) {
        leftMax = height[left];
      } else {
        water += leftMax - height[left];
      }
      left++;
    } else {
      if (height[right] >= rightMax) {
        rightMax = height[right];
      } else {
        water += rightMax - height[right];
      }
      right--;
    }
  }
  
  return water;
}
```

---

## Challenge 64: Group Anagrams

### Problem
Group strings that are anagrams of each other.

### Solution
```javascript
function groupAnagrams(strs) {
  const map = new Map();
  
  for (let str of strs) {
    const sorted = str.split('').sort().join('');
    if (!map.has(sorted)) {
      map.set(sorted, []);
    }
    map.get(sorted).push(str);
  }
  
  return Array.from(map.values());
}
```

---

## Challenge 65: Sliding Window Maximum

### Problem
Find the maximum value in each sliding window of size k.

### Solution
```javascript
function maxSlidingWindow(nums, k) {
  const result = [];
  const deque = [];
  
  for (let i = 0; i < nums.length; i++) {
    // Remove indices outside the window
    while (deque.length && deque[0] <= i - k) {
      deque.shift();
    }
    
    // Remove smaller elements from the back
    while (deque.length && nums[deque[deque.length - 1]] < nums[i]) {
      deque.pop();
    }
    
    deque.push(i);
    
    // Add to result once window is full
    if (i >= k - 1) {
      result.push(nums[deque[0]]);
    }
  }
  
  return result;
}
```

---

## Challenge 66: Minimum Window Substring

### Problem
Find the minimum window in string s that contains all characters from string t.

### Solution
```javascript
function minWindow(s, t) {
  const need = new Map();
  const window = new Map();
  
  for (let char of t) {
    need.set(char, (need.get(char) || 0) + 1);
  }
  
  let left = 0;
  let right = 0;
  let valid = 0;
  let start = 0;
  let len = Infinity;
  
  while (right < s.length) {
    const c = s[right];
    right++;
    
    if (need.has(c)) {
      window.set(c, (window.get(c) || 0) + 1);
      if (window.get(c) === need.get(c)) {
        valid++;
      }
    }
    
    while (valid === need.size) {
      if (right - left < len) {
        start = left;
        len = right - left;
      }
      
      const d = s[left];
      left++;
      
      if (need.has(d)) {
        if (window.get(d) === need.get(d)) {
          valid--;
        }
        window.set(d, window.get(d) - 1);
      }
    }
  }
  
  return len === Infinity ? '' : s.substr(start, len);
}
```

---

## Challenge 67: Graph DFS

### Problem
Implement depth-first search for a graph.

### Solution
```javascript
function dfs(graph, start, visited = new Set()) {
  visited.add(start);
  console.log(start);
  
  for (let neighbor of graph[start] || []) {
    if (!visited.has(neighbor)) {
      dfs(graph, neighbor, visited);
    }
  }
  
  return visited;
}

// Iterative version
function dfsIterative(graph, start) {
  const visited = new Set();
  const stack = [start];
  
  while (stack.length > 0) {
    const node = stack.pop();
    
    if (!visited.has(node)) {
      visited.add(node);
      console.log(node);
      
      for (let neighbor of graph[node] || []) {
        if (!visited.has(neighbor)) {
          stack.push(neighbor);
        }
      }
    }
  }
  
  return visited;
}
```

---

## Challenge 68: Graph BFS

### Problem
Implement breadth-first search for a graph.

### Solution
```javascript
function bfs(graph, start) {
  const visited = new Set();
  const queue = [start];
  visited.add(start);
  
  while (queue.length > 0) {
    const node = queue.shift();
    console.log(node);
    
    for (let neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push(neighbor);
      }
    }
  }
  
  return visited;
}
```

---

## Challenge 69: Course Schedule (Detect Cycle)

### Problem
Determine if you can finish all courses given prerequisites.

### Solution
```javascript
function canFinish(numCourses, prerequisites) {
  const graph = Array(numCourses).fill(0).map(() => []);
  const visited = new Array(numCourses).fill(0);
  
  // Build adjacency list
  for (let [course, prereq] of prerequisites) {
    graph[course].push(prereq);
  }
  
  function hasCycle(course) {
    if (visited[course] === 1) return true;  // Cycle detected
    if (visited[course] === 2) return false; // Already processed
    
    visited[course] = 1;
    
    for (let prereq of graph[course]) {
      if (hasCycle(prereq)) return true;
    }
    
    visited[course] = 2;
    return false;
  }
  
  for (let i = 0; i < numCourses; i++) {
    if (hasCycle(i)) return false;
  }
  
  return true;
}
```

---

## Challenge 70: Number of Islands

### Problem
Count the number of islands in a 2D grid.

### Solution
```javascript
function numIslands(grid) {
  if (!grid || grid.length === 0) return 0;
  
  let count = 0;
  const rows = grid.length;
  const cols = grid[0].length;
  
  function dfs(i, j) {
    if (i < 0 || i >= rows || j < 0 || j >= cols || grid[i][j] === '0') {
      return;
    }
    
    grid[i][j] = '0'; // Mark as visited
    
    dfs(i + 1, j);
    dfs(i - 1, j);
    dfs(i, j + 1);
    dfs(i, j - 1);
  }
  
  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      if (grid[i][j] === '1') {
        count++;
        dfs(i, j);
      }
    }
  }
  
  return count;
}
```

---

## Challenge 71: Clone Graph

### Problem
Deep clone an undirected graph.

### Solution
```javascript
class GraphNode {
  constructor(val, neighbors = []) {
    this.val = val;
    this.neighbors = neighbors;
  }
}

function cloneGraph(node) {
  if (!node) return null;
  
  const visited = new Map();
  
  function dfs(node) {
    if (visited.has(node)) {
      return visited.get(node);
    }
    
    const clone = new GraphNode(node.val);
    visited.set(node, clone);
    
    for (let neighbor of node.neighbors) {
      clone.neighbors.push(dfs(neighbor));
    }
    
    return clone;
  }
  
  return dfs(node);
}
```

---

## Challenge 72: Serialize and Deserialize Binary Tree

### Problem
Serialize and deserialize a binary tree.

### Solution
```javascript
class TreeNode {
  constructor(val) {
    this.val = val;
    this.left = this.right = null;
  }
}

function serialize(root) {
  if (!root) return 'null';
  return `${root.val},${serialize(root.left)},${serialize(root.right)}`;
}

function deserialize(data) {
  const values = data.split(',');
  
  function build() {
    const val = values.shift();
    if (val === 'null') return null;
    
    const node = new TreeNode(parseInt(val));
    node.left = build();
    node.right = build();
    return node;
  }
  
  return build();
}
```

---

## Challenge 73: Lowest Common Ancestor

### Problem
Find the lowest common ancestor of two nodes in a binary tree.

### Solution
```javascript
function lowestCommonAncestor(root, p, q) {
  if (!root || root === p || root === q) return root;
  
  const left = lowestCommonAncestor(root.left, p, q);
  const right = lowestCommonAncestor(root.right, p, q);
  
  if (left && right) return root;
  return left || right;
}
```

---

## Challenge 74: Binary Tree Level Order Traversal

### Problem
Return level order traversal of a binary tree.

### Solution
```javascript
function levelOrder(root) {
  if (!root) return [];
  
  const result = [];
  const queue = [root];
  
  while (queue.length > 0) {
    const levelSize = queue.length;
    const currentLevel = [];
    
    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      currentLevel.push(node.val);
      
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    
    result.push(currentLevel);
  }
  
  return result;
}
```

---

## Challenge 75: Validate Binary Search Tree

### Problem
Check if a binary tree is a valid BST.

### Solution
```javascript
function isValidBST(root, min = -Infinity, max = Infinity) {
  if (!root) return true;
  
  if (root.val <= min || root.val >= max) {
    return false;
  }
  
  return isValidBST(root.left, min, root.val) && 
         isValidBST(root.right, root.val, max);
}
```

---

## Challenge 76: Kth Smallest Element in BST

### Problem
Find the kth smallest element in a BST.

### Solution
```javascript
function kthSmallest(root, k) {
  const stack = [];
  let current = root;
  
  while (true) {
    while (current) {
      stack.push(current);
      current = current.left;
    }
    
    current = stack.pop();
    k--;
    
    if (k === 0) return current.val;
    
    current = current.right;
  }
}
```

---

## Challenge 77: Binary Tree Maximum Path Sum

### Problem
Find the maximum path sum in a binary tree.

### Solution
```javascript
function maxPathSum(root) {
  let maxSum = -Infinity;
  
  function dfs(node) {
    if (!node) return 0;
    
    const left = Math.max(0, dfs(node.left));
    const right = Math.max(0, dfs(node.right));
    
    maxSum = Math.max(maxSum, left + right + node.val);
    
    return Math.max(left, right) + node.val;
  }
  
  dfs(root);
  return maxSum;
}
```

---

## Challenge 78: Implement Trie (Prefix Tree)

### Problem
Implement a trie with insert, search, and startsWith operations.

### Solution
```javascript
class TrieNode {
  constructor() {
    this.children = {};
    this.isEndOfWord = false;
  }
}

class Trie {
  constructor() {
    this.root = new TrieNode();
  }
  
  insert(word) {
    let node = this.root;
    for (let char of word) {
      if (!node.children[char]) {
        node.children[char] = new TrieNode();
      }
      node = node.children[char];
    }
    node.isEndOfWord = true;
  }
  
  search(word) {
    let node = this.root;
    for (let char of word) {
      if (!node.children[char]) return false;
      node = node.children[char];
    }
    return node.isEndOfWord;
  }
  
  startsWith(prefix) {
    let node = this.root;
    for (let char of prefix) {
      if (!node.children[char]) return false;
      node = node.children[char];
    }
    return true;
  }
}
```

---

## Challenge 79: Word Search

### Problem
Check if a word exists in a 2D board.

### Solution
```javascript
function exist(board, word) {
  const rows = board.length;
  const cols = board[0].length;
  
  function dfs(i, j, k) {
    if (k === word.length) return true;
    if (i < 0 || i >= rows || j < 0 || j >= cols || board[i][j] !== word[k]) {
      return false;
    }
    
    const temp = board[i][j];
    board[i][j] = '#'; // Mark as visited
    
    const found = dfs(i + 1, j, k + 1) ||
                  dfs(i - 1, j, k + 1) ||
                  dfs(i, j + 1, k + 1) ||
                  dfs(i, j - 1, k + 1);
    
    board[i][j] = temp; // Restore
    return found;
  }
  
  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      if (dfs(i, j, 0)) return true;
    }
  }
  
  return false;
}
```

---

## Challenge 80: Letter Combinations of Phone Number

### Problem
Return all letter combinations that a phone number could represent.

### Solution
```javascript
function letterCombinations(digits) {
  if (!digits) return [];
  
  const map = {
    '2': 'abc', '3': 'def', '4': 'ghi', '5': 'jkl',
    '6': 'mno', '7': 'pqrs', '8': 'tuv', '9': 'wxyz'
  };
  
  const result = [];
  
  function backtrack(index, current) {
    if (index === digits.length) {
      result.push(current);
      return;
    }
    
    const letters = map[digits[index]];
    for (let letter of letters) {
      backtrack(index + 1, current + letter);
    }
  }
  
  backtrack(0, '');
  return result;
}
```

---

## Challenge 81: Combination Sum

### Problem
Find all unique combinations that sum to a target.

### Solution
```javascript
function combinationSum(candidates, target) {
  const result = [];
  
  function backtrack(start, current, sum) {
    if (sum === target) {
      result.push([...current]);
      return;
    }
    
    if (sum > target) return;
    
    for (let i = start; i < candidates.length; i++) {
      current.push(candidates[i]);
      backtrack(i, current, sum + candidates[i]);
      current.pop();
    }
  }
  
  backtrack(0, [], 0);
  return result;
}
```

---

## Challenge 82: Subsets

### Problem
Generate all possible subsets of an array.

### Solution
```javascript
function subsets(nums) {
  const result = [];
  
  function backtrack(start, current) {
    result.push([...current]);
    
    for (let i = start; i < nums.length; i++) {
      current.push(nums[i]);
      backtrack(i + 1, current);
      current.pop();
    }
  }
  
  backtrack(0, []);
  return result;
}
```

---

## Challenge 83: Permutations

### Problem
Generate all permutations of an array.

### Solution
```javascript
function permute(nums) {
  const result = [];
  
  function backtrack(current) {
    if (current.length === nums.length) {
      result.push([...current]);
      return;
    }
    
    for (let num of nums) {
      if (current.includes(num)) continue;
      current.push(num);
      backtrack(current);
      current.pop();
    }
  }
  
  backtrack([]);
  return result;
}
```

---

## Challenge 84: N-Queens

### Problem
Solve the N-Queens puzzle.

### Solution
```javascript
function solveNQueens(n) {
  const result = [];
  const board = Array(n).fill(0).map(() => Array(n).fill('.'));
  
  function isValid(row, col) {
    // Check column
    for (let i = 0; i < row; i++) {
      if (board[i][col] === 'Q') return false;
    }
    
    // Check diagonal
    for (let i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
      if (board[i][j] === 'Q') return false;
    }
    
    // Check anti-diagonal
    for (let i = row - 1, j = col + 1; i >= 0 && j < n; i--, j++) {
      if (board[i][j] === 'Q') return false;
    }
    
    return true;
  }
  
  function backtrack(row) {
    if (row === n) {
      result.push(board.map(r => r.join('')));
      return;
    }
    
    for (let col = 0; col < n; col++) {
      if (isValid(row, col)) {
        board[row][col] = 'Q';
        backtrack(row + 1);
        board[row][col] = '.';
      }
    }
  }
  
  backtrack(0);
  return result;
}
```

---

## Challenge 85: Sudoku Solver

### Problem
Solve a Sudoku puzzle.

### Solution
```javascript
function solveSudoku(board) {
  function isValid(row, col, num) {
    // Check row
    for (let j = 0; j < 9; j++) {
      if (board[row][j] === num) return false;
    }
    
    // Check column
    for (let i = 0; i < 9; i++) {
      if (board[i][col] === num) return false;
    }
    
    // Check 3x3 box
    const boxRow = Math.floor(row / 3) * 3;
    const boxCol = Math.floor(col / 3) * 3;
    for (let i = boxRow; i < boxRow + 3; i++) {
      for (let j = boxCol; j < boxCol + 3; j++) {
        if (board[i][j] === num) return false;
      }
    }
    
    return true;
  }
  
  function solve() {
    for (let i = 0; i < 9; i++) {
      for (let j = 0; j < 9; j++) {
        if (board[i][j] === '.') {
          for (let num = 1; num <= 9; num++) {
            const char = num.toString();
            if (isValid(i, j, char)) {
              board[i][j] = char;
              if (solve()) return true;
              board[i][j] = '.';
            }
          }
          return false;
        }
      }
    }
    return true;
  }
  
  solve();
}
```

---

## Challenge 86: Median of Two Sorted Arrays

### Problem
Find the median of two sorted arrays.

### Solution
```javascript
function findMedianSortedArrays(nums1, nums2) {
  if (nums1.length > nums2.length) {
    [nums1, nums2] = [nums2, nums1];
  }
  
  const m = nums1.length;
  const n = nums2.length;
  let low = 0;
  let high = m;
  
  while (low <= high) {
    const partition1 = Math.floor((low + high) / 2);
    const partition2 = Math.floor((m + n + 1) / 2) - partition1;
    
    const maxLeft1 = partition1 === 0 ? -Infinity : nums1[partition1 - 1];
    const minRight1 = partition1 === m ? Infinity : nums1[partition1];
    const maxLeft2 = partition2 === 0 ? -Infinity : nums2[partition2 - 1];
    const minRight2 = partition2 === n ? Infinity : nums2[partition2];
    
    if (maxLeft1 <= minRight2 && maxLeft2 <= minRight1) {
      if ((m + n) % 2 === 0) {
        return (Math.max(maxLeft1, maxLeft2) + Math.min(minRight1, minRight2)) / 2;
      } else {
        return Math.max(maxLeft1, maxLeft2);
      }
    } else if (maxLeft1 > minRight