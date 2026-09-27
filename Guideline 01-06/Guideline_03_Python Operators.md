# Python Guidebook
## Guideline 3: Python Operators

---

### 3.1 Arithmetic Operators

Guideline 2 already put four of these to work without naming them as a group: addition, subtraction, multiplication, and floor division. Python groups these together with three more under the label **arithmetic operators**, the symbols used to perform mathematical calculations on numeric values.

| Operator | Meaning | Example |
| --- | --- | --- |
| `+` | Addition | `a + b` |
| `-` | Subtraction | `a - b` |
| `*` | Multiplication | `a * b` |
| `/` | Division | `a / b` |
| `//` | Floor Division | `a // b` |
| `%` | Modulus (Remainder) | `a % b` |
| `**` | Exponentiation (Power) | `a ** b` |

The two new arrivals here are `%` and `**`. **Modulus (`%`)** returns whatever is left over after division, rather than the result of the division itself:

```python
print(10 % 3)   # 1, because 10 divided by 3 is 3 with 1 left over
print(9 % 3)    # 0, because 9 divides evenly into 3
```

Modulus is the standard way to answer questions like "is this number even or odd?" (`n % 2 == 0` for even), or "has this counter reached a multiple of five yet?", both of which will come up repeatedly once loops are introduced later in this book.

**Exponentiation (`**`)** raises the first value to the power of the second:

```python
print(2 ** 3)    # 8, because 2 * 2 * 2 = 8
print(5 ** 2)    # 25, because 5 * 5 = 25
```

It is easy to mistake `**` for multiplication at a glance, especially coming from other contexts where `^` is used for exponents instead. In Python, `^` means something entirely different, covered later in this guideline under bitwise operators, so `**` is the one to reach for whenever a power calculation is needed.

### 3.2 Comparison Operators

Where arithmetic operators produce a number, **comparison operators** produce a **Boolean** value, either `True` or `False`, based on how two values relate to each other. These are the backbone of the Conditional Statement guideline coming later in this book, since every `if` statement ultimately depends on a comparison resolving to `True` or `False`.

| Operator | Meaning | Example |
| --- | --- | --- |
| `>` | Greater than | `a > b` |
| `<` | Less than | `a < b` |
| `==` | Equal to | `a == b` |
| `!=` | Not equal to | `a != b` |
| `>=` | Greater than or equal to | `a >= b` |
| `<=` | Less than or equal to | `a <= b` |

```python
age = 20
print(age > 18)    # True
print(age == 21)   # False
print(age != 21)   # True
```

**A clarification that matters more than it looks:** `=` and `==` are not interchangeable, even though they differ by a single character. **`=` is the assignment operator.** It stores a value into a variable and does not compare anything at all:

```python
age = 20   # store the value 20 into the variable "age"
```

**`==` is the equality comparison operator.** It checks whether two values are the same and produces a Boolean result, without changing anything:

```python
print(age == 20)   # True, this is a question being asked, not a value being stored
```

Confusing the two is one of the most common early mistakes in Python, and it usually happens inside an `if` statement, where a beginner intends to ask a question but accidentally writes an assignment instead. Python is strict enough that `if age = 20:` will not silently do the wrong thing; it raises a `SyntaxError`, because assignment is a statement, not an expression that can produce a value for `if` to evaluate. The fix is simply to remember which symbol asks and which symbol stores: one equals sign sets a value, two equals signs check one.

### 3.3 Boolean Operators

The result of every comparison in the previous section is a value of Python's **`bool`** type, which has exactly two possible values: **`True`** and **`False`**, always capitalized. Booleans are not a cosmetic label on top of ordinary values, either; in Python, `True` and `False` are actually a specialized form of integers, where `True` behaves as `1` and `False` behaves as `0`:

```python
print(True + True)     # 2
print(False * 10)      # 0
print(True == 1)        # True
```

This is rarely something you will deliberately exploit as a beginner, but it explains behavior you may otherwise find confusing later, such as why summing a list of Boolean values gives you a count of how many were `True`.

### 3.4 Logical Operators

Where Boolean operators describe the `True`/`False` values themselves, **logical operators** combine multiple `True`/`False` results into a single overall answer. Python provides three: **`and`**, **`or`**, and **`not`**.

- **`and`** produces `True` only if both sides are `True`.
- **`or`** produces `True` if at least one side is `True`.
- **`not`** flips a Boolean value to its opposite.

```python
age = 20
has_id = True

print(age >= 18 and has_id)   # True, both conditions are True
print(age >= 65 or has_id)     # True, at least one condition is True
print(not has_id)               # False, the opposite of True
```

This is how a single condition in real code usually becomes several conditions combined, for instance checking that a user is old enough *and* has provided identification before granting access to something, rather than checking either fact in isolation.

### 3.5 Bitwise Operators

Bitwise operators reach back to Guideline 1's discussion of binary. Instead of operating on a number's ordinary decimal value, **bitwise operators** act directly on the individual bits, the raw 0s and 1s, that make up a number's binary representation.

| Operator | Meaning | Example |
| --- | --- | --- |
| `&` | Bitwise AND | `a & b` |
| `\|` | Bitwise OR | `a \| b` |
| `~` | Bitwise NOT | `~a` |
| `^` | Bitwise XOR | `a ^ b` |
| `>>` | Right Shift | `a >> 2` |
| `<<` | Left Shift | `a << 2` |

`&` and `\|` compare a number's bits position by position: `&` keeps a `1` only where both numbers have a `1` in that position, and `\|` keeps a `1` where either number has a `1`. `^` (XOR) keeps a `1` only where the two numbers *disagree* at that position. `~` flips every bit of a single number to its opposite. `<<` and `>>` shift a number's bits left or right by a given number of places, which has the effect of multiplying or dividing the number by powers of two.

```python
a = 6    # binary: 0110
b = 3    # binary: 0011

print(a & b)    # 2, binary 0010
print(a | b)    # 7, binary 0111
print(a ^ b)    # 5, binary 0101
print(a << 1)   # 12, binary 1100 (shifted left by one place)
print(a >> 1)   # 3, binary 0011 (shifted right by one place)
```

Bitwise operators are used far less often in everyday application code than arithmetic or comparison operators, but they show up regularly in low-level programming, working with flags and permission systems, graphics, and certain performance-sensitive calculations, so it is worth being able to recognize them even before you have a regular use for them.

### 3.6 Assignment Operators

The plain `=` from section 3.2 has a family of shorthand variants, called **augmented assignment operators**, that combine a calculation with a reassignment in a single step. Instead of writing `a = a + 5`, you can write `a += 5`, and Python will take the current value of `a`, add 5 to it, and store the result back into `a`, all in one line.

| Operator | Meaning | Example |
| --- | --- | --- |
| `=` | Assign value | `a = 5` |
| `+=` | Add and assign | `a += 5` |
| `-=` | Subtract and assign | `a -= 5` |
| `*=` | Multiply and assign | `a *= 5` |
| `/=` | Divide and assign | `a /= 5` |
| `//=` | Floor divide and assign | `a //= 5` |
| `%=` | Modulus and assign | `a %= 5` |
| `**=` | Exponent and assign | `a **= 5` |
| `<<=` | Left shift and assign | `a <<= 2` |
| `>>=` | Right shift and assign | `a >>= 2` |

```python
score = 10
score += 5    # same as score = score + 5
print(score)   # 15

score *= 2     # same as score = score * 2
print(score)   # 30
```

These are used constantly once loops enter the picture, since a loop very often needs to update the same variable, a running total, a counter, a score, on every single pass, and `total += value` inside a loop body is far more common in real code than writing out `total = total + value` in full each time.

### 3.7 Identity Operators

**`is`** and **`is not`** ask a different question than `==` does. `==` checks whether two values are *equal*. `is` checks whether two variables refer to the exact same object in memory, the same underlying piece of data, not merely two separate pieces of data that happen to hold equal values.

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)   # True, they hold equal values
print(a is b)   # False, they are two separate list objects in memory
print(a is c)   # True, "c" points to the very same object as "a"
```

For simple values like small integers and short strings, `is` can sometimes appear to behave like `==` due to internal optimizations Python performs, which makes it tempting to use them interchangeably. Resist that temptation: `is` is intended for identity checks, most commonly comparing something against Python's special `None` value (`if result is None:`), and using it as a substitute for `==` on ordinary values will eventually produce a bug that is confusing to track down.

### 3.8 Membership Operators

**`in`** and **`not in`** check whether a value exists inside a sequence, such as a string, a list, or another collection, without you having to manually search through it yourself.

```python
name = "Python"
print("P" in name)        # True
print("z" in name)        # False

allowed_users = ["alice", "bob", "carol"]
print("dave" not in allowed_users)   # True
```

Membership operators read almost like plain English, which is very much the point: `"dave" not in allowed_users` communicates its own intent clearly enough that it barely needs a comment explaining what it does. They will reappear frequently once lists, strings, and other collections are covered in depth later in this book, since checking whether something belongs to a group is one of the most common questions a program needs to answer.

---
*End of Guideline 3.*

---
*Next: Guideline 4 — [To be defined]*
