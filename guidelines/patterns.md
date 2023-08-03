# Coding Patterns

## <a name="avoid-else-guard-block">Avoiding the ELSE by improving IF statements and using Guard Blocks</a>

### Use early return

When you're in a function or a method, you can avoid nested if-else statements by using early return.

```python
def check_positive(number):
    if number <= 0:
        return False
    return True
```

### Use dictionary mappings

Sometimes, you can replace an if-else chain with a dictionary. This makes the code cleaner and more efficient.

```python
def get_direction(direction):
    return {
        'up': 1,
        'down': -1,
    }.get(direction, 0)
```

### Use guard blocks

Guard blocks are a way of handling exceptional cases or error conditions at the start of a function or a code block. They're often used with an early return.

```python
def divide_numbers(numerator, denominator):
    if denominator == 0:
        print("Cannot divide by zero!")
        return None
    return numerator / denominator
```

In this example, the function checks if the denominator is zero at the start. If it is, it immediately returns None, avoiding an exception later in the function.

[Guard clauses for better “if” statements — Python](https://medium.com/lemon-code/guard-clauses-3bc0cd96a2d3)

### Leverage the power of list comprehensions

List comprehensions are a unique feature in Python which can often replace if-else blocks. They are used for creating new lists from existing ones.

```python
numbers = [1, 2, 3, 4, 5]
even_numbers = [num for num in numbers if num % 2 == 0]
```



