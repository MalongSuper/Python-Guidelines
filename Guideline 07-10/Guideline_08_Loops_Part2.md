# Guideline 8: Loops
## Part 2

Part 1 covered the two loop structures Python provides — `while` and `for` — along with what happens when one loop is nested inside another. Part 2 covers how to change a loop's behavior *while it's running*, two convenience tools that make certain kinds of loops far more readable, and then spends most of its length on worked examples, because loops are one of those topics that only really click once you've built several real things with them. The section on combining loops with conditional statements is marked as important for a reason: it is, without much exaggeration, the single most common pattern in all of practical programming, and nearly every example in this guideline uses it.

---

## 1. Break and Continue

So far, a loop has always run to natural completion — a `while` loop stops when its condition becomes false, and a `for` loop stops when its sequence runs out. `break` and `continue` are two keywords that let you interrupt that natural flow from *inside* the loop body, based on something you discover mid-iteration.

### `break`

`break` immediately exits the loop it's inside — completely, and with no further iterations at all, regardless of whether the loop's condition is still true or the sequence still has items left.

```python
for number in range(1, 11):
    if number == 6:
        break
    print(number)
```

This prints `1, 2, 3, 4, 5` and then stops entirely — the moment `number` reaches 6, `break` fires, and the loop ends right there. Numbers 7 through 10 are never even checked.

`break` is especially useful for search-style logic, where you want to stop looking the instant you've found what you're after, rather than needlessly continuing to check everything else:

```python
target = "o"
word = "python"

for letter in word:
    if letter == target:
        print(f"Found '{target}'!")
        break
```

### `continue`

`continue` does something more limited: it skips the *rest of the current iteration's body* and jumps straight to the next iteration — for a `for` loop, that means moving to the next item in the sequence; for a `while` loop, that means jumping back to re-check the condition. Unlike `break`, the loop itself is not stopped — it keeps going, just without finishing whatever code came after `continue` in that particular pass.

```python
for number in range(1, 11):
    if number % 2 == 0:
        continue
    print(number)
```

This prints only the odd numbers — `1, 3, 5, 7, 9`. Every time `number` is even, `continue` skips the `print(number)` line for that iteration specifically, but the loop still moves on to check `number + 1` next, exactly as it would have anyway.

**A caution about nested loops:** `break` and `continue` only ever affect the *nearest enclosing loop* — the one they're directly written inside. If a `break` sits inside an inner loop that itself is nested inside an outer loop, it exits only the inner loop; the outer loop is completely unaffected and continues on to its next iteration as normal.

```python
for i in range(3):
    for j in range(3):
        if j == 1:
            break
        print(f"i={i}, j={j}")
```

Here, the `break` fires every time `j` reaches 1, but it only ever exits the *inner* loop for that particular pass of `i`. The outer loop doesn't know or care that the inner loop was cut short — it moves on to the next value of `i` regardless, and the inner loop starts fresh (`j` back at 0) each time.

---

## 2. Enumerate (Commonly Used for Lists)

A plain `for` loop over a sequence gives you each *value*, one at a time, but it doesn't automatically tell you *where* in the sequence that value came from. Very often, you want both — the position and the value together. That's exactly what the built-in `enumerate()` function provides.

```python
word = "python"

for index, letter in enumerate(word):
    print(index, letter)
```

`enumerate(word)` produces a running pair for each item: `(0, 'p')`, `(1, 'y')`, `(2, 't')`, and so on. Writing `for index, letter in enumerate(word):` **unpacks** each pair directly into two variables in one step, rather than forcing you to pull the position and the value apart manually.

By default, the count starts at 0, matching how Python indexes things everywhere else. If you'd rather it start somewhere else — 1 is common when the output is meant for a human reader, since most people don't naturally count from zero — `enumerate()` accepts a `start` argument:

```python
word = "python"

for position, letter in enumerate(word, start=1):
    print(f"Letter {position}: {letter}")
```

`enumerate()` works on any iterable, not just strings — and it becomes especially natural once lists are introduced in a later guideline, since numbering the items of a list (first item, second item, and so on) is one of the most common things you'll ever want to do with one.

---

## 3. Zip Method (Use in a For Loop, a Great Alternative to Enumerate)

Where `enumerate()` pairs each item in *one* sequence with its own position, `zip()` pairs up items from **two or more separate sequences**, matching them by position — the first item of each sequence together, then the second item of each together, and so on.

```python
names = "abc"
scores = "xyz"

for name, score in zip(names, scores):
    print(name, score)
```

This prints `a x`, `b y`, `c z` — `zip()` walks through both sequences in lockstep, producing a matched pair on each step. It works with any number of iterables at once, not just two.

A detail worth knowing: **`zip()` stops as soon as the shortest sequence runs out.** If one sequence has 3 items and another has 5, the loop only runs 3 times — the extra 2 items in the longer sequence are simply never reached, with no error raised.

```python
short = "ab"
long = "wxyz"

for a, b in zip(short, long):
    print(a, b)
```

This prints only `a w` and `b x` — the `y` and `z` from `long` are silently ignored, because `short` ran out first.

The reason `zip()` is described as an alternative to `enumerate()` is that pairing a sequence with its own position is really just a special case of pairing two sequences together — one of them happens to be `range()`:

```python
word = "python"

for index, letter in zip(range(len(word)), word):
    print(index, letter)
```

This produces exactly the same output as `enumerate(word)` did earlier. In practice, `enumerate()` is the more direct and more commonly used tool when all you need is a sequence paired with its own position — but recognizing that `zip()` can do the same job, and that it's the natural tool once you have two *genuinely separate* sequences you want to walk through together, is worth having clearly in mind.

---

## 4. Loops with Conditional Statements (VERY IMPORTANT)

Every example so far in this guideline that did anything interesting — filtering odd numbers, searching for a letter, stopping early — did it by putting an `if` statement *inside* a loop. This combination is worth calling out directly and deliberately, because it is, more than any single other pattern, what real programs are actually built out of.

A loop by itself only repeats. A conditional by itself only decides once. Put an `if`/`elif`/`else` inside a loop's body, and you get something categorically more powerful: a decision that gets re-made, potentially differently, on every single iteration, based on whatever that iteration's specific value happens to be.

```python
for number in range(1, 21):
    if number % 3 == 0 and number % 5 == 0:
        print(f"{number}: divisible by both 3 and 5")
    elif number % 3 == 0:
        print(f"{number}: divisible by 3")
    elif number % 5 == 0:
        print(f"{number}: divisible by 5")
    else:
        print(number)
```

Notice what's actually happening here: the loop provides the repetition (twenty iterations, one per number from 1 to 20), and the conditional, re-evaluated fresh on every single one of those iterations, decides what to actually do with *that specific number*. Neither piece could produce this result alone — the loop doesn't know anything about divisibility, and the conditional, on its own, can only ever look at one number, one time. Together, they can classify an entire range.

Nearly every example in the rest of this guideline follows this exact shape: a loop provides the repetition, and a conditional inside it makes a decision that depends on the current state of that particular pass. Watching for this pattern — loop for repetition, conditional for decision, working together — is the most useful single habit this guideline can leave you with.

---

## 5. Worked Examples

### 5.1 Sum of Numbers (For Loop)

```python
n = int(input("Sum the numbers from 1 to: "))
total = 0

for number in range(1, n + 1):
    total += number

print(f"The sum of 1 to {n} is {total}.")
```

`total` starts at 0 and accumulates one number per iteration — this pattern, a variable initialized before the loop and updated inside it, is called an **accumulator**, and it's the backbone of almost every loop that produces a running total, count, or combined result.

### 5.2 Prime Numbers

A number is prime if it has no divisors other than 1 and itself. The check below tests every number from 2 up to (but not including) the candidate number, using a Boolean flag — a variable that starts `True` and only gets set to `False` if a divisor is actually found:

```python
number = int(input("Enter a number: "))
is_prime = True

if number < 2:
    is_prime = False
else:
    for divisor in range(2, number):
        if number % divisor == 0:
            is_prime = False
            break

if is_prime:
    print(f"{number} is prime.")
else:
    print(f"{number} is not prime.")
```

The `break` here is doing meaningful work: the moment a single divisor is found, there's no reason to keep checking the rest — the number has already been proven not prime, so the loop exits immediately rather than wasting iterations confirming what's already known.

### 5.3 Greatest Common Divisor (GCD)

The GCD of two numbers is the largest number that divides both of them evenly. One direct approach: start from the smaller of the two numbers and count downward, stopping at the first value that divides both.

```python
a = int(input("Enter the first number: "))
b = int(input("Enter the second number: "))

smaller = min(a, b)

for i in range(smaller, 0, -1):
    if a % i == 0 and b % i == 0:
        print(f"The GCD of {a} and {b} is {i}.")
        break
```

`range(smaller, 0, -1)` counts *downward* from `smaller` to 1 (recall from Part 1 that `range()`'s stop value is never included, so `0` is correctly excluded). Because the count starts at the largest possible candidate and works down, the very first number that divides both `a` and `b` evenly is guaranteed to be the greatest one — which is exactly why `break` can fire the moment a match is found.

### 5.4 Enter Numbers, Find Their Sum (Stop at 0)

This example introduces a genuinely useful pattern: `while True`, an intentionally infinite loop, combined with `break` as the *only* way out. This is different from the accidental infinite loops warned about in Part 1 — here, the loop never checks a condition of its own at all; `break`, triggered by a condition inside the body, is doing that job entirely on purpose.

```python
total = 0

while True:
    number = int(input("Enter a number (0 to stop): "))
    if number == 0:
        break
    total += number

print(f"The total is {total}.")
```

This pattern — `while True` with a `break` guarded by an `if` — is extremely common specifically because it handles a case a normal `while` condition can't handle cleanly: you need to *read* a value (the user's input) before you can possibly know whether the loop should stop, so there's no condition you could check up front, before the first iteration even begins.

### 5.5 Display Tables

**Multiplication table**, for a number the user provides:

```python
number = int(input("Show the multiplication table for: "))

for i in range(1, 11):
    print(f"{number} x {i} = {number * i}")
```

**Divisibility table**, checking every number from 1 to 20 against 3 and 5 — a direct application of the "loops with conditionals" section above:

```python
for number in range(1, 21):
    if number % 3 == 0 and number % 5 == 0:
        print(f"{number}: divisible by 3 and 5")
    elif number % 3 == 0:
        print(f"{number}: divisible by 3")
    elif number % 5 == 0:
        print(f"{number}: divisible by 5")
    else:
        print(f"{number}: divisible by neither")
```

**Celsius to Fahrenheit table**, from 0°C to 100°C in steps of 10, using the conversion `F = C × 9/5 + 32` and the `.1f` format specifier from Guideline 5 to keep the output tidy:

```python
for celsius in range(0, 101, 10):
    fahrenheit = celsius * 9 / 5 + 32
    print(f"{celsius:>3}°C = {fahrenheit:>6.1f}°F")
```

**Kilograms to pounds table**, using the conversion `1 kg ≈ 2.20462 lb`:

```python
for kg in range(0, 101, 10):
    pounds = kg * 2.20462
    print(f"{kg:>3} kg = {pounds:>7.2f} lb")
```

All four tables share the exact same skeleton: a `for` loop stepping through a range of values, and one calculation or conditional check applied fresh on every iteration — the same loop-plus-decision shape from Section 4, just with the "decision" sometimes being a calculation rather than a branch.

### 5.6 Username and Password (Nested Loop with Lockout)

This example nests a loop inside a conditional's branch, and gives that loop its own internal counter and exit conditions — a genuinely realistic (simplified) login system.

```python
correct_username = "admin"
correct_password = "python123"

username = input("Enter username: ")

if username == correct_username:
    attempts = 0
    logged_in = False

    while attempts < 3:
        password = input("Enter password: ")
        if password == correct_password:
            print("Login successful.")
            logged_in = True
            break
        else:
            attempts += 1
            remaining = 3 - attempts
            print(f"Incorrect password. {remaining} attempt(s) remaining.")

    if not logged_in:
        print("Account locked due to too many failed attempts.")
else:
    print("Username not found.")
```

Walk through the structure: the outer `if` checks the username exactly once, and only proceeds into the whole password-checking apparatus if it's correct — this mirrors the nested-conditional idea from Guideline 7, where a later question only makes sense once an earlier one has already been answered a specific way. Inside, the `while attempts < 3` loop gives the user up to three tries; a correct password triggers `break` and sets `logged_in = True` immediately, while an incorrect one increments `attempts` and lets the loop continue naturally back to its condition check. If all three attempts are used up without ever breaking out, the loop ends on its own (because `attempts < 3` finally becomes false), `logged_in` is still `False`, and the lockout message prints.

### 5.7 Draw Patterns (Square, Triangle, Pyramid)

**Square**, using two independent `range()` loops for rows and columns:

```python
size = 4

for row in range(size):
    for column in range(size):
        print("*", end="")
    print()
```

**Triangle**, where the inner loop's range grows with each pass of the outer loop:

```python
rows = 5

for i in range(1, rows + 1):
    for j in range(i):
        print("*", end="")
    print()
```

**Pyramid**, which needs one extra tool: multiplying a string by an integer repeats that string that many times — `"*" * 3` produces `"***"`, and `" " * 2` produces two spaces. Combining a shrinking block of leading spaces with a growing block of stars, row by row, centers the triangle into a pyramid:

```python
rows = 5

for i in range(1, rows + 1):
    spaces = " " * (rows - i)
    stars = "*" * (2 * i - 1)
    print(spaces + stars)
```

On the first row (`i = 1`), there are `rows - 1` leading spaces and a single star. On the last row (`i = rows`), there are zero leading spaces and `2 × rows - 1` stars — wide enough to sit directly beneath every row above it, forming the pyramid's base. Every one of these three patterns is built from the same underlying idea introduced in Part 1: the outer loop controls *which row* is being drawn, and the inner loop (or, in the pyramid's case, string repetition standing in for one) controls *what that row actually contains*.
