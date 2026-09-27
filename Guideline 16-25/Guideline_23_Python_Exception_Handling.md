# Guideline 23: Python Exception Handling

Things go wrong at runtime — a user types text where a number was expected, a file doesn't exist, a division by zero slips through. Python's answer to "something went wrong" is the **exception**: a special signal that interrupts normal execution and, if nothing catches it, crashes the program with an error message. This guideline covers how to anticipate problems yourself, how to catch exceptions Python raises automatically, and how to raise your own.

---

## 1. Custom Errors with If-Else (Things You Program Yourself)

Before reaching for Python's exception system at all, the simplest form of error handling is one you already know: check the condition yourself, with a plain `if`/`else`, before the risky operation ever runs.

```python
def divide(a, b):
    if b == 0:
        print("Error: cannot divide by zero.")
    else:
        print(a / b)

divide(10, 2)  # 5.0
divide(10, 0)  # Error: cannot divide by zero.
```

This is a completely legitimate way to handle an error — for anything you can **check in advance**, an `if`/`else` guard is often simpler and clearer than reaching for the machinery covered in the rest of this guideline. Its limitation shows up when the failure isn't something you can predict and check beforehand — a file that might or might not exist, a network request that might time out, user input in a format you didn't anticipate. That's the gap exceptions are built to fill.

---

## 2. The `try`-`except` Method

A `try` block wraps code that *might* fail; an `except` block catches the failure if it happens, instead of letting the program crash.

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("You can't divide by zero.")
```

Naming the specific exception type (`ZeroDivisionError` here) means this `except` only catches *that* kind of problem — anything else still crashes normally, which is usually what you want, since silently swallowing an unrelated bug is worse than letting it surface.

Multiple `except` blocks handle different failure types differently:

```python
try:
    value = int(input("Enter a number: "))
    print(10 / value)
except ZeroDivisionError:
    print("You can't divide by zero.")
except ValueError:
    print("That wasn't a valid number.")
```

A single `except` can also catch several exception types at once, by grouping them in a tuple:

```python
except (ZeroDivisionError, ValueError):
    print("Something about that input or calculation was invalid.")
```

And a bare `except Exception` acts as a catch-all fallback for anything not already handled by an earlier, more specific `except` — useful as a last resort, but best placed *after* the specific cases, since Python checks `except` blocks in order and stops at the first match.

---

## 3. Printing the Actual Error (`print(e)`)

Writing your own message like `"That wasn't a valid number"` is friendly, but it throws away the actual, specific detail Python generated about *why* something failed. Capturing the exception object with `as e` gives you access to that real message:

```python
try:
    value = int("hello")
except ValueError as e:
    print("Custom message: invalid input.")
    print("Actual error:", e)

# Custom message: invalid input.
# Actual error: invalid literal for int() with base 10: 'hello'
```

While you're developing and debugging, `print(e)` (or logging it) is often far more useful than a generic custom message — it tells you exactly what Python saw and why it objected, rather than just that *something* went wrong.

---

## 4. Difference Between `else` and `finally`

Two more optional blocks can attach to a `try`/`except`, and they run under very different conditions:

- **`else`** runs only if the `try` block completed with **no exception at all**.
- **`finally`** runs **no matter what** — exception or not, caught or uncaught — always the last thing to execute.

```python
try:
    number = int(input("Enter a number: "))
except ValueError:
    print("That wasn't a number.")
else:
    print(f"Great, you entered {number}.")
finally:
    print("Done processing this input.")
```

If the input is valid: `except` is skipped, `else` runs (printing the success message), and `finally` runs afterward. If the input is invalid: `except` runs (printing the error), `else` is skipped entirely, and `finally` **still runs**.

`finally` is the natural home for cleanup code that has to happen either way — closing a file you opened at the start of the `try` block, for instance, regardless of whether reading from it succeeded (a direct callback to the file-handling risks covered in Guideline 22, before `with open()` was introduced as the safer alternative).

```python
f = open("data.txt", "r")
try:
    contents = f.read()
    number = int(contents)
except ValueError:
    print("File didn't contain a valid number.")
finally:
    f.close()  # runs whether or not the ValueError happened
```

---

## 5. The `raise` Method

Sometimes *you* are the one who knows something has gone wrong — not Python. `raise` lets you trigger an exception manually, on your own terms, using any of Python's built-in exception types:

```python
def set_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative.")
    print(f"Age set to {age}")

set_age(25)   # Age set to 25
set_age(-5)   # raises ValueError: Age cannot be negative.
```

This is the formal, Python-native version of the "custom error" idea from Section 1 — instead of just printing a message and moving on, `raise` actually stops execution and produces a real exception, which calling code can then catch with `try`/`except` if it chooses to.

You're not limited to Python's built-in exception types either — defining your own, by inheriting from `Exception`, lets an error carry a name specific to your program's own logic:

```python
class NegativeAgeError(Exception):
    pass

def set_age(age):
    if age < 0:
        raise NegativeAgeError("Age cannot be negative.")
    print(f"Age set to {age}")

try:
    set_age(-5)
except NegativeAgeError as e:
    print("Custom exception caught:", e)
```

---

## 6. Re-Raising Exceptions

Sometimes you want to react to an exception — log it, print a diagnostic, clean something up — *without* actually suppressing it. Calling `raise` with no arguments inside an `except` block re-throws the exact exception currently being handled, letting it continue propagating upward after you've had your chance to respond to it.

```python
def process(value):
    try:
        return 10 / value
    except ZeroDivisionError as e:
        print("Logged: a division by zero was attempted.")
        raise  # re-raises the same ZeroDivisionError

process(0)
# Logged: a division by zero was attempted.
# (then the program still crashes with ZeroDivisionError, as if unhandled)
```

This matters whenever code further up the call stack needs to know a failure happened — re-raising means you can add logging or side effects at one layer without silently hiding the problem from whoever called your function.

---

## 7. The `assert` Method

`assert` is a sanity check: it states a condition you believe **must** be true at this point in the code, and if that condition turns out to be `False`, Python raises an `AssertionError` immediately.

```python
def get_average(numbers):
    assert len(numbers) > 0, "Cannot average an empty list"
    return sum(numbers) / len(numbers)

print(get_average([1, 2, 3]))  # 2.0
print(get_average([]))         # AssertionError: Cannot average an empty list
```

`assert` shows up constantly in two specific roles: **validating assumptions inside your own function** (as above — the function's logic simply doesn't make sense on an empty list), and **testing that a function produces the result you expect it to**, which is exactly the foundation that formal testing tools are built on:

```python
def add(a, b):
    return a + b

assert add(2, 3) == 5
assert add(-1, 1) == 0
print("All tests passed.")
```

If `add()` had a bug and actually returned the wrong value, the `assert` would raise an `AssertionError` right at the point of the incorrect result — flagging exactly which expectation failed.

---

## 8. Assertions vs. If-Else vs. Try-Except

All three of these can react to "something is wrong" — but they exist for different kinds of wrong, and mixing them up is a common source of confusing code.

| Tool | Best suited for | Key characteristic |
|---|---|---|
| **`if`-`else`** | Conditions you can check **in advance**, that are an expected, normal part of your program's logic (a user might enter 0; that's not a bug) | No exception is involved at all — just a branch in the code |
| **`try`-`except`** | Failures you **can't** conveniently check for in advance, or that come from outside your control (a missing file, bad network response, unpredictable input) | Lets the risky operation actually be attempted, and recovers gracefully if it fails |
| **`assert`** | Conditions that should be **impossible** if your own code is correct — a way of documenting and checking your own assumptions, mainly during development and testing | Meant to catch *your* bugs, not to handle expected user error — and can be stripped out entirely when Python is run with optimizations enabled, so it should never be relied on for something the program *must* check in production |

A practical way to keep them straight: if a condition is something you *expect* might legitimately happen (invalid user input, a file that may or may not exist), reach for `if`-`else` when you can check beforehand, and `try`-`except` when you can't. If a condition would only be `False` because of a mistake **in your own code** — a function called with arguments it was never supposed to receive, an internal invariant that must hold — that's what `assert` is for. Using `assert` to validate ordinary user input is a common misstep, precisely because assertions are meant to be removable safety nets for developers, not a permanent line of defense for the finished program.

---

Between checking for the errors you can anticipate, catching the ones you can't with `try`-`except`, raising your own when your code detects a problem, and using `assert` to keep your own assumptions honest, this covers the full toolkit Python gives you for turning "something went wrong" from a crash into a handled, predictable outcome.
