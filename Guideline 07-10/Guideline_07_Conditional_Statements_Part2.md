# Guideline 7: Conditional Statements
## Part 2

Part 1 covered the mechanics of making a decision in Python: `if`, `else`, `elif`, combining conditions with `and` / `or`, and the difference between separate `if` statements and true nesting. Part 2 builds on that foundation in two directions. First, two built-in functions — `all()` and `any()` — that let you check a whole collection of conditions at once instead of writing them out one by one. Second, a set of practical, fully worked examples that put everything from this guideline to real use, closing with a short but genuinely useful piece of formal logic: De Morgan's Laws.

---

## 1. The `all()` and `any()` Functions

Guideline 6 introduced a number of built-in functions that operate on an entire iterable at once — `min()`, `max()`, `sum()`, and others. `all()` and `any()` belong to that same family, except instead of returning a number, they return a single `True` or `False` by looking at every Boolean value inside the iterable.

**`all()`** returns `True` only if *every single element* in the iterable is truthy. If even one element is `False` (or falsy), `all()` returns `False`.

```python
conditions = [True, True, True]
print(all(conditions))   # True

conditions = [True, False, True]
print(all(conditions))   # False
```

**`any()`** returns `True` if *at least one element* in the iterable is truthy. It only returns `False` if every single element is `False`.

```python
conditions = [False, False, True]
print(any(conditions))   # True

conditions = [False, False, False]
print(any(conditions))   # False
```

Where this actually becomes useful is when the list isn't hand-written like above, but is instead *generated* from a real check — most commonly using a list comprehension (a preview of a tool covered in full later in this book, but simple enough to use here already):

```python
numbers = [4, 8, 12, 16]

all_even = all(n % 2 == 0 for n in numbers)
print(all_even)   # True — every number in the list is even

numbers = [4, 7, 12, 16]
any_odd = any(n % 2 != 0 for n in numbers)
print(any_odd)    # True — at least one number (7) is odd
```

Naturally, both slot directly into an `if` statement, exactly like any other condition:

```python
scores = [72, 85, 91, 68]

if all(score >= 60 for score in scores):
    print("Everyone passed.")
else:
    print("At least one person failed.")

if any(score >= 90 for score in scores):
    print("At least one person scored a distinction.")
```

Conceptually, `all()` is what you get if you took every element and chained them together with `and`, and `any()` is what you get if you chained them together with `or` — except `all()` and `any()` scale to a list of any length, whereas writing out `and` by hand only works if you already know exactly how many conditions there are.

One behavior worth knowing, because it surprises people the first time they see it: **`all()` of an empty iterable is `True`**, and **`any()` of an empty iterable is `False`**.

```python
print(all([]))   # True
print(any([]))   # False
```

This isn't an arbitrary rule — it follows the same logic as the operators themselves. `all()` asks "is there any element that fails?" and if there are no elements at all, there is nothing to fail, so the answer defaults to `True`. `any()` asks "is there any element that succeeds?" and with no elements present, there is nothing that could have succeeded, so the answer defaults to `False`.

---

## 2. Practical Examples Using If-Else

The rest of this guideline is a set of complete, self-contained examples. Each one is a genuinely common task, and each one is built entirely out of tools already covered: `if`/`elif`/`else`, comparison operators, and `and`/`or`.

### 2.1 Odd or Even

The building block here is the modulo operator `%`, covered back in Guideline 3, which returns the remainder of a division. Any number divided by 2 leaves a remainder of either 0 or 1 — nothing else is possible — which makes it the natural test for evenness.

```python
number = int(input("Enter a whole number: "))

if number % 2 == 0:
    print(f"{number} is even.")
else:
    print(f"{number} is odd.")
```

`number % 2 == 0` reads naturally once broken apart: "the remainder of number divided by 2, is it equal to 0?" If it is, the number is even; if the remainder is 1 instead, the number is odd. Note this works correctly for negative numbers too — in Python, `-4 % 2` evaluates to `0`, so negative even numbers are still correctly identified as even.

### 2.2 Leap Year

A year is a leap year if it satisfies a slightly more layered rule than most people remember correctly: it must be divisible by 4 — *except* that century years (divisible by 100) are **not** leap years, *unless* they are also divisible by 400. That's why 2000 was a leap year but 1900 was not, even though both are divisible by 100.

```python
year = int(input("Enter a year: "))

if year % 4 == 0 and year % 100 != 0:
    print(f"{year} is a leap year.")
elif year % 400 == 0:
    print(f"{year} is a leap year.")
else:
    print(f"{year} is not a leap year.")
```

Walk through the logic carefully, because this is a good test of everything from Part 1. The first condition, `year % 4 == 0 and year % 100 != 0`, captures the ordinary case: divisible by 4, and *not* a century year. The `elif` only gets checked if that first condition failed — meaning the year either wasn't divisible by 4 at all, or it was a century year. Of those two possibilities, only the second one has any chance of still being a leap year, and only if it's divisible by 400, which the `elif` checks directly. If neither condition holds, `else` correctly reports that the year is not a leap year.

The same logic can be written as a single condition using `or`, which is worth seeing once you're comfortable with the version above:

```python
year = int(input("Enter a year: "))

if (year % 4 == 0 and year % 100 != 0) or year % 400 == 0:
    print(f"{year} is a leap year.")
else:
    print(f"{year} is not a leap year.")
```

Both versions are correct and produce identical results for every possible year — the first is arguably easier to read on a first pass; the second is more compact once the rule is familiar.

### 2.3 Number Divisibility (2, 3, 4, 5, and 9)

Unlike the odd/even check, a number can be divisible by *more than one* of these values at the same time — 36, for instance, is divisible by 2, 3, 4, and 9 all at once, but not by 5. That makes this a case for **separate, independent `if` statements**, not an `elif` chain, exactly as discussed in Part 1: an `elif` chain would only report the first true case and silently skip the rest.

```python
number = int(input("Enter a whole number: "))

if number % 2 == 0:
    print(f"{number} is divisible by 2.")
if number % 3 == 0:
    print(f"{number} is divisible by 3.")
if number % 4 == 0:
    print(f"{number} is divisible by 4.")
if number % 5 == 0:
    print(f"{number} is divisible by 5.")
if number % 9 == 0:
    print(f"{number} is divisible by 9.")

if not any([number % 2 == 0, number % 3 == 0, number % 4 == 0,
            number % 5 == 0, number % 9 == 0]):
    print(f"{number} is not divisible by 2, 3, 4, 5, or 9.")
```

The final block is where `any()` from earlier in this guideline earns its place: rather than manually checking "was every single one of the five messages above skipped," `any()` collapses all five conditions into one check, and `not` flips the result — if none of the five divisibility checks were true, print a message saying so.

As a small aside worth knowing: divisibility by 3 and by 9 both have a well-known shortcut that doesn't require division at all — a number is divisible by 3 (or 9) if the *sum of its digits* is divisible by 3 (or 9). For example, 4,617 has digits summing to 4+6+1+7 = 18, which is divisible by 9, and so is 4,617 itself. It's not needed for the code above — `%` already handles it directly — but it's a handy piece of number sense to have.

### 2.4 Grade Level (A to F)

This example brings back the grading logic from Part 1, but turns it into a complete, input-driven program with a boundary case handled explicitly: a score outside the valid 0–100 range shouldn't be graded at all.

```python
score = float(input("Enter a score (0-100): "))

if score < 0 or score > 100:
    print("Invalid score. Please enter a value between 0 and 100.")
elif score >= 90:
    print("Grade: A")
elif score >= 80:
    print("Grade: B")
elif score >= 70:
    print("Grade: C")
elif score >= 60:
    print("Grade: D")
else:
    print("Grade: F")
```

Putting the validity check first, as its own condition in the same `elif` chain, is a deliberate design choice: it guarantees that an invalid score can never accidentally fall through and get assigned a letter grade, because the moment the first condition is true, every other branch in the chain is skipped — this is exactly the "first true condition wins" behavior discussed in Part 1, and it's being used here on purpose, not just for correctness but for safety.

---

## 3. Applying Logic Rules: De Morgan's Laws

De Morgan's Laws are two rules from formal logic that describe exactly what happens when you push a `not` through an `and` or an `or`. They're worth learning properly here because they show up constantly once conditions start getting more complex, and because they let you rewrite an awkward, double-negative condition into something far more readable.

The two laws are:

> **not (A and B)** is the same as **(not A) or (not B)**
>
> **not (A or B)** is the same as **(not A) and (not B)**

In plain language: negating an "and" flips it into an "or" of the negations, and negating an "or" flips it into an "and" of the negations. Notice the operator itself switches — that's the entire content of the rule, and it's easy to forget under pressure, which is exactly why it's worth internalizing now.

Here it is verified directly in code, checking every possible combination of two Boolean values:

```python
for A in [True, False]:
    for B in [True, False]:
        left = not (A and B)
        right = (not A) or (not B)
        print(A, B, "->", left == right)
```

Running this prints `True` for every single combination of `A` and `B` — the two sides are logically identical in every case, which is what makes this a *law* rather than a coincidence that happens to work sometimes.

The practical value shows up when a condition is written as a negation of a compound expression, and you want to simplify it. Take the entry-check example from Part 1:

```python
age = 16
has_id = False

if not (age >= 18 and has_id):
    print("Entry denied.")
```

That reads a little awkwardly — "not (A and B)" forces you to mentally evaluate the inner condition first and then flip it. Applying De Morgan's Law rewrites it as:

```python
if age < 18 or not has_id:
    print("Entry denied.")
```

This is the same rule — `not (age >= 18)` becomes `age < 18`, `not (has_id)` stays as `not has_id`, and the `and` becomes an `or` — but it now reads as a direct list of the actual reasons entry might be denied, rather than a negated statement about when entry is allowed. Both versions produce identical results for every possible value of `age` and `has_id`; the second is simply easier for a human reading the code to follow at a glance.

The same transformation works in the other direction too. A condition like `not (day == "Saturday" or day == "Sunday")` — "not a weekend day" — becomes `day != "Saturday" and day != "Sunday"` under the second law: not Saturday, and also not Sunday. Whenever you find yourself writing `not (...)` around a compound `and`/`or` condition, it's worth pausing to check whether De Morgan's Law would let you say the same thing without the outer negation at all.
