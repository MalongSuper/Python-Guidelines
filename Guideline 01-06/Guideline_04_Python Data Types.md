# Python Guidebook
## Guideline 4: Python Data Types

---

Every value in Python belongs to a **data type**, a category that determines what kind of information the value holds and what operations can legally be performed on it. This is not a new idea introduced for the first time here; Guideline 2 already ran into it directly, when `input()` handed back a string that first had to be converted with `int()` or `float()` before it could be used in arithmetic. This guideline gives that behavior a proper foundation by covering Python's basic, built-in data types on their own terms, before more complex, multi-value data types such as lists, tuples, dictionaries, and sets get a dedicated guideline of their own later in this book.

### 4.1 str — Strings (Text)

A **string** is a sequence of characters, used to represent text. Strings are created by surrounding text with either single quotes or double quotes, both of which work identically in Python.

```python
name = "Iris"
greeting = 'Hello there'
```

Strings can be joined together (concatenated) with `+`, and repeated with `*`:

```python
first = "Py"
second = "thon"
print(first + second)   # Python
print("ab" * 3)          # ababab
```

Anything wrapped in quotes is treated as text, even if it looks numeric. `"25"` is a string containing the characters `2` and `5`, not the number twenty-five, which is exactly the distinction that made type casting necessary back in Guideline 2.

### 4.2 int — Integers (Whole Numbers)

An **integer** is a whole number, positive, negative, or zero, with no decimal component.

```python
age = 25
temperature = -10
count = 0
```

Unlike some other languages, Python integers are not limited to a fixed number of bits and can grow arbitrarily large without special handling, so calculations like `2 ** 100` produce an exact, correct result rather than overflowing or losing precision.

### 4.3 float — Floating-Point Numbers (Decimal Numbers)

A **float** is a number that includes a decimal point, used to represent values that are not whole.

```python
price = 19.99
gpa = 3.75
pi_estimate = 3.14
```

Even a whole-looking value becomes a float the moment a decimal point is present: `5.0` is a float, while `5` is an int, and Python treats them as genuinely different types even though they represent the same mathematical quantity. This distinction matters because floats are stored in a computer using a binary approximation that cannot represent every decimal value with perfect precision, which is why a calculation like `0.1 + 0.2` in Python does not print exactly `0.3`, but something like `0.30000000000000004`. This is not a bug specific to Python; it is a consequence of how nearly all programming languages represent decimal numbers in binary, and it is worth knowing about early so that a tiny discrepancy like this does not look like broken code the first time you encounter it.

### 4.4 bool — Boolean Values (True or False)

A **Boolean**, introduced in Guideline 3, has exactly two possible values, `True` and `False`, always capitalized, and is most often the direct result of a comparison.

```python
is_adult = 20 >= 18
print(is_adult)   # True
```

### 4.5 None — The Absence of a Value

**`None`** is a special, singular value in Python that represents the deliberate absence of a value, rather than zero, an empty string, or `False`. It has its own type, `NoneType`, and it is commonly used to signal that a variable exists but has not yet been given a meaningful value, or that a function was called but had nothing useful to return.

```python
result = None
print(result)   # None

def find_user(username):
    if username == "admin":
        return "Found the admin account"
    return None   # explicitly signals "nothing was found"
```

`None` is not the same as `0`, `False`, or `""`, even though all of them can behave similarly in certain checks, a point that becomes important in the next section.

### 4.6 Data Type Methods

**`type()`** tells you exactly what data type a given value currently is, which is especially useful when a value's type is not obvious just from looking at it, or when tracking down a bug caused by a value being the wrong type.

```python
print(type(25))       # <class 'int'>
print(type(19.99))    # <class 'float'>
print(type("Iris"))   # <class 'str'>
print(type(True))     # <class 'bool'>
print(type(None))     # <class 'NoneType'>
```

**`bool()`** converts any value into its Boolean equivalent, `True` or `False`, based on a rule Python applies consistently across every type: certain values are considered inherently "empty" or "nothing," and convert to `False`; everything else converts to `True`. The specific values that convert to `False` are `0`, `0.0`, `""` (an empty string), `None`, and empty collections such as `[]` or `{}` (covered in a later guideline). This is often described as a value's **truthiness**.

```python
print(bool(0))        # False
print(bool(1))        # True
print(bool(""))        # False
print(bool("hello"))   # True
print(bool(None))      # False
```

This is not just a conversion trick; it is the same rule Python quietly applies whenever a value is placed directly into an `if` statement without an explicit comparison, so understanding truthiness now will make conditional logic in the next guideline far more predictable.

**`round()`** rounds a float to the nearest whole number, or to a specified number of decimal places when given a second argument.

```python
print(round(4.3))       # 4
print(round(4.7))       # 5
print(round(3.14159, 2))   # 3.14
```

Python's rounding carries one special rule that regularly surprises beginners: when a number sits **exactly** halfway between two possibilities, Python does not always round up. Instead, it rounds to whichever neighboring value is **even**, a convention called **round-half-to-even**, or informally, "banker's rounding."

```python
print(round(2.5))   # 2, rounds down to the nearest even number
print(round(3.5))   # 4, rounds up to the nearest even number
print(round(0.5))   # 0
print(round(1.5))   # 2
```

This is deliberate, not a bug: rounding every single halfway value upward, as many people learn in school, introduces a small but consistent upward bias when a large number of values are rounded in sequence, since ties always break the same direction. Round-half-to-even distributes that bias evenly instead, which matters in fields like finance and statistics where rounding many values consistently upward or downward can meaningfully skew a total over time, hence the "banker's" nickname. It is a rule worth simply knowing about in advance, since a beginner who does not expect it will otherwise assume `round()` is behaving inconsistently.

---
*End of Guideline 4.*

---
*Next: Guideline 5 — [To be defined]*
