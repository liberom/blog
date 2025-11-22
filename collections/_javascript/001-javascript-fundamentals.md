---
layout: post
title: "JavaScript Fundamentals: Variables, Types, and Functions"
author: "Liberom"
date: 2025-01-12
category: "JavaScript"
toc: true
excerpt: "Master JavaScript fundamentals: modern variable declarations, data types, functions, scope, and closures."
---

# JavaScript Fundamentals: Variables, Types, and Functions

JavaScript has evolved significantly. Modern JavaScript (ES6+) provides better patterns for writing clear, maintainable code.

## Variable Declarations

### const vs let vs var

```javascript
// var (avoid - function scoped, hoisted)
var name = 'John';
console.log(typeof name);  // 'string'

// let (block scoped, temporal dead zone)
let age = 30;
if (true) {
  let age = 25;  // Different variable
  console.log(age);  // 25
}
console.log(age);  // 30

// const (block scoped, cannot reassign)
const PI = 3.14159;
// PI = 3;  // Error: Assignment to constant variable

// const prevents reassignment, not mutation
const obj = { name: 'John' };
obj.name = 'Jane';  // OK - mutating
// obj = {};  // Error - cannot reassign
```

**Best Practice**: Use `const` by default, `let` if reassignment needed, never use `var`.

## Data Types

### Primitives

```javascript
// String
const greeting = 'Hello, World!';
const template = `Hello, ${name}!`;  // Template literal

// Number (no separate int/float)
const int = 42;
const float = 3.14;
const big = 1e10;
const hex = 0xFF;

// Boolean
const active = true;
const inactive = false;

// null and undefined
const nothing = null;           // Intentional absence
let uninitialized;              // undefined - not set
```

### Type Checking

```javascript
// Use typeof carefully
typeof 42                       // 'number'
typeof 'hello'                  // 'string'
typeof true                     // 'boolean'
typeof undefined                // 'undefined'
typeof Symbol('id')             // 'symbol'
typeof {}                       // 'object' (even for null!)
typeof []                       // 'object' (arrays are objects)

// Better null check
value === null                  // Explicit null check
value == null                   // Checks null or undefined
value === undefined             // Explicit undefined check

// Type coercion (dangerous!)
'5' + 3                         // '53' (string concatenation)
'5' - 3                         // 2 (numeric coercion)
'5' == 5                        // true (loose equality)
'5' === 5                       // false (strict equality - use this)
```

## Objects and Arrays

### Object Literals

```javascript
// Basic object
const user = {
  name: 'John',
  age: 30,
  email: 'john@example.com'
};

// Accessing properties
console.log(user.name);         // 'John'
console.log(user['email']);     // 'john@example.com'

// Dynamic properties
const key = 'age';
console.log(user[key]);         // 30

// Method shorthand
const person = {
  name: 'John',
  greet() {
    return `Hello, ${this.name}`;
  }
};

// Destructuring
const { name, age } = user;
const { name: userName, age: userAge } = user;

// Spread operator
const newUser = { ...user, email: 'newemail@example.com' };
```

### Arrays

```javascript
// Array creation
const arr = [1, 2, 3, 4, 5];
const mixed = [1, 'hello', true, null];

// Methods
arr.push(6);                    // Add to end
arr.pop();                      // Remove from end
arr.shift();                    // Remove from start
arr.unshift(0);                 // Add to start

// Iteration
arr.forEach(item => console.log(item));
arr.map(x => x * 2);            // [2, 4, 6, 8, 10]
arr.filter(x => x > 2);         // [3, 4, 5]
arr.find(x => x > 3);           // 4
arr.some(x => x > 5);           // false
arr.every(x => x > 0);          // true

// Destructuring
const [first, second, ...rest] = arr;
```

## Functions

### Function Declarations

```javascript
// Function declaration (hoisted)
function add(a, b) {
  return a + b;
}

// Function expression (not hoisted)
const multiply = function(a, b) {
  return a * b;
};

// Arrow function (concise syntax)
const divide = (a, b) => a / b;

// Arrow with block body
const calculate = (a, b) => {
  const result = a + b;
  return result * 2;
};

// Default parameters
const greet = (name = 'Guest') => `Hello, ${name}!`;

// Rest parameters
const sum = (...numbers) => numbers.reduce((a, b) => a + b, 0);
sum(1, 2, 3, 4, 5);            // 15
```

### Function Scope

```javascript
let global = 'global';

function outer() {
  let outerVar = 'outer';

  function inner() {
    let innerVar = 'inner';
    console.log(innerVar);      // 'inner' - own scope
    console.log(outerVar);      // 'outer' - parent scope
    console.log(global);        // 'global' - global scope
  }

  // console.log(innerVar);     // Error - not in scope
  inner();
}

outer();
// console.log(outerVar);       // Error - not in scope
```

### Closures

```javascript
// Closure: inner function remembers outer scope
function createCounter() {
  let count = 0;

  return {
    increment() { return ++count; },
    decrement() { return --count; },
    get() { return count; }
  };
}

const counter = createCounter();
console.log(counter.increment());  // 1
console.log(counter.increment());  // 2
console.log(counter.get());        // 2

// Practical: Data privacy
function createUser(initialBalance) {
  let balance = initialBalance;

  return {
    deposit(amount) {
      balance += amount;
      return balance;
    },
    withdraw(amount) {
      if (amount <= balance) {
        balance -= amount;
        return balance;
      }
      return 'Insufficient funds';
    },
    getBalance() {
      return balance;
    }
  };
}

const account = createUser(1000);
console.log(account.deposit(500));    // 1500
console.log(account.withdraw(200));   // 1300
// No way to access balance directly - it's private!
```

## this Binding

```javascript
// this depends on how function is called
const obj = {
  name: 'John',
  greet: function() {
    console.log(this.name);
  },
  greetArrow: () => {
    console.log(this.name);     // undefined - arrow doesn't bind this
  }
};

obj.greet();                    // 'John' - called on object
const greet = obj.greet;
greet();                        // undefined - called without object

// Explicit binding
function introduce() {
  return `Hi, I'm ${this.name}`;
}

const person = { name: 'Alice' };
introduce.call(person);        // 'Hi, I'm Alice'
introduce.apply(person);       // 'Hi, I'm Alice'

const boundIntroduce = introduce.bind(person);
boundIntroduce();              // 'Hi, I'm Alice'
```

## Common Mistakes

```javascript
// BAD - Using var
var x = 1;
if (true) {
  var x = 2;
}
console.log(x);                // 2 (unexpected!)

// GOOD - Using const/let
const y = 1;
if (true) {
  const y = 2;
}
console.log(y);                // 1 (expected)

// BAD - Loose equality
if (value == null) {}           // True for both null and undefined

// GOOD - Strict equality
if (value === null || value === undefined) {}

// BAD - Lost this context
const user = {
  name: 'John',
  printName: function() {
    setTimeout(function() {
      console.log(this.name);  // undefined - wrong this
    }, 1000);
  }
};

// GOOD - Arrow function preserves this
const user = {
  name: 'John',
  printName: function() {
    setTimeout(() => {
      console.log(this.name);  // 'John' - correct
    }, 1000);
  }
};

// BAD - Mutating arguments
function addItem(arr) {
  arr.push('item');            // Modifies original
}

// GOOD - Immutable approach
function addItem(arr) {
  return [...arr, 'item'];     // Returns new array
}
```

## Conclusion

JavaScript fundamentals:
- Use `const` by default for immutability
- Understand scope and closures for proper encapsulation
- Use arrow functions for cleaner syntax
- Prefer strict equality (`===`)
- Leverage modern syntax (destructuring, spread operator)

