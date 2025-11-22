---
layout: post
title: "Python Fundamentals: Variables, Types, and Data Structures"
author: "Liberom"
date: 2025-01-11
category: "Python"
toc: true
excerpt: "Master Python basics: variables, types, collections, and functional programming patterns."
---

# Python Fundamentals: Variables, Types, and Data Structures

Python's simplicity and readability make it ideal for beginners and professionals alike. Understanding fundamentals is key.

## Variables and Types

### Dynamic Typing

```python
# Python uses dynamic typing
name = 'John'               # string
age = 30                    # int
height = 5.9                # float
is_active = True            # bool

# Type checking
type(name)                  # <class 'str'>
type(age)                   # <class 'int'>
isinstance(age, int)        # True

# Type conversion
int('42')                   # 42
str(42)                     # '42'
float('3.14')               # 3.14
bool(0)                     # False
bool(1)                     # True
```

### Naming Conventions

```python
# snake_case for variables and functions
user_name = 'John'
calculate_total = lambda x: x * 2

# UPPER_CASE for constants
MAX_RETRIES = 3
PI = 3.14159

# PascalCase for classes
class UserAccount:
    pass

class DataProcessor:
    pass
```

## Data Structures

### Lists (Mutable, Ordered)

```python
# Creating lists
numbers = [1, 2, 3, 4, 5]
mixed = [1, 'hello', 3.14, True, None]
empty = []

# Accessing elements
numbers[0]                  # 1 (0-indexed)
numbers[-1]                 # 5 (last element)
numbers[1:4]                # [2, 3, 4] (slice)
numbers[:3]                 # [1, 2, 3]
numbers[::2]                # [1, 3, 5] (every 2nd)

# Modifying lists
numbers.append(6)           # Add to end
numbers.extend([7, 8])      # Add multiple
numbers.insert(0, 0)        # Insert at position
numbers.remove(5)           # Remove value
popped = numbers.pop()      # Remove and return last
numbers.pop(0)              # Remove at index

# List operations
len(numbers)                # 8
numbers.count(2)            # 1
numbers.index(3)            # 2
numbers.sort()              # Sort in place
sorted(numbers, reverse=True)  # Return sorted copy

# List comprehension
squares = [x**2 for x in range(5)]  # [0, 1, 4, 9, 16]
evens = [x for x in numbers if x % 2 == 0]
```

### Tuples (Immutable, Ordered)

```python
# Creating tuples
coordinates = (10, 20)
single = (42,)              # Comma needed for single element
empty = ()

# Accessing elements (like lists)
coordinates[0]              # 10
coordinates[-1]             # 20

# Unpacking
x, y = coordinates
first, *rest = [1, 2, 3, 4] # rest = [2, 3, 4]

# Tuples are immutable
# coordinates[0] = 5        # Error

# Why use tuples?
# - Dictionary keys (lists can't be)
# - Return multiple values
# - Ensure data isn't modified
def get_user():
    return ('John', 30, 'john@example.com')  # Better than dict for fixed structure

user_name, user_age, user_email = get_user()
```

### Dictionaries (Mutable, Key-Value)

```python
# Creating dictionaries
user = {
    'name': 'John',
    'age': 30,
    'email': 'john@example.com'
}

# Accessing values
user['name']                # 'John'
user.get('name')            # 'John' (safe)
user.get('phone', 'N/A')    # 'N/A' (default if missing)

# Adding/modifying
user['phone'] = '555-1234'
user.update({'city': 'NYC', 'state': 'NY'})

# Removing
del user['phone']
removed = user.pop('state')

# Iterating
for key in user:
    print(key, user[key])

for key, value in user.items():
    print(f'{key}: {value}')

# Dictionary comprehension
squares = {x: x**2 for x in range(5)}  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

### Sets (Unordered, Unique)

```python
# Creating sets
numbers = {1, 2, 3, 4, 5}
unique = set([1, 1, 2, 2, 3])  # {1, 2, 3}

# Set operations
numbers.add(6)
numbers.remove(5)           # Error if not present
numbers.discard(5)          # No error

# Set algebra
set1 = {1, 2, 3}
set2 = {2, 3, 4}

set1 & set2                 # {2, 3} - intersection
set1 | set2                 # {1, 2, 3, 4} - union
set1 - set2                 # {1} - difference
set1 ^ set2                 # {1, 4} - symmetric difference

# Membership
2 in set1                   # True
5 not in set1               # True

# Removing duplicates
duplicates = [1, 1, 2, 2, 3, 3]
unique = list(set(duplicates))  # [1, 2, 3]
```

## Strings

### String Operations

```python
# String creation
single = 'single quotes'
double = "double quotes"
multi = '''multiline
string'''

# String formatting
name = 'John'
age = 30

# Old style (avoid)
'Hello %s, age %d' % (name, age)

# str.format()
'Hello {}, age {}'.format(name, age)
'Hello {name}, age {age}'.format(name=name, age=age)

# f-strings (modern, preferred)
f'Hello {name}, age {age}'
f'Hello {name}, age {age + 5}'
f'Value: {3.14159:.2f}'     # Formatting

# String methods
text = 'Hello World'
text.lower()                # 'hello world'
text.upper()                # 'HELLO WORLD'
text.replace('World', 'Python')  # 'Hello Python'
text.split()                # ['Hello', 'World']
text.strip()                # Remove whitespace

# Checking content
text.startswith('Hello')    # True
text.endswith('World')      # True
text.isdigit()              # False
text.isalpha()              # False (has space)

# Joining
words = ['Hello', 'World', 'Python']
' '.join(words)             # 'Hello World Python'
```

## Functions

### Function Definition

```python
# Basic function
def greet(name):
    return f'Hello, {name}!'

greet('John')               # 'Hello, John!'

# Default parameters
def greet(name='Guest'):
    return f'Hello, {name}!'

greet()                     # 'Hello, Guest!'

# Multiple returns
def get_user():
    return ('John', 30, 'john@example.com')

name, age, email = get_user()

# *args - variable positional arguments
def sum_all(*numbers):
    return sum(numbers)

sum_all(1, 2, 3, 4, 5)      # 15

# **kwargs - variable keyword arguments
def print_user(**info):
    for key, value in info.items():
        print(f'{key}: {value}')

print_user(name='John', age=30, city='NYC')

# Combining all
def flex(*args, **kwargs):
    print(f'Args: {args}')
    print(f'Kwargs: {kwargs}')

flex(1, 2, 3, name='John')
```

### Lambda Functions

```python
# Anonymous function
square = lambda x: x ** 2
square(5)                   # 25

# With map
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))  # [1, 4, 9, 16, 25]

# With filter
evens = list(filter(lambda x: x % 2 == 0, numbers))  # [2, 4]

# With sorted
users = [
    {'name': 'John', 'age': 30},
    {'name': 'Alice', 'age': 25}
]
sorted(users, key=lambda u: u['age'])
```

## Common Patterns

### Iteration

```python
# For loop
for i in range(5):
    print(i)                # 0, 1, 2, 3, 4

for item in [1, 2, 3]:
    print(item)

# Enumerate (index + value)
for index, item in enumerate(['a', 'b', 'c']):
    print(f'{index}: {item}')  # 0: a, 1: b, 2: c

# While loop
count = 0
while count < 5:
    print(count)
    count += 1

# List comprehension
squares = [x**2 for x in range(5)]

# Dict comprehension
user_ages = {name: age for name, age in [('John', 30), ('Alice', 25)]}
```

### Exception Handling

```python
# Try-except
try:
    result = 10 / 0
except ZeroDivisionError:
    print('Cannot divide by zero')
except Exception as e:
    print(f'Error: {e}')
finally:
    print('Cleanup code')

# Raising exceptions
def validate_age(age):
    if age < 0:
        raise ValueError('Age cannot be negative')
    return age
```

## Common Mistakes

```python
# BAD - Mutable default argument
def add_item(item, items=[]):
    items.append(item)
    return items

add_item(1)                 # [1]
add_item(2)                 # [1, 2] - unexpected!

# GOOD
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

# BAD - Modifying list while iterating
numbers = [1, 2, 3, 4, 5]
for num in numbers:
    if num % 2 == 0:
        numbers.remove(num)  # Skips elements!

# GOOD
numbers = [x for x in numbers if x % 2 != 0]

# BAD - Comparing with None using ==
if value == None:
    pass

# GOOD
if value is None:
    pass
```

## Conclusion

Python fundamentals:
- Use appropriate data structures for your task
- Write functions with clear purposes
- Use f-strings for readable formatting
- Master list/dict comprehensions for concise code
- Handle exceptions explicitly
- Follow PEP 8 naming conventions

