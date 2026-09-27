# Python Guidebook
## Guideline 2: Understanding Python Syntaxes

---

### 2.1 The print() Statement

`print()` is the first piece of real Python syntax most people ever type, and it remains one of the most frequently used statements in the language, whether you are debugging a large program or simply displaying a result. At its simplest, it writes whatever you give it out to the console:

```python
print("Hello, world!")
```

`print()` is not limited to fixed text. You can hand it a calculation directly, and Python will evaluate the expression first and print the result:

```python
print(5 + 3)        # 8
print(12 * 4)        # 48
print("Total: ", 5 + 3)   # Total:  8
```

Notice that last line: `print()` accepts multiple arguments separated by commas, and it will print each one in order, joined together with a separator in between. By default, that separator is a single space, and the whole line ends with a newline character once printing is finished, which is why each call to `print()` normally starts on a fresh new line the next time you call it.

Both of these defaults can be changed using two keyword arguments: `sep` and `end`.

- **`sep`** controls what is placed between multiple arguments in the same `print()` call. It defaults to a single space (`" "`), but you can set it to anything you like, including an empty string.
- **`end`** controls what is placed after everything has been printed, instead of the default newline (`"\n"`). This is useful when you want several `print()` calls to appear on the same line rather than stacking downward.

```python
print("2026", "09", "26", sep="-")        # 2026-09-26
print("Loading", end="...")
print("done")                              # Loading...done
```

Without the `sep` argument, the first line above would have printed as `2026 09 26` with plain spaces. Without the `end` argument, the second example would have printed `Loading` and `done` on two separate lines instead of one continuous line. Small as they seem, `sep` and `end` come up constantly once you start formatting output for a person to actually read, rather than just confirming a value exists.

### 2.2 Variables

A **variable** is a named location where a value is stored, so that the value can be reused, updated, or referred to by a meaningful name instead of being retyped every time it is needed.

```python
radius = 4
area = radius * radius * 3.14159
```

That example works, but it hides a problem worth naming directly: **`3.14159` is a magic number**. A magic number is a literal value typed directly into code with no explanation of what it represents or why that specific value was chosen. Anyone reading `radius * radius * 3.14159` six months from now, including you, has to stop and infer that this is an approximation of pi, rather than simply being told. Magic numbers also create maintenance risk: if the same unexplained value is used in five different places and needs to change, you have to hunt down every occurrence individually, and it is easy to miss one. The fix is to store the value in a clearly named variable once, and reference that name everywhere instead:

```python
PI = 3.14159
radius = 4
area = radius * radius * PI
```

Now the calculation reads as what it actually means, and if a more precise value of pi is ever needed, it only has to change in one place. This same principle applies to any unexplained literal, not just mathematical constants: a hardcoded `86400` is much clearer written as `SECONDS_IN_A_DAY`, and a hardcoded `0.07` tax rate is much clearer written as `TAX_RATE`.

Python does not let you name a variable anything you like; a handful of rules govern what is and is not a legal name:

- **A variable name cannot start with a number.** It must begin with a letter or an underscore. `2nd_place` is invalid; `second_place` or `_2nd_place` are fine.
- **A variable name cannot contain spaces.** `my variable` is invalid. Use an underscore (`my_variable`) or capitalize each subsequent word (`myVariable`) instead.
- **A variable name cannot be a Python keyword.** Keywords are words reserved by the language itself for its own grammar, such as `if`, `else`, `for`, `while`, `def`, `class`, `return`, `import`, `True`, `False`, and `None`. Using one of these as a variable name produces a syntax error, because Python cannot tell whether you mean the keyword or a variable.
- **A variable name should avoid shadowing a built-in function or module name.** This one will not cause an immediate error the way the rules above do, which is exactly what makes it dangerous. Python ships with built-in functions like `print`, `input`, `list`, `str`, and `sum` already available everywhere. If you write `list = [1, 2, 3]`, that assignment succeeds, but it silently overwrites your access to the real `list()` function for the rest of that program, and any later line that tries to use `list()` as a function will now fail in a confusing way that has nothing obviously to do with the variable you named far earlier.

```python
# Invalid names
2nd_score = 90        # starts with a number
my score = 90          # contains a space
class = "Beginner"     # "class" is a reserved keyword

# Valid, but risky
str = "hello"          # shadows the built-in str() function

# Valid and safe
second_score = 90
my_score = 90
skill_level = "Beginner"
greeting = "hello"
```

Good variable naming is not a cosmetic habit. It is one of the most direct ways a beginner's code and an experienced programmer's code visibly differ, long before either one demonstrates any deeper technical skill.

### 2.3 The input() Statement

`input()` pauses your program, displays an optional prompt message to the user, and waits for them to type something and press Enter. Whatever they typed is then handed back to your program as the result of the `input()` call.

```python
name = input("What is your name? ")
print("Hello,", name)
```

There is one critical detail that catches nearly every beginner at least once: **`input()` always returns a string**, regardless of what the user actually typed. If you ask someone for their age and they type `25`, Python does not hand you back the number `25`. It hands you back the text `"25"`, which looks identical when printed but behaves completely differently in a calculation.

```python
age = input("Enter your age: ")   # age is the string "25", not the number 25
print(age + 5)                     # this raises an error: strings and numbers can't be added directly
```

To actually use numeric input in a calculation, you must explicitly convert it using **`int()`** for whole numbers or **`float()`** for decimal numbers, a process called type casting:

```python
age = int(input("Enter your age: "))
print(age + 5)     # works correctly, because age is now an integer

price = float(input("Enter the price: "))
print(price * 1.08)   # works correctly, because price is now a float
```

If the text typed cannot actually be converted, for instance if someone types `"twenty-five"` instead of `25`, `int()` and `float()` will raise a `ValueError` rather than guessing. Handling that gracefully involves error handling, which is covered later in this book; for now, the important habit to build is simply remembering that anything coming out of `input()` is text until you deliberately convert it into the type you actually need.

### 2.4 Examples

**Example 1: Asking for a name and age, and printing them back**

```python
name = input("What is your name? ")
age = int(input("How old are you? "))

print("Hello,", name, end="! ")
print("You are", age, "years old.")
```

Here, `name` is left as a string, since a name has no reason to be converted into a number, while `age` is immediately wrapped in `int()` at the moment it is captured, since it will need to behave like a number the moment it is used in a calculation. The `end="! "` argument on the first `print()` call keeps the greeting and the age statement on the same visible line rather than stacking them on separate lines.

**Example 2: Taking two numbers and performing every basic arithmetic operation**

```python
num1 = float(input("Enter the first number: "))
num2 = float(input("Enter the second number: "))

print("Addition:", num1 + num2)
print("Subtraction:", num1 - num2)
print("Multiplication:", num1 * num2)
print("Division:", num1 / num2)
print("Floor Division:", num1 // num2)
```

Both inputs are cast with `float()` rather than `int()`, since the numbers a user enters are not guaranteed to be whole, and using `float()` keeps the program working correctly whether someone types `10` or `10.5`. Four of the five operations behave exactly as expected from ordinary arithmetic: addition (`+`), subtraction (`-`), multiplication (`*`), and standard division (`/`), which always returns a decimal (float) result even when the numbers divide evenly. The fifth, **floor division (`//`)**, behaves differently on purpose: it performs the division and then discards anything after the decimal point, returning only the whole number of times the first value divides into the second. Given `num1 = 7` and `num2 = 2`, `num1 / num2` returns `3.5`, while `num1 // num2` returns `3.0`, the same calculation with the fractional remainder deliberately dropped. This distinction becomes important later whenever a program needs a count of whole units, such as how many full boxes are needed to pack a given number of items, rather than a precise decimal answer.

---
*End of Guideline 2.*

---
*Next: Guideline 3 — [To be defined]*
