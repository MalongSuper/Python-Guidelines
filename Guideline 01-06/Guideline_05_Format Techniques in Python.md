# Python Guidebook
## Guideline 5: Format Techniques in Python

---

Every example so far in this book has combined text and values inside `print()` using commas, but that is only the most basic of several ways Python lets you build formatted output. As programs grow past a single line, the difference between output that is merely correct and output that is actually readable starts to matter, and Python offers a progression of tools for exactly that, from the simplest to the most capable.

### 5.1 Combining Values: Comma, %, and +

**The comma**, already used throughout this book, is the simplest option. Passing multiple arguments to `print()`, separated by commas, lets Python handle the joining for you, inserting a space (or whatever `sep` is set to, from Guideline 2) between each piece automatically.

```python
name = "Iris"
age = 5
print("Name:", name, "Age:", age)   # Name: Iris Age: 5
```

**The `%` operator**, sometimes called "printf-style" formatting because it borrows its approach from the C language, embeds placeholders directly inside a string and fills them in from a set of values supplied afterward. The two placeholders you will see most often are `%s` (insert as a string) and `%d` (insert as a decimal integer):

```python
name = "Iris"
age = 5
print("Name: %s, Age: %d" % (name, age))   # Name: Iris, Age: 5
```

This is older syntax that predates both of the methods covered later in this guideline, and you will still encounter it in existing codebases, but it is no longer the recommended approach for new code, largely because the placeholders must match the order and type of the values exactly, with little flexibility.

**The `+` operator** concatenates strings directly, the same concatenation introduced back in Guideline 4. Its major limitation is that `+` only works when every value involved is already a string; unlike the comma or `%`, it will not silently convert a number for you.

```python
name = "Iris"
age = 5
print("Name: " + name + ", Age: " + str(age))   # Name: Iris, Age: 5
```

Notice `str(age)` doing the work there. Leaving it out and writing `"Age: " + age` directly would raise a `TypeError`, since Python refuses to guess whether you meant to combine a number with text or actually perform an operation between two unrelated types. This is the main reason `+` is used less often for formatting full lines of output than the comma or the tools below, even though it remains the standard way to join strings that are already known to be strings.

### 5.2 The .format() Method

The **`.format()` method** improves on `%` formatting by using named or numbered placeholders, written as curly braces `{}`, directly inside a string, with the actual values supplied as arguments to `.format()` itself.

```python
name = "Iris"
age = 5
print("Name: {}, Age: {}".format(name, age))   # Name: Iris, Age: 5
```

Placeholders can also be numbered, which allows the same value to be reused more than once, or reordered without changing the arguments themselves, and they can be named, which makes a long string with many placeholders considerably easier to read and verify:

```python
print("{0} is {1}. {0} loves coding.".format(name, age))
print("Name: {n}, Age: {a}".format(n=name, a=age))
```

`.format()` was, for a long time, the recommended replacement for `%` formatting, and it remains fully supported and reasonably common in existing Python code, but it has largely been superseded for new code by the tool covered next.

### 5.3 The f-string

An **f-string** (formatted string literal) is written by placing an `f` directly before the opening quotation mark. Inside an f-string, any expression wrapped in curly braces `{}` is evaluated immediately and inserted into the string, without needing a separate `.format()` call at all.

```python
name = "Iris"
age = 5
print(f"Name: {name}, Age: {age}")            # Name: Iris, Age: 5
print(f"Next year, {name} will be {age + 1}")   # Next year, Iris will be 6
```

That second example is worth noticing closely: f-strings do not just insert a variable's value, they evaluate any valid expression placed inside the braces, including calculations, meaning `age + 1` is computed on the spot rather than requiring a separate variable to be created first. Because of this combination of brevity and power, f-strings are the modern, generally preferred way to build formatted strings in Python.

F-strings also support **format specifiers**: an optional `:` followed by a code inside the braces, controlling exactly how a value should be displayed.

- **`.f`** displays a floating-point number with a fixed number of decimal places. `.2f` means "two digits after the decimal point," rounding as needed.

  ```python
  pi = 3.14159
  print(f"{pi:.2f}")   # 3.14
  ```

- **`d`** formats a value as a plain decimal integer.

  ```python
  quantity = 12
  print(f"{quantity:d}")   # 12
  ```

- **`.e`** displays the number in exponential (scientific) notation, useful for very large or very small values where a plain decimal form would be hard to read.

  ```python
  distance = 149600000
  print(f"{distance:.2e}")   # 1.50e+08
  ```

- **`.0%`** multiplies the value by 100 and displays it as a percentage, rounded to zero decimal places. This is the one most likely to catch a beginner off guard, since it expects a decimal fraction as input, not an already-multiplied number.

  ```python
  ratio = 0.256
  print(f"{ratio:.0%}")   # 26%
  ```

These same specifiers work identically inside `.format()`'s curly braces (`"{:.2f}".format(pi)`), since both tools share the same underlying formatting mini-language; f-strings have simply made writing them faster and more direct.

### 5.4 Using Escape Characters

An **escape character** is a special two-character sequence, starting with a backslash `\`, that represents a character which either cannot be typed directly into a string or would otherwise be interpreted as ending the string early.

| Sequence | Meaning | Effect |
| --- | --- | --- |
| `\n` | Newline | Moves the following text to a new line |
| `\t` | Tab | Inserts a horizontal tab space |
| `\r` | Carriage return | Moves the cursor back to the start of the current line |
| `\b` | Backspace | Removes the character immediately before it |

```python
print("Line one\nLine two")     # prints on two separate lines
print("Name:\tIris")            # inserts a tab between "Name:" and "Iris"
print("Hello\bWorld")            # World, printed with the "o" removed by the backspace
```

`\r` is the least intuitive of the four: rather than starting a new line, it moves the cursor back to the very beginning of the current line, which means whatever is printed after it can overwrite characters that were already there. This behavior is genuinely useful for things like a text-based progress indicator that updates in place on a single line rather than printing a new line for every update, but it can also look like nothing happened at all when tested inside certain consoles and notebook environments that handle `\r` differently, which is worth knowing before assuming it is broken.

### 5.5 Comments

A **comment** is text in your source code that Python deliberately ignores when the program runs. Comments exist entirely for the benefit of whoever reads the code, not the computer executing it, whether that is a collaborator, a future version of yourself, or an instructor reviewing your work.

Python provides two forms, suited to two different jobs.

**The `#` symbol** starts a single-line comment, ending automatically at the end of that line. It is the right tool for a short, quick clarification attached directly to the line of code it describes:

```python
tax_rate = 0.07   # sales tax rate, update if local law changes
```

**Triple quotes** (`"""` or `'''`), spanning as many lines as needed, are the right tool for a longer explanation, one that would be awkward to squeeze onto a single line or that needs to explain several related lines at once, such as describing what an entire block of code, or an entire program, is meant to do before it begins.

```python
"""
This script asks the user for two numbers and prints
the result of every basic arithmetic operation between them.
Written for Guideline 2's worked example.
"""

num1 = float(input("Enter the first number: "))
num2 = float(input("Enter the second number: "))
```

One technical note worth being upfront about: a triple-quoted block like this is not, strictly speaking, a dedicated comment syntax the way `#` is. It is an ordinary string literal that Python happens to evaluate and then discard, since nothing is done with it afterward. When placed as the very first statement inside a function, class, or module, this same triple-quoted string is given a special name, a **docstring**, and tools can actually retrieve and display it as that object's documentation, which is a use covered in more depth once functions are introduced later in this book. For now, it is enough to know it as the practical way to write a comment longer than a single line.

---
*End of Guideline 5.*

---
*Next: Guideline 6 — [To be defined]*
