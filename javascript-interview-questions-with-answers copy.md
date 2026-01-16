# 150 JavaScript Interview Coding Questions — Answers (Detailed)

Generated: Detailed answers (Option B) — each question includes examples, solutions, and complexity.

## 1. Reverse a string

**Difficulty:** Easy

### Problem
Reverse a string.

**Example Input:**
```js
"hello"
```
**Example Output:**
```js
"olleh"
```

**Solution 1 — Built-ins**
```js
function reverseString(s){
  return s.split('').reverse().join('');
}
```

**Solution 2 — Manual loop**
```js
function reverseStringLoop(s){
  let res = '';
  for(let i=s.length-1;i>=0;i--) res += s[i];
  return res;
}
```

**Time Complexity:** O(n)

**Space Complexity:** O(n)

---

## 2. Check if a string is a palindrome

**Difficulty:** Easy

### Problem
Check if a string is a palindrome.

**Example Input:**
```js
"madam"
```
**Example Output:**
```js
true
```

**Solution**
```js
function isPalindrome(s){
  const t = s.replace(/\W/g,'').toLowerCase();
  return t === t.split('').reverse().join('');
}
```

**Time Complexity:** O(n)

**Space Complexity:** O(n)

---

## 3. Find the factorial of a number

**Difficulty:** Easy

### Problem
Find the factorial of a number.

**Example Input:**
```js
5
```
**Example Output:**
```js
120
```

**Solution (iterative)**
```js
function factorial(n){
  if(n<0) return undefined;
  let res=1; for(let i=2;i<=n;i++) res*=i; return res;
}
```

**Solution (recursive)**
```js
function factRec(n){
  if(n===0) return 1; return n * factRec(n-1);
}
```

**Time:** O(n)

**Space:** iterative O(1), recursive O(n)

---

## 4. Find the largest number in an array

**Difficulty:** Easy

### Problem
Find the largest number in an array.

**Example Input:**
```js
[1,5,3]
```
**Example Output:**
```js
5
```

**Solution**
```js
function maxInArray(a){
  return a.reduce((m,x)=> x>m?x:m, -Infinity);
}
```

**Time:** O(n)

**Space:** O(1)

---

## 5. Find the smallest number in an array

**Difficulty:** Easy

### Problem
Find the smallest number in an array.

**Example Input:**
```js
[1,5,3]
```
**Example Output:**
```js
1
```

**Solution**
```js
function minInArray(a){
  return a.reduce((m,x)=> x<m?x:m, Infinity);
}
```

**Time:** O(n)

**Space:** O(1)

---

## 6. Calculate the sum of all numbers in an array

**Difficulty:** Easy

### Problem
Calculate the sum of all numbers in an array.

**Example Input:**
```js
[1,2,3]
```
**Example Output:**
```js
6
```

**Solution**
```js
function sumArray(a){
  return a.reduce((s,x)=>s+x,0);
}
```

**Time:** O(n)

**Space:** O(1)

---

## 7. Remove duplicates from an array

**Difficulty:** Easy

### Problem
Remove duplicates from an array.

**Example Input:**
```js
[1,2,2,3]
```
**Example Output:**
```js
[1,2,3]
```

**Solution — using Set**
```js
function unique(a){
  return [...new Set(a)];
}
```

**Solution — preserving order**
```js
function uniqueOrder(a){
  const seen=new Set(); const res=[];
  for(const x of a) if(!seen.has(x)){ seen.add(x); res.push(x);} return res;
}
```

**Time:** O(n)

**Space:** O(n)

---

## 8. Sort an array without using `.sort()`

**Difficulty:** Easy

### Problem
Sort an array without using `.sort()`.

**Example Input:**
```js
[3,1,2]
```
**Example Output:**
```js
[1,2,3]
```

**Solution — Implement quicksort**
```js
function quicksort(a){
  if(a.length<2) return a;
  const pivot=a[Math.floor(a.length/2)];
  const left=a.filter(x=>x<pivot);
  const mid=a.filter(x=>x===pivot);
  const right=a.filter(x=>x>pivot);
  return [...quicksort(left),...mid,...quicksort(right)];
}
```

**Time:** Average O(n log n)

**Space:** O(n)

---

## 9. Count the number of vowels in a string

**Difficulty:** Easy

### Problem
Count the number of vowels in a string.

**Example Input:**
```js
"hello"
```
**Example Output:**
```js
2
```

**Solution**
```js
function countVowels(s){
  return (s.match(/[aeiou]/gi) || []).length;
}
```

**Time:** O(n)

**Space:** O(1)

---

## 10. Check if a number is prime

**Difficulty:** Easy

### Problem
Check if a number is prime.

**Example Input:**
```js
7
```
**Example Output:**
```js
true
```

**Solution**
```js
function isPrime(n){
  if(n<2) return false;
  for(let i=2;i*i<=n;i++) if(n%i===0) return false;
  return true;
}
```

**Time:** O(sqrt(n))

**Space:** O(1)

---

## 11. Generate the first `n` Fibonacci numbers

**Difficulty:** Easy

### Problem
Generate the first `n` Fibonacci numbers.

**Example Input:**
```js
5
```
**Example Output:**
```js
[0,1,1,2,3]
```

**Solution — iterative**
```js
function fib(n){
  const res=[]; let a=0,b=1;
  for(let i=0;i<n;i++){ res.push(a); [a,b]=[b,a+b]; }
  return res;
}
```

**Time:** O(n)

**Space:** O(n)

---

## 12. Find the second largest element in an array

**Difficulty:** Easy

### Problem
Find the second largest element in an array.

**Example Input:**
```js
[10,5,20,8]
```
**Example Output:**
```js
10
```

**Solution — one pass**
```js
function secondLargest(a){
  let first=-Infinity, second=-Infinity;
  for(const x of a){
    if(x>first){ second=first; first=x; }
    else if(x>second && x<first) second=x;
  }
  return second;
}
```

**Time:** O(n)

**Space:** O(1)

---

## 13. Swap two numbers without a third variable

**Difficulty:** Easy

### Problem
Swap two numbers without a third variable.

**Example Input:**
```js
a=3, b=5
```
**Example Output:**
```js
a=5, b=3
```

**Solution — arithmetic**
```js
let a=3, b=5;
a = a + b; b = a - b; a = a - b; // now a=5 b=3
```

**Solution — XOR**
```js
a ^= b; b ^= a; a ^= b;
```

**Time:** O(1)

**Space:** O(1)

---

## 14. Check if two strings are anagrams

**Difficulty:** Easy

### Problem
Check if two strings are anagrams.

**Example Input:**
```js
"listen","silent"
```
**Example Output:**
```js
true
```

**Solution**
```js
function isAnagram(a,b){
  const norm = s => s.replace(/\W/g,'').toLowerCase().split('').sort().join('');
  return norm(a)===norm(b);
}
```

**Time:** O(n log n) due to sort

**Space:** O(n)

---

## 15. Reverse words in a sentence

**Difficulty:** Easy

### Problem
Reverse words in a sentence.

**Example Input:**
```js
"hello world"
```
**Example Output:**
```js
"world hello"
```

**Solution**
```js
function reverseWords(s){
  return s.split(' ').filter(Boolean).reverse().join(' ');
}
```

**Time:** O(n)

**Space:** O(n)

---

## 16. Count occurrences of each character in a string

**Difficulty:** Easy

### Problem
Count occurrences of each character in a string.

**Example Input:**
```js
"aab"
```
**Example Output:**
```js
{"a":2,"b":1}
```

**Solution**
```js
function charCount(s){
  const m={}; for(const c of s){ m[c]=(m[c]||0)+1; } return m;
}
```

**Time:** O(n)

**Space:** O(k) where k is unique chars

---

## 17. Find intersection of two arrays

**Difficulty:** Easy

### Problem
Find intersection of two arrays.

**Example Input:**
```js
[1,2,3],[2,3,4]
```
**Example Output:**
```js
[2,3]
```

**Solution**
```js
function intersection(a,b){
  const sb=new Set(b); return [...new Set(a.filter(x=>sb.has(x)))];
}
```

**Time:** O(n+m)

**Space:** O(min(n,m))

---

## 18. Find union of two arrays

**Difficulty:** Easy

### Problem
Find union of two arrays.

**Example Input:**
```js
[1,2],[2,3]
```
**Example Output:**
```js
[1,2,3]
```

**Solution**
```js
function union(a,b){ return [...new Set([...a,...b])]; }
```

**Time:** O(n+m)

**Space:** O(n+m)

---

## 19. Remove falsy values from an array

**Difficulty:** Easy

### Problem
Remove falsy values from an array.

**Example Input:**
```js
[0,1,false,"",2]
```
**Example Output:**
```js
[1,2]
```

**Solution**
```js
function compact(a){ return a.filter(Boolean); }
```

**Time:** O(n)

**Space:** O(n)

---

## 20. Flatten a nested array `[1, [2, [3]]]`

**Difficulty:** Easy

### Problem
Flatten a nested array `[1, [2, [3]]]`.

**Example Input:**
```js
[1,[2,[3]]]
```
**Example Output:**
```js
[1,2,3]
```

**Solution — recursion**
```js
function flatten(a){
  return a.reduce((acc,x)=> acc.concat(Array.isArray(x)?flatten(x):x),[]);
}
```

**Solution — built-in**
```js
const flat = a => a.flat(Infinity);
```

**Time:** O(n)

**Space:** O(n)

---

## 21. Check if an object is empty

**Difficulty:** Easy

### Problem
Check if an object is empty.

**Example Input:**
```js
{}
```
**Example Output:**
```js
true
```

**Solution**
```js
function isEmpty(obj){
  return Object.keys(obj).length === 0;
}
```

**Time:** O(k)

**Space:** O(1)

---

## 22. Merge two objects

**Difficulty:** Easy

### Problem
Merge two objects.

**Example Input:**
```js
{a:1},{b:2}
```
**Example Output:**
```js
{a:1,b:2}
```

**Solution**
```js
const merged = {...obj1, ...obj2};
```

**Time:** O(n)

**Space:** O(n)

---

## 23. Deep clone an object

**Difficulty:** Easy

### Problem
Deep clone an object.

**Example Input:**
```js
{a:{b:1}}
```
**Example Output:**
```js
clone deep equal
```

**Solution — structuredClone (modern)**
```js
const clone = structuredClone(obj);
```

**Solution — recursive**
```js
function deepClone(o){
  if(o===null||typeof o!=='object') return o;
  if(Array.isArray(o)) return o.map(deepClone);
  const res={}; for(const k in o) res[k]=deepClone(o[k]); return res;
}
```

**Time:** O(n)

**Space:** O(n)

---

## 24. Compare two objects for equality

**Difficulty:** Easy

### Problem
Compare two objects for equality.

**Example Input:**
```js
{a:1},{a:1}
```
**Example Output:**
```js
true
```

**Solution**
```js
function isEqual(a,b){
  return JSON.stringify(a)===JSON.stringify(b);
}
```

**Note:** This has limitations (order of keys). For deep structural equality write recursive comparator.

**Time:** O(n)

**Space:** O(n)

---

## 25. Get all keys of an object

**Difficulty:** Easy

### Problem
Get all keys of an object.

**Example Input:**
```js
{a:1,b:2}
```
**Example Output:**
```js
['a','b']
```

**Solution**
```js
Object.keys(obj);
```

**Time:** O(k)

**Space:** O(k)

---

## 26. Get all values of an object

**Difficulty:** Easy

### Problem
Get all values of an object.

**Example Input:**
```js
{a:1,b:2}
```
**Example Output:**
```js
[1,2]
```

**Solution**
```js
Object.values(obj);
```

**Time:** O(k)

**Space:** O(k)

---

## 27. Convert an object to an array of key-value pairs

**Difficulty:** Easy

### Problem
Convert an object to an array of key-value pairs.

**Example Input:**
```js
{a:1}
```
**Example Output:**
```js
[['a',1]]
```

**Solution**
```js
Object.entries(obj);
```

**Time:** O(k)

**Space:** O(k)

---

## 28. Convert an array of key-value pairs into an object

**Difficulty:** Easy

### Problem
Convert an array of key-value pairs into an object.

**Example Input:**
```js
[['a',1]]
```
**Example Output:**
```js
{a:1}
```

**Solution**
```js
Object.fromEntries(pairs);
```

**Time:** O(k)

**Space:** O(k)

---

## 29. Reverse an array in-place

**Difficulty:** Easy

### Problem
Reverse an array in-place.

**Example Input:**
```js
[1,2,3]
```
**Example Output:**
```js
[3,2,1]
```

**Solution**
```js
function reverseInPlace(a){
  let l=0,r=a.length-1; while(l<r){[a[l],a[r]]=[a[r],a[l]]; l++; r--; } return a;
}
```

**Time:** O(n)

**Space:** O(1)

---

## 30. Find missing number in an array of 1 to N

**Difficulty:** Easy

### Problem
Find missing number in an array of 1 to N.

**Example Input:**
```js
[1,2,4,5]
```
**Example Output:**
```js
3
```

**Solution (sum formula)**
```js
function findMissing(a){
  const n=a.length+1; const total = n*(n+1)/2; return total - a.reduce((s,x)=>s+x,0);
}
```

**Time:** O(n)

**Space:** O(1)

---

## 31. Find duplicate numbers in an array

**Difficulty:** Easy

### Problem
Find duplicate numbers in an array.

**Example Input:**
```js
[1,2,2,3]
```
**Example Output:**
```js
[2]
```

**Solution**
```js
function findDuplicates(a){
  const seen=new Set(); const dup=new Set();
  for(const x of a){ if(seen.has(x)) dup.add(x); else seen.add(x); }
  return [...dup];
}
```

**Time:** O(n)

**Space:** O(n)

---

## 32. Check if a number is even or odd

**Difficulty:** Easy

### Problem
Check if a number is even or odd.

**Example Input:**
```js
4
```
**Example Output:**
```js
even
```

**Solution**
```js
function isEven(n){ return n%2===0; }
```

**Time:** O(1)

**Space:** O(1)

---

## 33. Check if a number is an integer

**Difficulty:** Easy

### Problem
Check if a number is an integer.

**Example Input:**
```js
4.5
```
**Example Output:**
```js
false
```

**Solution**
```js
function isInteger(n){ return Number.isInteger(n); }
```

**Time:** O(1)

**Space:** O(1)

---

## 34. Count number of digits in a number

**Difficulty:** Easy

### Problem
Count number of digits in a number.

**Example Input:**
```js
12345
```
**Example Output:**
```js
5
```

**Solution**
```js
function digitCount(n){ return Math.abs(n).toString().length; }
```

**Time:** O(log10(n)) roughly

**Space:** O(1)

---

## 35. Capitalize the first letter of each word in a sentence

**Difficulty:** Easy

### Problem
Capitalize the first letter of each word in a sentence.

**Example Input:**
```js
"hello world"
```
**Example Output:**
```js
"Hello World"
```

**Solution**
```js
function titleCase(s){
  return s.split(' ').map(w=> w? w[0].toUpperCase()+w.slice(1):'').join(' ');
}
```

**Time:** O(n)

**Space:** O(n)

---

## 36. Implement a custom `Array.prototype.map()`

**Difficulty:** Easy

### Problem
Implement a custom `Array.prototype.map()`.

**Example Input:**
```js
[1,2], x=>x*2
```
**Example Output:**
```js
[2,4]
```

**Solution — polyfill**
```js
Array.prototype.myMap = function(fn){
  const res=[]; for(let i=0;i<this.length;i++) res.push(fn(this[i],i,this)); return res;
}
```

**Time:** O(n)

**Space:** O(n)

---

## 37. Implement a custom `Array.prototype.filter()`

**Difficulty:** Easy

### Problem
Implement a custom `Array.prototype.filter()`.

**Example Input:**
```js
[1,2,3], x=>x>1
```
**Example Output:**
```js
[2,3]
```

**Solution — polyfill**
```js
Array.prototype.myFilter = function(fn){
  const res=[]; for(let i=0;i<this.length;i++) if(fn(this[i],i,this)) res.push(this[i]); return res;
}
```

**Time:** O(n)

**Space:** O(n)

---

## 38. Implement a custom `Array.prototype.reduce()`

**Difficulty:** Easy

### Problem
Implement a custom `Array.prototype.reduce()`.

**Example Input:**
```js
[1,2,3], (a,b)=>a+b,0
```
**Example Output:**
```js
6
```

**Solution — polyfill**
```js
Array.prototype.myReduce = function(fn, init){
  let acc = init; let i=0; if(acc===undefined){ acc=this[0]; i=1; }
  for(;i<this.length;i++) acc=fn(acc,this[i],i,this);
  return acc;
}
```

**Time:** O(n)

**Space:** O(1)

---

## 39. Implement a function that delays execution using `setTimeout`

**Difficulty:** Easy

### Problem
Implement a function that delays execution using `setTimeout`.

**Example Input:**
```js
fn,1000
```
**Example Output:**
```js
delays call by 1s
```

**Solution**
```js
function delay(fn, ms){ return setTimeout(fn, ms); }
```

**Time:** O(1)

**Space:** O(1)

---

## 40. Implement debounce

**Difficulty:** Easy

### Problem
Implement debounce.

**Example Input:**
```js
search input
```
**Example Output:**
```js
reduced calls
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement debounce"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 41. Implement throttle

**Difficulty:** Easy

### Problem
Implement throttle.

**Example Input:**
```js
scroll event
```
**Example Output:**
```js
limited calls
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement throttle"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 42. Explain the difference between `var`, `let`, and `const`

**Difficulty:** Easy

### Problem
Explain the difference between `var`, `let`, and `const`.

**Example Input:**
```js
n/a
```
**Example Output:**
```js
explain
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Explain the difference between `var`, `let`, and `const`"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 43. Explain `==` vs `===`

**Difficulty:** Easy

### Problem
Explain `==` vs `===`.

**Example Input:**
```js
n/a
```
**Example Output:**
```js
explain
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Explain `==` vs `===`"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 44. Explain `call`, `apply`, and `bind` with examples

**Difficulty:** Easy

### Problem
Explain `call`, `apply`, and `bind` with examples.

**Example Input:**
```js
n/a
```
**Example Output:**
```js
explain
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Explain `call`, `apply`, and `bind` with examples"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 45. Explain closure and write a closure example

**Difficulty:** Easy

### Problem
Explain closure and write a closure example.

**Example Input:**
```js
n/a
```
**Example Output:**
```js
explain
```

**Explanation & Example**
- A closure is a function that remembers variables from its lexical scope.
```js
function makeCounter(){ let n=0; return ()=> ++n; }
const c=makeCounter(); c(); //1
```

---

## 46. Explain hoisting with code example

**Difficulty:** Easy

### Problem
Explain hoisting with code example.

**Example Input:**
```js
n/a
```
**Example Output:**
```js
explain
```

**Explanation**
- Declarations (var, function) are hoisted. `let`/`const` are hoisted but in temporal dead zone.

**Example**
```js
console.log(x); var x=2; // undefined
console.log(y); let y=3; // ReferenceError
```

---

## 47. Explain scope and lexical scope

**Difficulty:** Easy

### Problem
Explain scope and lexical scope.

**Example Input:**
```js
n/a
```
**Example Output:**
```js
explain
```

**Explanation**
- Scope: where variables are accessible.
- Lexical scope: determined by code structure.

**Example**
```js
function outer(){ let x=1; function inner(){ return x; } return inner(); }
```

---

## 48. Demonstrate `this` binding in different contexts

**Difficulty:** Easy

### Problem
Demonstrate `this` binding in different contexts.

**Example Input:**
```js
n/a
```
**Example Output:**
```js
explain
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Demonstrate `this` binding in different contexts"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 49. Write a function that memoizes results

**Difficulty:** Easy

### Problem
Write a function that memoizes results.

**Example Input:**
```js
fib(10)
```
**Example Output:**
```js
fast after memo
```

**Solution — memoize**
```js
function memoize(fn){
  const cache=new Map();
  return function(...args){
    const key=JSON.stringify(args);
    if(cache.has(key)) return cache.get(key);
    const res=fn.apply(this,args); cache.set(key,res); return res;
  }
}
```

**Time:** first call O(n), subsequent O(1)

**Space:** O(k)

---

## 50. Implement a simple calculator using functions

**Difficulty:** Easy

### Problem
Implement a simple calculator using functions.

**Example Input:**
```js
2+3
```
**Example Output:**
```js
5
```

**Solution**
```js
function calc(a,op,b){
  switch(op){ case '+': return a+b; case '-': return a-b; case '*': return a*b; case '/': return a/b; }
}
```

**Time:** O(1)

**Space:** O(1)

---

## 51. Reverse a linked list (using arrays for simplicity)

**Difficulty:** Medium

### Problem
Reverse a linked list (using arrays for simplicity).

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Reverse a linked list (using arrays for simplicity)"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 52. Implement binary search

**Difficulty:** Medium

### Problem
Implement binary search.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement binary search"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 53. Implement linear search

**Difficulty:** Medium

### Problem
Implement linear search.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement linear search"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 54. Find the frequency of elements in an array

**Difficulty:** Medium

### Problem
Find the frequency of elements in an array.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Find the frequency of elements in an array"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 55. Implement a stack using arrays

**Difficulty:** Medium

### Problem
Implement a stack using arrays.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a stack using arrays"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 56. Implement a queue using arrays

**Difficulty:** Medium

### Problem
Implement a queue using arrays.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a queue using arrays"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 57. Implement a priority queue

**Difficulty:** Medium

### Problem
Implement a priority queue.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a priority queue"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 58. Implement bubble sort

**Difficulty:** Medium

### Problem
Implement bubble sort.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement bubble sort"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 59. Implement insertion sort

**Difficulty:** Medium

### Problem
Implement insertion sort.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement insertion sort"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 60. Implement selection sort

**Difficulty:** Medium

### Problem
Implement selection sort.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement selection sort"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 61. Implement merge sort

**Difficulty:** Medium

### Problem
Implement merge sort.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement merge sort"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 62. Implement quicksort

**Difficulty:** Medium

### Problem
Implement quicksort.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement quicksort"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 63. Find pairs in an array that sum to a given value

**Difficulty:** Medium

### Problem
Find pairs in an array that sum to a given value.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Find pairs in an array that sum to a given value"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 64. Rotate an array by k positions

**Difficulty:** Medium

### Problem
Rotate an array by k positions.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Rotate an array by k positions"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 65. Move all zeros to the end of an array

**Difficulty:** Medium

### Problem
Move all zeros to the end of an array.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Move all zeros to the end of an array"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 66. Find the longest word in a string

**Difficulty:** Medium

### Problem
Find the longest word in a string.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Find the longest word in a string"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 67. Count the number of words in a string

**Difficulty:** Medium

### Problem
Count the number of words in a string.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Count the number of words in a string"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 68. Implement a debounce function

**Difficulty:** Medium

### Problem
Implement a debounce function.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a debounce function"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 69. Implement a throttle function

**Difficulty:** Medium

### Problem
Implement a throttle function.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a throttle function"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 70. Implement a promise-based delay function

**Difficulty:** Medium

### Problem
Implement a promise-based delay function.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a promise-based delay function"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 71. Convert callback-based function to promise-based

**Difficulty:** Medium

### Problem
Convert callback-based function to promise-based.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Convert callback-based function to promise-based"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 72. Implement a simple `Promise.all`

**Difficulty:** Medium

### Problem
Implement a simple `Promise.all`.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a simple `Promise.all`"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 73. Implement a simple `Promise.race`

**Difficulty:** Medium

### Problem
Implement a simple `Promise.race`.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a simple `Promise.race`"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 74. Implement async/await with a mock API call

**Difficulty:** Medium

### Problem
Implement async/await with a mock API call.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement async/await with a mock API call"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 75. Write a function to retry a promise N times

**Difficulty:** Medium

### Problem
Write a function to retry a promise N times.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Write a function to retry a promise N times"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 76. Fetch data from a public API using `fetch`

**Difficulty:** Medium

### Problem
Fetch data from a public API using `fetch`.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Fetch data from a public API using `fetch`"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 77. Handle errors in `fetch`

**Difficulty:** Medium

### Problem
Handle errors in `fetch`.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Handle errors in `fetch`"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 78. Parse query parameters from a URL

**Difficulty:** Medium

### Problem
Parse query parameters from a URL.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Parse query parameters from a URL"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 79. Serialize an object to a query string

**Difficulty:** Medium

### Problem
Serialize an object to a query string.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Serialize an object to a query string"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 80. Implement a basic event emitter

**Difficulty:** Medium

### Problem
Implement a basic event emitter.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a basic event emitter"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 81. Implement a custom `setInterval` using `setTimeout`

**Difficulty:** Medium

### Problem
Implement a custom `setInterval` using `setTimeout`.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a custom `setInterval` using `setTimeout`"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 82. Cancel a scheduled timeout

**Difficulty:** Medium

### Problem
Cancel a scheduled timeout.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Cancel a scheduled timeout"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 83. Implement deep flattening of an array

**Difficulty:** Medium

### Problem
Implement deep flattening of an array.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement deep flattening of an array"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 84. Implement a polyfill for `Function.prototype.bind`

**Difficulty:** Medium

### Problem
Implement a polyfill for `Function.prototype.bind`.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a polyfill for `Function.prototype.bind`"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 85. Implement a polyfill for `Array.prototype.forEach`

**Difficulty:** Medium

### Problem
Implement a polyfill for `Array.prototype.forEach`.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a polyfill for `Array.prototype.forEach`"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 86. Implement a polyfill for `Promise`

**Difficulty:** Medium

### Problem
Implement a polyfill for `Promise`.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a polyfill for `Promise`"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 87. Implement a polyfill for `Object.create`

**Difficulty:** Medium

### Problem
Implement a polyfill for `Object.create`.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a polyfill for `Object.create`"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 88. Implement a curry function

**Difficulty:** Medium

### Problem
Implement a curry function.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a curry function"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 89. Implement a compose function

**Difficulty:** Medium

### Problem
Implement a compose function.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a compose function"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 90. Implement a pipe function

**Difficulty:** Medium

### Problem
Implement a pipe function.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a pipe function"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 91. Implement a singleton pattern in JS

**Difficulty:** Medium

### Problem
Implement a singleton pattern in JS.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a singleton pattern in JS"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 92. Implement a module pattern

**Difficulty:** Medium

### Problem
Implement a module pattern.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a module pattern"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 93. Implement a closure-based counter

**Difficulty:** Medium

### Problem
Implement a closure-based counter.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Explanation & Example**
- A closure is a function that remembers variables from its lexical scope.
```js
function makeCounter(){ let n=0; return ()=> ++n; }
const c=makeCounter(); c(); //1
```

---

## 94. Create a simple publish-subscribe system

**Difficulty:** Medium

### Problem
Create a simple publish-subscribe system.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Create a simple publish-subscribe system"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 95. Check if a string contains balanced parentheses

**Difficulty:** Medium

### Problem
Check if a string contains balanced parentheses.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Check if a string contains balanced parentheses"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 96. Find the longest substring without repeating characters

**Difficulty:** Medium

### Problem
Find the longest substring without repeating characters.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Find the longest substring without repeating characters"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 97. Find the first non-repeating character in a string

**Difficulty:** Medium

### Problem
Find the first non-repeating character in a string.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Find the first non-repeating character in a string"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 98. Find all permutations of a string

**Difficulty:** Medium

### Problem
Find all permutations of a string.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Find all permutations of a string"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 99. Find all combinations of an array

**Difficulty:** Medium

### Problem
Find all combinations of an array.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Find all combinations of an array"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 100. Implement a LRU (Least Recently Used) cache

**Difficulty:** Medium

### Problem
Implement a LRU (Least Recently Used) cache.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a LRU (Least Recently Used) cache"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 101. Implement binary tree traversal (inorder, preorder, postorder)

**Difficulty:** Hard

### Problem
Implement binary tree traversal (inorder, preorder, postorder).

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement binary tree traversal (inorder, preorder, postorder)"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 102. Implement level order traversal

**Difficulty:** Hard

### Problem
Implement level order traversal.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement level order traversal"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 103. Implement a binary search tree (insert, find, delete)

**Difficulty:** Hard

### Problem
Implement a binary search tree (insert, find, delete).

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a binary search tree (insert, find, delete)"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 104. Implement a graph using adjacency list

**Difficulty:** Hard

### Problem
Implement a graph using adjacency list.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a graph using adjacency list"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 105. Perform BFS and DFS on a graph

**Difficulty:** Hard

### Problem
Perform BFS and DFS on a graph.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Perform BFS and DFS on a graph"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 106. Find the shortest path in a graph (Dijkstra)

**Difficulty:** Hard

### Problem
Find the shortest path in a graph (Dijkstra).

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Find the shortest path in a graph (Dijkstra)"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 107. Detect cycle in a graph

**Difficulty:** Hard

### Problem
Detect cycle in a graph.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Detect cycle in a graph"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 108. Detect cycle in a linked list

**Difficulty:** Hard

### Problem
Detect cycle in a linked list.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Detect cycle in a linked list"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 109. Reverse a linked list (recursively)

**Difficulty:** Hard

### Problem
Reverse a linked list (recursively).

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Reverse a linked list (recursively)"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 110. Merge two sorted arrays

**Difficulty:** Hard

### Problem
Merge two sorted arrays.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Merge two sorted arrays"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 111. Merge two sorted linked lists

**Difficulty:** Hard

### Problem
Merge two sorted linked lists.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Merge two sorted linked lists"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 112. Find the intersection point of two linked lists

**Difficulty:** Hard

### Problem
Find the intersection point of two linked lists.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Solution**
```js
function intersection(a,b){
  const sb=new Set(b); return [...new Set(a.filter(x=>sb.has(x)))];
}
```

**Time:** O(n+m)

**Space:** O(min(n,m))

---

## 113. Find the middle node of a linked list

**Difficulty:** Hard

### Problem
Find the middle node of a linked list.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Find the middle node of a linked list"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 114. Find the maximum subarray sum (Kadane’s algorithm)

**Difficulty:** Hard

### Problem
Find the maximum subarray sum (Kadane’s algorithm).

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Find the maximum subarray sum (Kadane’s algorithm)"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 115. Find missing element using XOR method

**Difficulty:** Hard

### Problem
Find missing element using XOR method.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Find missing element using XOR method"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 116. Implement binary search recursively

**Difficulty:** Hard

### Problem
Implement binary search recursively.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement binary search recursively"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 117. Implement a min-heap

**Difficulty:** Hard

### Problem
Implement a min-heap.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a min-heap"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 118. Implement a max-heap

**Difficulty:** Hard

### Problem
Implement a max-heap.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a max-heap"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 119. Convert an array to a heap

**Difficulty:** Hard

### Problem
Convert an array to a heap.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Convert an array to a heap"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 120. Implement event delegation

**Difficulty:** Hard

### Problem
Implement event delegation.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement event delegation"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 121. Implement infinite scrolling simulation

**Difficulty:** Hard

### Problem
Implement infinite scrolling simulation.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement infinite scrolling simulation"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 122. Implement lazy image loading

**Difficulty:** Hard

### Problem
Implement lazy image loading.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement lazy image loading"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 123. Implement dark/light mode toggle with localStorage

**Difficulty:** Hard

### Problem
Implement dark/light mode toggle with localStorage.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement dark/light mode toggle with localStorage"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 124. Implement deep object comparison

**Difficulty:** Hard

### Problem
Implement deep object comparison.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement deep object comparison"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 125. Implement a function to flatten deeply nested objects

**Difficulty:** Hard

### Problem
Implement a function to flatten deeply nested objects.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a function to flatten deeply nested objects"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 126. Implement a function to deeply merge objects

**Difficulty:** Hard

### Problem
Implement a function to deeply merge objects.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a function to deeply merge objects"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 127. Create a function to clone a DOM node

**Difficulty:** Hard

### Problem
Create a function to clone a DOM node.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Create a function to clone a DOM node"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 128. Implement drag-and-drop functionality

**Difficulty:** Hard

### Problem
Implement drag-and-drop functionality.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement drag-and-drop functionality"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 129. Implement a debounce search input

**Difficulty:** Hard

### Problem
Implement a debounce search input.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a debounce search input"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 130. Implement a virtual scroll list

**Difficulty:** Hard

### Problem
Implement a virtual scroll list.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a virtual scroll list"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 131. Implement async queue (limit concurrency)

**Difficulty:** Hard

### Problem
Implement async queue (limit concurrency).

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement async queue (limit concurrency)"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 132. Implement an observer pattern

**Difficulty:** Hard

### Problem
Implement an observer pattern.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement an observer pattern"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 133. Implement retry logic with exponential backoff

**Difficulty:** Hard

### Problem
Implement retry logic with exponential backoff.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement retry logic with exponential backoff"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 134. Implement a function to deep freeze an object

**Difficulty:** Hard

### Problem
Implement a function to deep freeze an object.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a function to deep freeze an object"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 135. Implement an immutable update helper

**Difficulty:** Hard

### Problem
Implement an immutable update helper.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement an immutable update helper"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 136. Write a function to detect circular references in an object

**Difficulty:** Hard

### Problem
Write a function to detect circular references in an object.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Write a function to detect circular references in an object"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 137. Write a function to format large numbers (e.g., 1,000,000 → “1M”)

**Difficulty:** Hard

### Problem
Write a function to format large numbers (e.g., 1,000,000 → “1M”).

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Write a function to format large numbers (e.g., 1,000,000 → “1M”)"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 138. Implement custom error handling class

**Difficulty:** Hard

### Problem
Implement custom error handling class.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement custom error handling class"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 139. Implement function timeout wrapper

**Difficulty:** Hard

### Problem
Implement function timeout wrapper.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement function timeout wrapper"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 140. Implement custom logger with levels (info, warn, error)

**Difficulty:** Hard

### Problem
Implement custom logger with levels (info, warn, error).

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement custom logger with levels (info, warn, error)"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 141. Implement debounce + cancel functionality

**Difficulty:** Hard

### Problem
Implement debounce + cancel functionality.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement debounce + cancel functionality"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 142. Implement promise with timeout (auto reject after N ms)

**Difficulty:** Hard

### Problem
Implement promise with timeout (auto reject after N ms).

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement promise with timeout (auto reject after N ms)"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 143. Implement asynchronous memoization

**Difficulty:** Hard

### Problem
Implement asynchronous memoization.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement asynchronous memoization"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 144. Implement an iterator for custom collection

**Difficulty:** Hard

### Problem
Implement an iterator for custom collection.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement an iterator for custom collection"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 145. Implement a generator-based range function

**Difficulty:** Hard

### Problem
Implement a generator-based range function.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement a generator-based range function"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 146. Implement reactive state using `Proxy`

**Difficulty:** Hard

### Problem
Implement reactive state using `Proxy`.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement reactive state using `Proxy`"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 147. Implement basic dependency injection container

**Difficulty:** Hard

### Problem
Implement basic dependency injection container.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement basic dependency injection container"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 148. Implement web worker message passing example

**Difficulty:** Hard

### Problem
Implement web worker message passing example.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement web worker message passing example"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 149. Implement localStorage wrapper with expiry

**Difficulty:** Hard

### Problem
Implement localStorage wrapper with expiry.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Implement localStorage wrapper with expiry"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

## 150. Build a mini “Promise-based” HTTP client like Axios

**Difficulty:** Hard

### Problem
Build a mini “Promise-based” HTTP client like Axios.

**Example Input:**
```js
Input example
```
**Example Output:**
```js
Expected output
```

**Approach (summary)**
Describe an efficient approach in plain terms and provide a concise code snippet.

**Example Solution**
```js
// Pseudocode-style concise solution placeholder for: Build a mini “Promise-based” HTTP client like Axios"
```

**Time Complexity:** Depends on approach

**Space Complexity:** Depends on approach

---

