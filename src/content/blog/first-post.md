---
title: 'Mastering Python Closures, Decorators, and Non-Local Variables'
description: 'Deep dive into closures, decorators, and non-local scopes in Python with practical examples, use cases, and best practices for writing clean and efficient code.'
pubDate: 'Jun 24 2025'
heroImage: '../../assets/blog_1.png'
---

Have you ever wondered how Python's `@property` decorator works under the hood? Or how libraries like Flask use decorators to register routes? The magic lies in understanding three interconnected concepts: **closures**, **decorators**, and **non-local variables**.

In this comprehensive guide, we'll build from the ground up, exploring these powerful concepts that enable elegant, functional programming patterns in Python.

> "Closures are a feature that allows a function to access variables from an outer scope even after the outer function has returned. They're like a backpack that the function carries around with all the variables it needs." — *Anonymous Developer*

## Prerequisites

Before we dive in, you should have:

- Basic Python knowledge (functions, variables, scope)
- Understanding of function arguments and return values
- Familiarity with Python's `def` keyword

## Table of Contents

1. [Understanding Scopes: The Foundation](#understanding-scopes-the-foundation)
2. [Non-Local Variables: Breaking Scope Boundaries](#non-local-variables-breaking-scope-boundaries)  
3. [Closures: Functions That Remember](#closures-functions-that-remember)
4. [Decorators: Elegant Function Enhancement](#decorators-elegant-function-enhancement)
5. [Common Pitfalls and Best Practices](#common-pitfalls-and-best-practices)
6. [Conclusion and What's Next](#conclusion-and-whats-next)

---

## Understanding Scopes: The Foundation

Before diving into closures and decorators, let's establish a solid understanding of Python's scoping rules using the **LEGB** principle:

- **L**ocal: Inside the current function
- **E**nclosing: Inside any enclosing functions  
- **G**lobal: At the top level of the module
- **B**uilt-in: Names pre-defined in Python

```python
# Global scope
global_var = "I'm global"

def outer_function():
    # Enclosing scope
    enclosing_var = "I'm enclosing"
    
    def inner_function():
        # Local scope
        local_var = "I'm local"
        
        print(f"Local variable: {local_var}")
        print(f"Enclosing variable: {enclosing_var}")  # accessing enclosing scope
        print(f"Global variable: {global_var}")        # accessing global scope
        print(f"Built-in variable: {len([1, 2, 3])}")  # accessing built-in scope
    
    return inner_function

# Usage
my_function = outer_function()
my_function()
```

**Output:**
```
Local variable: I'm local
Enclosing variable: I'm enclosing
Global variable: I'm global
Built-in variable: 3
```

Understanding these scoping rules is crucial because closures rely heavily on the enclosing scope to maintain state.

---

## Non-Local Variables: Breaking Scope Boundaries

The `nonlocal` keyword allows inner functions to modify variables in their enclosing scope, creating powerful patterns for state management.

### Basic Non-Local Usage

```python
def make_counter():
    count = 0  # Enclosing scope variable
    
    def increment():
        nonlocal count  # Declare we want to modify enclosing scope variable
        count += 1
        return count
    
    return increment

# Usage
counter = make_counter()
print(counter())  # 1
print(counter())  # 2
print(counter())  # 3

# Each counter maintains its own state
another_counter = make_counter()
print(another_counter())  # 1
print(counter())          # 4
```

### Multiple Non-Local Variables

```python
def make_bank_account(initial_balance=0):
    balance = initial_balance
    transaction_history = []
    
    def deposit(amount):
        nonlocal balance
        balance += amount
        transaction_history.append(f"Deposited ${amount}")
        return balance
    
    def withdraw(amount):
        nonlocal balance
        if balance >= amount:
            balance -= amount
            transaction_history.append(f"Withdrew ${amount}")
            return balance
        else:
            raise ValueError("Insufficient funds")
    
    def get_balance():
        return balance
    
    def get_transaction_history():
        return transaction_history.copy()
    
    return {
        'deposit': deposit,
        'withdraw': withdraw,
        'get_balance': get_balance,
        'get_transaction_history': get_transaction_history
    }

# Usage
account = make_bank_account(100)
print(account['get_balance']())     # 100
print(account['deposit'](50))       # 150
print(account['withdraw'](30))      # 120
print(account['get_balance']())     # 120
print(account['get_transaction_history']())  # ['Deposited $50', 'Withdrew $30']
```

The `nonlocal` keyword is what makes these patterns possible, allowing inner functions to maintain and modify state across multiple calls.

---

## Closures: Functions That Remember

A **closure** is created when a nested function references variables from its enclosing scope. The function "closes over" these variables, preserving their values even after the outer function has finished executing.

### Anatomy of a Closure

```python
def create_multiplier(factor):
    """Outer function that creates the closure"""
    
    def multiply(x):
        """Inner function - the actual closure"""
        return x * factor  # References 'factor' from enclosing scope
    
    return multiply  # Return the inner function, not call it

# Create closures with different factors
double = create_multiplier(2)
triple = create_multiplier(3)

# The closures remember their factor
print(double(5))   # 10 (2 * 5)
print(triple(5))   # 15 (3 * 5)

# Verify they're different functions with different enclosed variables
print(double.__closure__[0].cell_contents)  # 2
print(triple.__closure__[0].cell_contents)  # 3
```

### State Preservation with Closures

```python
def make_averager():
    """Creates a closure that calculates running average"""
    numbers = []
    
    def add_number(number):
        """Inner function that adds a number and calculates the average"""
        numbers.append(number)
        return sum(numbers) / len(numbers)
    
    return add_number

# Usage
averager = make_averager()
print(averager(10))  # 10.0
print(averager(20))  # 15.0 (average of 10, 20)
print(averager(30))  # 20.0 (average of 10, 20, 30)
```

Closures are incredibly powerful because they maintain state without requiring classes or global variables.

---

## Decorators: Elegant Function Enhancement

Decorators are syntactic sugar for applying closures to modify or enhance functions. They follow the pattern: `decorator(function) -> enhanced_function`.

### Basic Decorator Structure

```python
def my_decorator(func):
    """A basic decorator template"""
    def wrapper(*args, **kwargs):
        # Code to run before the original function
        print(f"Calling function: {func.__name__}")
        
        # Call the original function
        result = func(*args, **kwargs)
        
        # Code to run after the original function
        print(f"Function {func.__name__} completed")
        
        return result
    
    return wrapper

# Usage with @ syntax
@my_decorator
def greet(name):
    return f"Hello, {name}!"

# Equivalent to: greet = my_decorator(greet)

print(greet("Alice"))
```

**Output:**
```
Calling function: greet
Function greet completed
Hello, Alice!
```

### Practical Decorator Example

```python
import time
import functools

def timer(func):
    """Decorator to measure function execution time"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.perf_counter()
        result = func(*args, **kwargs)
        end_time = time.perf_counter()
        print(f"{func.__name__} took {end_time - start_time:.4f} seconds")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
    return "Done!"

result = slow_function()  # Prints timing information
```

Decorators make it incredibly easy to add functionality to existing functions without modifying their core logic.

---

## Common Pitfalls and Best Practices

### Late Binding Gotcha

⚠️ **Common Pitfall**: Closures exhibit **late binding** behavior with loop variables:

```python
# WRONG: All functions will print 3
functions = []
for i in range(3):
    functions.append(lambda: print(i))

for func in functions:
    func()  # Prints: 3, 3, 3

# CORRECT: Capture the loop variable
functions = []
for i in range(3):
    functions.append(lambda x=i: print(x))  # Default argument captures current value

for func in functions:
    func()  # Prints: 0, 1, 2
```

### Best Practices

1. **Always use `functools.wraps`** in decorators to preserve function metadata
2. **Be aware of late binding** when creating closures in loops
3. **Use closures for encapsulation** when you need private state
4. **Prefer closures over classes** for simple stateful functions

---

## Conclusion and What's Next

Closures, decorators, and non-local variables are powerful Python features that enable elegant, functional programming patterns. In this post, we've covered:

- **Scopes**: Understanding the LEGB rule as our foundation
- **Non-local variables**: Enabling inner functions to modify enclosing scope
- **Closures**: Functions that "remember" their enclosing environment  
- **Decorators**: Syntactic sugar for applying function enhancements

### Key Takeaways

1. **Closures preserve state** without requiring classes or global variables
2. **Decorators enhance reusability** by separating concerns
3. **Non-local enables sophisticated state management** patterns
4. **Python's scoping rules make closures predictable** and easy to reason about
5. **Watch out for late binding** when creating closures in loops
---
