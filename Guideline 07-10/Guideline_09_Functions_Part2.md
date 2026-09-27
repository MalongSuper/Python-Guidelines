# Guideline 9: Python Functions
## Part 2

Part 1 established what a function is, the difference between returning a value and printing one, and how to define a function using `def`. Part 2 puts functions to work alongside everything else in this book so far — variables, conditionals, and loops — and then covers three ideas that change how you think about functions more fundamentally: returning several values at once, defining one function inside another, and what actually happens when two functions share the same name.

---

## 1. Variables with Functions

A function's return value is not a special, limited kind of output — the moment `return` hands a value back, that value behaves exactly like any other value in Python. It can be stored in a variable, and once it's stored, that variable can be used anywhere a plain number, string, or Boolean could be used: in further arithmetic, inside another condition, or passed straight into another function call.

```python
def add(a, b):
    return a + b

total = add(3, 4)
print(total)          # 7

doubled = total * 2
print(doubled)         # 14
```

`total` isn't a reference back to the function, and it isn't somehow tied to `add()` after this point — it is simply the plain integer `7`, sitting in a variable, indistinguishable from if you had written `total = 7` directly. This is worth being clear on precisely because it's easy to assume a variable holding a function's result is somehow special; it isn't. The function's job ends the instant it returns.

This also means a returned value can be fed directly into another function call, without an intermediate variable at all, if you don't need to keep it around separately:

```python
def square(n):
    return n * n

result = add(square(3), square(4))
print(result)   # 25
```

Here, `square(3)` and `square(4)` are each evaluated first — producing `9` and `16` — and *those* results are what actually get passed into `add()`. Python always works from the inside out: the innermost function calls resolve to plain values before the outer call ever runs.

---

## 2. Using a Function Inside a Conditional or a Loop

Because a function call is just an expression that produces a value, it can be placed directly inside an `if` condition or a loop, exactly the same way a comparison operator would be.

```python
def is_prime(number):
    if number < 2:
        return False
    for divisor in range(2, number):
        if number % divisor == 0:
            return False
    return True

n = int(input("Enter a number: "))

if is_prime(n):
    print(f"{n} is prime.")
else:
    print(f"{n} is not prime.")
```

`is_prime(n)` is called once, its `True` or `False` result is used directly as the `if` condition — there's no need for an extra variable in between; the function call *is* the condition.

Calling a function repeatedly inside a loop is where the reusability from Part 1 really pays off — the exact same logic runs fresh on every iteration, against a different value each time, without ever being rewritten:

```python
prime_count = 0

for number in range(1, 51):
    if is_prime(number):
        prime_count += 1

print(f"There are {prime_count} prime numbers between 1 and 50.")
```

This combines three ideas from earlier guidelines directly: the `for` loop provides the repetition (Guideline 8), `is_prime()` provides the decision for each individual number (Guideline 9, Part 1), and `prime_count` is an accumulator (also Guideline 8) that builds up a running total across every iteration. None of these three pieces changed to work together — they simply compose, which is exactly what well-designed functions are supposed to let you do.

---

## 3. Functions That Return Multiple Values (Or as a Collection)

A `return` statement isn't limited to a single value — separating several values with commas returns all of them together, bundled into one structure.

```python
def divide_with_remainder(a, b):
    quotient = a // b
    remainder = a % b
    return quotient, remainder

q, r = divide_with_remainder(17, 5)
print(q)   # 3
print(r)   # 2
```

`q, r = divide_with_remainder(17, 5)` **unpacks** the two returned values directly into two separate variables in one step — this is the exact same unpacking behavior already used with `enumerate()` and `zip()` back in Guideline 8, just applied to a function's return value instead of a loop.

If you capture the return value with a single variable instead of unpacking it, that variable ends up holding the *entire bundle* at once, printed together in parentheses:

```python
result = divide_with_remainder(17, 5)
print(result)   # (3, 2)
```

That parenthesized pairing is called a **tuple** — a small, ordered collection of values grouped together as one — and it will get a full guideline of its own later in this book, alongside lists, dictionaries, and sets. For now, the important part isn't the name of the structure; it's that a function can hand back more than one piece of information from a single call, and that unpacking it directly into named variables, as `q, r = ...` did above, is usually the clearest way to work with it.

A second example, returning two related statistics from the same pair of numbers:

```python
def sum_and_average(a, b):
    total = a + b
    average = total / 2
    return total, average

total, average = sum_and_average(10, 14)
print(f"Total: {total}, Average: {average}")
```

Both `total` and `average` come from a single function call — there's no need for two separate functions, or two separate calls, when the two results are naturally computed from the same underlying work.

---

## 4. Nested Functions

Just as an `if` statement can contain another `if`, and a loop can contain another loop, a function can contain another function, defined entirely within its body. This is called a **nested function**, and it only exists while the outer function is actually running — it cannot be called from anywhere outside.

```python
def describe_number(n):
    def is_even_helper(x):
        return x % 2 == 0

    if is_even_helper(n):
        return f"{n} is even"
    else:
        return f"{n} is odd"

print(describe_number(7))    # 7 is odd
print(describe_number(12))   # 12 is even
```

`is_even_helper()` is defined fresh, inside `describe_number()`, every time `describe_number()` is called. It's available for use anywhere inside `describe_number()`'s body — but try to call it from outside:

```python
is_even_helper(4)
```

```
NameError: name 'is_even_helper' is not defined
```

This isn't a bug — it's the entire point of nesting. `is_even_helper()` was never meant to be a general-purpose tool available to the whole program; it's a small, private piece of logic that only `describe_number()` needs, and nesting it keeps that fact explicit. As programs grow larger, this becomes genuinely useful: it keeps a program's list of "names anyone could call from anywhere" limited to the functions that are actually meant to be used broadly, while small helper steps stay tucked inside the one function that relies on them.

---

## 5. Multiple Functions with the Same Name (Overriding)

What happens if you define two functions with the exact same name? Unlike some other programming languages, Python does not keep both versions around and pick between them based on how they're called — **the second definition completely replaces the first.** This is called **overriding**, and it happens silently, with no warning or error.

```python
def greet():
    print("Hello!")

def greet():
    print("Hi there!")

greet()
```

```
Hi there!
```

The first `greet()` is not "still there but shadowed" — it is gone entirely, as though it had never been written. Calling `greet()` can only ever reach the second definition, because that's the only one Python still has a record of.

The reason this happens becomes clear once you recognize what a function's name actually *is*: it's a variable, exactly like any other, whose value happens to be a function object rather than a number or a string. `def greet():` is really doing the same fundamental thing as `x = 5` — it's binding a name to a value. And just as writing `x = 5` followed later by `x = 10` leaves `x` holding only `10`, writing `def greet():` twice leaves `greet` bound to only the second function. There was never a rule against reassigning a name — functions simply make it easy to forget that a `def` is doing exactly that.

---

## 6. Overriding Built-in Functions

If a function's name is just a variable, and a variable can be reassigned by writing a new `def` with the same name — nothing stops that same rule from applying to a *built-in* function's name too. `print`, `len`, `sum`, `list`, `type`, and every other built-in are themselves just names that Python has already bound to function objects before your program even starts running. Defining your own function (or even a plain variable) with one of those exact names overrides the built-in for the rest of the program, exactly the way `greet()` was overridden above.

```python
def print(message):
    pass  # does nothing at all

print("This will never appear.")
```

Nothing is displayed here — the real `print()` function, the one that actually writes to the screen, has been completely replaced by this new, empty version. Every future call to `print()` in this program will use the new definition, not the original, until the program ends or `print` is reassigned back to what it was.

This is rarely something you'd want to do on purpose, and it is a genuinely common accidental bug — usually not through redefining a function, but through an ordinary variable assignment that happens to reuse a built-in's name:

```python
sum = 100          # sum is now just an integer — the built-in function is gone

total = sum([1, 2, 3])
```

```
TypeError: 'int' object is not callable
```

`sum` used to refer to the built-in function that adds up a collection of numbers. After `sum = 100`, it refers to the integer `100` instead — and integers can't be "called" with parentheses the way functions can, which is exactly what the error above is reporting. The built-in hasn't crashed or been deleted from Python itself; it's simply no longer what the name `sum` points to *in this program*, because that name was reassigned.

The practical takeaway is simple: avoid naming your own variables or functions `print`, `input`, `len`, `sum`, `list`, `type`, `str`, `id`, or any other name you're already relying on as a built-in — not because Python forbids it, but because it doesn't warn you when you do it, and the resulting errors (like the `TypeError` above) tend to show up somewhere else entirely, at a point where the actual mistake is no longer visible on screen. If this does happen inside an interactive environment like a Jupyter notebook or Google Colab, simply reassigning the name back won't always feel clean — restarting the notebook's runtime is often the most reliable way to get the original built-in back exactly as it was.
