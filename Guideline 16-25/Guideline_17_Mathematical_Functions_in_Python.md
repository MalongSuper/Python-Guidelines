# Guideline 17: Mathematical Functions in Python

Python ships with a handful of mathematical tools built directly into the language — no setup required — but the moment your work leans toward anything resembling data science, you'll reach for more than that. This guideline is a sneak peek at one of Python's biggest strengths: **external libraries**. Rather than reinventing a square root function yourself, you `import` a module that someone has already written, tested, and optimized, and you simply use it.

This guidebook won't attempt to cover every library Python offers — that would be its own book. Instead, we'll focus on the one that shows up constantly in the study of data: the **math module**. But before we get there, let's finish the built-in functions that handle basic math without needing an import at all.

---

## 1. Built-in Mathematical Functions (Outside the `math` Module)

These functions are always available — you never need to `import` anything to use them. You've likely already met a few of these back in Guideline 6, but they're worth gathering here specifically for their mathematical role.

### `abs()`

Returns the absolute value — the distance from zero, always non-negative.

```python
print(abs(-7))    # 7
print(abs(7))     # 7
print(abs(-3.5))  # 3.5
```

### `divmod()`

Returns a tuple of `(quotient, remainder)` in a single call — equivalent to using `//` and `%` separately, but in one step.

```python
print(divmod(17, 5))  # (3, 2)  -> 17 // 5 = 3, 17 % 5 = 2
```

### `max()` / `min()`

Return the largest or smallest value from a set of arguments, or from a single iterable.

```python
print(max(4, 9, 2))       # 9
print(min([4, 9, 2]))     # 2
print(max("apple", "banana", key=len))  # banana (longest string)
```

### `pow()`

Raises a number to a power — equivalent to the `**` operator, but as a function call. It also supports a third argument for modular exponentiation (raise to a power, *then* take the modulus, computed efficiently in one step).

```python
print(pow(2, 5))     # 32   (same as 2 ** 5)
print(pow(2, 5, 7))  # 4    (2 ** 5) % 7, computed efficiently
```

### `round()`

Rounds a number to the nearest whole number, or to a given number of decimal places. As covered back in Guideline 4, remember that Python rounds ties to the nearest *even* digit (banker's rounding).

```python
print(round(3.6))       # 4
print(round(3.14159, 2)) # 3.14
print(round(2.5))       # 2  (rounds to even, not always "up")
```

### `sum()`

Adds up all the values in an iterable, with an optional starting value.

```python
print(sum([1, 2, 3, 4]))       # 10
print(sum([1, 2, 3, 4], 100))  # 110  (100 + the total)
```

None of these require an import — they're part of core Python, always ready to use.

---

## 2. The `map()` Function

`map()` applies a function to *every element* of an iterable, without you having to write the loop yourself. Its signature is:

```python
map(function, iterable)
```

This is especially useful for math involving lists — instead of looping through a list to transform every value, you hand `map()` the function you want applied, and it does the work element by element. The catch: `map()` doesn't return a list directly — it returns a *map object*, a lazy iterator that produces values on demand. Wrap it in `list()` to see the results.

```python
numbers = [1, 4, 9, 16]

# Apply abs() to every element
result = map(abs, [-3, -7, 2, -1])
print(list(result))  # [3, 7, 2, 1]

# Using a lambda for a custom calculation
squared_roots = map(lambda x: x ** 0.5, numbers)
print(list(squared_roots))  # [1.0, 2.0, 3.0, 4.0]
```

`map()` can also take *multiple* iterables at once, applying the function to corresponding elements from each:

```python
a = [1, 2, 3]
b = [10, 20, 30]

totals = map(lambda x, y: x + y, a, b)
print(list(totals))  # [11, 22, 33]
```

You've already seen this exact *result* achievable with a list comprehension (`[x + y for x, y in zip(a, b)]`) — and that's a fair comparison to draw. `map()` and list comprehensions frequently solve the same problem; `map()` tends to read more naturally when you're applying an existing function (like `abs` or `str`) across a list, while comprehensions tend to read more naturally when the transformation is a custom expression written inline. Neither is strictly better — it's a matter of what's clearest for the situation.

---

## 3. The `math` Module

For anything beyond the basics — trigonometry, logarithms, rounding toward specific directions, combinatorics — Python hands the job to the **math module**, part of the standard library. Since it isn't loaded automatically, you bring it in with an `import` statement first.

### Importing the module

```python
import math
print(math.sqrt(16))  # 4.0
```

You can also import specific functions directly, so you don't need the `math.` prefix each time:

```python
from math import sqrt, pi
print(sqrt(16))  # 4.0
print(pi)        # 3.141592653589793
```

Or give the module a shorter alias — a common convention when a module name is long or used constantly:

```python
import math as m
print(m.sqrt(16))  # 4.0
```

### Built-ins vs. their `math` counterparts

A few `math` functions look like duplicates of built-ins you already know — but they behave a little differently. `math.fabs()`, for instance, does the same job as `abs()`, but with one guaranteed difference: `math.fabs()` **always** returns a `float`, even if you pass in an integer, whereas `abs()` preserves the type it was given.

```python
print(abs(-5))        # 5      (int stays int)
print(math.fabs(-5))  # 5.0    (always float)
```

Small distinctions like this matter more than they seem — if your code depends on a value staying an integer, reaching for the wrong one can quietly introduce a type you didn't expect.

### Reference Table: Common `math` Functions

| Function | What It Does | Example (Code) |
|---|---|---|
| `math.fabs(x)` | Absolute value, always returned as a float | `math.fabs(-5)` → `5.0` |
| `math.ceil(x)` | Rounds **up** to the nearest whole number | `math.ceil(4.1)` → `5` |
| `math.floor(x)` | Rounds **down** to the nearest whole number | `math.floor(4.9)` → `4` |
| `math.comb(n, k)` | Number of ways to choose `k` items from `n`, order doesn't matter | `math.comb(5, 2)` → `10` |
| `math.factorial(n)` | The factorial of `n` (`n!`) | `math.factorial(5)` → `120` |
| `math.gcd(a, b)` | Greatest common divisor of two (or more) integers | `math.gcd(12, 18)` → `6` |
| `math.isclose(a, b)` | Checks whether two floats are "close enough" to be considered equal | `math.isclose(0.1 + 0.2, 0.3)` → `True` |
| `math.prod(iterable)` | The product of all values in an iterable (multiplication's version of `sum()`) | `math.prod([1, 2, 3, 4])` → `24` |
| `math.sqrt(x)` | Square root of `x` | `math.sqrt(49)` → `7.0` |
| `math.cbrt(x)` | Cube root of `x` (available from Python 3.11 onward) | `math.cbrt(27)` → `3.0` |
| `math.exp(x)` | Euler's number `e` raised to the power `x` | `math.exp(1)` → `2.718281828459045` |
| `math.log(x)` | Natural logarithm (base `e`) of `x` | `math.log(math.e)` → `1.0` |
| `math.log10(x)` | Logarithm of `x`, base 10 | `math.log10(1000)` → `3.0` |
| `math.log2(x)` | Logarithm of `x`, base 2 | `math.log2(8)` → `3.0` |
| `math.sin(x)` | Sine of `x`, where `x` is in **radians** | `math.sin(math.pi / 2)` → `1.0` |
| `math.cos(x)` | Cosine of `x`, in radians | `math.cos(0)` → `1.0` |
| `math.tan(x)` | Tangent of `x`, in radians | `math.tan(0)` → `0.0` |
| `math.atan2(y, x)` | Angle (in radians) between the positive x-axis and the point `(x, y)` | `math.atan2(1, 1)` → `0.785...` |
| `math.degrees(x)` | Converts an angle from radians to degrees | `math.degrees(math.pi)` → `180.0` |
| `math.radians(x)` | Converts an angle from degrees to radians | `math.radians(180)` → `3.14159...` |
| `math.hypot(x, y)` | Length of the hypotenuse of a right triangle with legs `x` and `y` | `math.hypot(3, 4)` → `5.0` |
| `math.pi` | The constant π (not a function — a value) | `math.pi` → `3.141592653589793` |
| `math.e` | Euler's number, the base of natural logarithms (a value) | `math.e` → `2.718281828459045` |
| `math.tau` | The constant τ, equal to 2π (a value) | `math.tau` → `6.283185307179586` |
| `math.inf` | Represents positive infinity (a value) | `math.inf > 10 ** 100` → `True` |
| `math.nan` | "Not a Number" — represents an undefined or unrepresentable numeric result (a value) | `math.nan == math.nan` → `False` |

A few things worth pointing out about that last group: `pi`, `e`, `tau`, `inf`, and `nan` are not functions you *call* — there are no parentheses after them. They're constant values that live inside the module, accessed exactly like any other attribute. And `nan` has a famously strange property: it is never equal to itself. This isn't a bug — it reflects the mathematical idea that an undefined value can't be meaningfully compared, even to another instance of itself. If you ever need to check whether a value *is* `nan`, use `math.isnan(x)` rather than `x == math.nan`.

Also worth remembering: **every trigonometric function in the `math` module expects radians, not degrees.** If your angle is in degrees, run it through `math.radians()` first — this is precisely why `degrees()` and `radians()` sit right alongside `sin`, `cos`, and `tan` in the module rather than being an afterthought.

---

Between the handful of always-available built-ins, `map()` for applying a function across a list without writing the loop by hand, and the `math` module's much larger toolbox, you now have what you need for the overwhelming majority of everyday mathematical work in Python. This is also your first real taste of what "importing a library" looks like in practice — a pattern you'll see again and again as this book moves toward data science proper.
