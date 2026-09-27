# Python Guidebook
## Guideline 6: Built-in Methods

---

Every guideline so far has quietly relied on tools that were never explicitly explained: `print()`, `input()`, `int()`, `round()`, `type()`. These are all examples of **built-in functions**, pieces of Python code that ship with the language itself, already written and ready to use, so that you never have to build basic, universally needed operations from scratch. Underneath, a built-in function is genuinely just Python code, the same kind you are learning to write, except that it was written once by the language's maintainers and made permanently available to every program.

Before going further, one distinction is worth making explicit, since this guideline mixes both kinds freely: a **function** is called on its own, by name, with any values it needs passed inside parentheses, like `len(text)`. A **method** is a function that belongs to a specific value and is called using dot notation directly on that value, like `text.upper()`. Both are "built-in" in the sense that neither requires you to write the underlying logic yourself, but they are written and called differently, and the difference matters once you start guessing how to call something you have not used before.

This guideline is a broad tour of the built-ins you will keep running into for the rest of this book. A few of them, especially `map()` and `filter()`, and the collection-conversion functions like `list()` and `dict()`, touch on ideas (functions as values, and collection data types) that get a full, dedicated explanation later; here, they are introduced only far enough to be recognized on sight.

### 6.1 The Familiar Two

`print()` and `input()` have appeared in every guideline since Guideline 2. They are worth naming here specifically as built-in functions, because they establish the pattern every other entry on this list follows: they are not special language keywords, they are ordinary functions, written and maintained by Python itself, that simply happen to be so fundamental they are available immediately, without any setup.

### 6.2 Shortcuts for Multiple Values

**`eval()`** takes a string and evaluates it as if it were actual Python code, returning the result.

```python
result = eval("2 + 3 * 4")
print(result)   # 14
```

`eval()` deserves a direct warning alongside its introduction: because it executes whatever text it is given as real, live code, running `eval()` on text that comes from an untrusted source, most dangerously, direct user input, means allowing that source to run arbitrary code inside your program. This is a genuine security risk in real applications, not a theoretical one, and it is why `eval()` is used sparingly in professional code and essentially never on unfiltered user input.

**`map()`** applies a given function to every item in an iterable (a value that can be looped over, such as a string or a list) and hands back the transformed results.

```python
numbers = ["1", "2", "3"]
converted = list(map(int, numbers))
print(converted)   # [1, 2, 3]
```

Notice the `list()` wrapped around `map(...)`. In Python 3, `map()` does not immediately produce a visible list; it produces a `map` object that generates its results only as they are requested, which is efficient but means you must convert it, usually with `list()`, before you can print or inspect it directly.

**`.split()`** breaks a string apart into a list of smaller strings, dividing at whitespace by default, or at a specific character if one is provided.

```python
sentence = "Learning Python is fun"
words = sentence.split()
print(words)   # ['Learning', 'Python', 'is', 'fun']

date = "2026-09-26"
parts = date.split("-")
print(parts)   # ['2026', '09', '26']
```

### 6.3 Daily-Use Utility Tools

These next five are among the most frequently used built-ins in ordinary Python code, almost all of them operating on a collection of values such as a list, a bracketed group of comma-separated values that will be covered fully in its own guideline, but simple enough to use here on sight.

```python
scores = [88, 92, 79, 95, 84]
```

- **`min()`** and **`max()`** return the smallest and largest value in a collection. `print(min(scores))` gives `79`; `print(max(scores))` gives `95`.
- **`round()`**, already covered in Guideline 4, rounds a number to the nearest whole number or to a specified number of decimal places.
- **`len()`** returns how many items a collection contains. `print(len(scores))` gives `5`.
- **`sum()`** adds every numeric value in a collection together. `print(sum(scores))` gives `438`.
- **`sorted()`** returns a brand new list containing the same values in ascending order, without changing the original. `print(sorted(scores))` gives `[79, 84, 88, 92, 95]`; passing `reverse=True` sorts it in descending order instead.

### 6.4 Checking and Converting Types

`type()`, `str()`, `float()`, `int()`, and `bool()` have already appeared across Guidelines 2 and 4: `type()` reports a value's data type, and the other four convert a value into that specific type. Four more conversion functions round this group out, each one turning an iterable into a specific kind of collection: `list()` produces an ordered, changeable list; `tuple()` produces an ordered but unchangeable tuple; `set()` produces an unordered collection with no duplicate values; and `dict()` constructs a dictionary of key-value pairs. These four collection types each get a full guideline of their own later in this book; for now, it is enough to recognize that these functions exist and each produce a distinctly different kind of result from the same input.

```python
text = "abc"
print(list(text))    # ['a', 'b', 'c']
print(tuple(text))   # ('a', 'b', 'c')
print(set(text))      # {'a', 'b', 'c'}
```

### 6.5 reversed() and filter()

**`reversed()`** returns the items of an iterable in reverse order, again as a special iterator object rather than a directly printable result, so it is typically wrapped in `list()` just like `map()`.

```python
numbers = [1, 2, 3, 4]
print(list(reversed(numbers)))   # [4, 3, 2, 1]
```

**`filter()`** keeps only the items from an iterable that satisfy a given condition, discarding the rest. Like `map()`, it takes a function as its first argument, and here that function is often written as a quick, throwaway **lambda**, a small unnamed function covered fully in a later guideline, but readable enough in its simplest form to use here: `lambda x: x > 80` simply means "given a value named `x`, check whether `x` is greater than 80."

```python
scores = [88, 92, 79, 95, 84]
passing = list(filter(lambda x: x > 80, scores))
print(passing)   # [88, 92, 95, 84]
```

### 6.6 Character and Number Conversions

Every character a computer displays is, underneath, stored as a number, a connection first raised back in Guideline 1's discussion of binary. **`ord()`** returns that underlying numeric code (its Unicode, or for common English characters, ASCII value) for a single character, and **`chr()`** does the reverse, converting a numeric code back into its corresponding character.

```python
print(ord("A"))   # 65
print(chr(65))     # A
```

**`ascii()`** returns a string representation of a value that is guaranteed to be safe to display using only standard ASCII characters, automatically escaping any character outside that range rather than printing it directly.

```python
print(ascii("café"))   # 'caf\xe9'
```

### 6.7 Boolean Aggregation

**`any()`** and **`all()`** both take an iterable of values and reduce it down to a single Boolean, building directly on the truthiness rule introduced with `bool()` in Guideline 4. `any()` returns `True` if at least one value in the iterable is truthy; `all()` returns `True` only if every single value is.

```python
results = [True, False, True]
print(any(results))   # True, at least one is True
print(all(results))    # False, not every value is True
```

### 6.8 id() and divmod()

**`id()`** returns a number representing a value's identity, its specific location in memory during that program's run, the same underlying concept that Guideline 3's `is` operator checks when comparing whether two variables reference the exact same object.

```python
x = [1, 2, 3]
print(id(x))   # some large integer, unique to this object during this run
```

**`divmod()`** takes two numbers and returns both the floor division result and the modulus result together, as a single pair, rather than requiring two separate calculations with `//` and `%` as in Guideline 3.

```python
print(divmod(17, 5))   # (3, 2), meaning 17 // 5 = 3 and 17 % 5 = 2
```

### 6.9 Number Base Conversions

**`bin()`**, **`hex()`**, and **`oct()`** convert a decimal integer into a string showing that same number in binary, hexadecimal, or octal form, each prefixed to indicate which base is being shown (`0b` for binary, `0x` for hexadecimal, `0o` for octal). These tie directly back to Guideline 1's discussion of binary as the computer's native representation, and to Guideline 3's bitwise operators, which operate on exactly this binary form.

```python
print(bin(10))   # 0b1010
print(hex(255))   # 0xff
print(oct(8))      # 0o10
```

### 6.10 String and List Methods

The remaining entries are **methods**, called with dot notation directly on the value they act upon, rather than standalone functions.

- **`.sort()`** sorts a list in place, permanently rearranging the original list itself rather than producing a new one.
- **`.pop(i)`** removes and returns the item at a given index from a list.
- **`.upper()`** and **`.lower()`** return a copy of a string converted entirely to uppercase or lowercase.
- **`.reverse()`** reverses a list in place, permanently altering the original list's order.
- **`.replace(old, new)`** returns a copy of a string with every occurrence of `old` swapped out for `new`.

```python
scores = [88, 92, 79, 95, 84]
scores.sort()
print(scores)   # [79, 84, 88, 92, 95], the original list itself is now sorted

removed = scores.pop(0)
print(removed, scores)   # 79 [84, 88, 92, 95]

name = "Iris"
print(name.upper())   # IRIS
print(name.lower())    # iris

scores.reverse()
print(scores)   # [95, 92, 88, 84]

message = "I like cats"
print(message.replace("cats", "dogs"))   # I like dogs
```

**A distinction worth locking in now, since it is the single most common source of confusion between this section and 6.3:** `sorted(a_list)` and `a_list.sort()` sound like they do the same thing, and produce the same ordering, but they behave very differently underneath. `sorted()` is a function that leaves the original list completely untouched and hands back a brand new, separately sorted list. `.sort()` is a method that changes the original list directly and returns nothing at all (`None`) to signal that the change happened in place rather than producing a new value. Assigning the result of `.sort()` back to a variable, expecting it to behave like `sorted()`, is a mistake beginners make often enough that it is worth remembering deliberately: if you need the original order preserved somewhere, use `sorted()`; if you are fine permanently rearranging the list you already have, `.sort()` is the more direct tool.

---
*End of Guideline 6.*

---
*Next: Guideline 7 — [To be defined]*
