# Guideline 7: Conditional Statements
## Part 1

Every guideline up to this point has covered code that runs the exact same way every single time you execute it. Line one runs, then line two, then line three, in a straight, unbroken sequence. That kind of program is honest, but it isn't very smart. It cannot tell the difference between a positive number and a negative one. It cannot decide whether a password is correct. It cannot tell a five-year-old apart from a fifty-year-old, even if you hand it both of their ages directly.

Conditional statements are what change that. They let a program look at a value, ask a question about it, and choose a different path depending on the answer. This is the first real branching point in your code — the first place where "what happens next" is no longer fixed, but depends on the data in front of it. Because this idea is so central to almost everything that comes after it in this book, Guideline 7 is split into two parts. Part 1 covers the foundation: `if`, `else`, `elif`, combining conditions with `and` / `or`, and the difference between separate `if` statements and a proper nested structure. Part 2 will build on this foundation with more advanced patterns.

---

## 1. If-Else Statements

### The `if` statement

An `if` statement asks Python a yes-or-no question. If the answer is `True`, the block of code underneath it runs. If the answer is `False`, that block is skipped entirely, and Python moves on to whatever comes after it.

```python
age = 20

if age >= 18:
    print("You are eligible to vote.")
```

There are three parts to notice here, and all three are non-negotiable:

1. **The keyword `if`**, followed by a condition — an expression that evaluates to either `True` or `False`.
2. **A colon `:`** at the end of the condition line. Forgetting this is one of the most common beginner errors in Python.
3. **An indented block** underneath. Python does not use curly braces `{}` like many other languages to mark where a block of code begins and ends — it uses indentation itself. Every line that belongs to the `if` block must be indented by the same amount (four spaces is the standard convention). The moment a line is no longer indented, Python considers the block over.

This matters more in Python than in almost any other popular language. Indentation is not a style choice here — it is syntax. Get it wrong, and your program either crashes with an `IndentationError` or, worse, runs but does something you didn't intend.

### Comparison operators

The condition inside an `if` statement is almost always built using a comparison operator — a symbol that compares two values and produces a `True` or `False` result. These are the ones you will use constantly:

| Operator | Meaning | Example | Result |
|---|---|---|---|
| `>` | Greater than | `7 > 3` | `True` |
| `<` | Less than | `7 < 3` | `False` |
| `>=` | Greater than or equal to | `5 >= 5` | `True` |
| `<=` | Less than or equal to | `4 <= 3` | `False` |
| `==` | Equal to | `10 == 10` | `True` |
| `!=` | Not equal to | `10 != 5` | `True` |

**The one that trips up almost every beginner at least once is `==`.**

A single equals sign, `=`, is the **assignment operator**. It takes a value and stores it in a variable. A double equals sign, `==`, is the **comparison operator**. It asks whether two values are the same, and hands back `True` or `False` — it does not store anything.

```python
x = 5      # assignment: x now holds the value 5
x == 5     # comparison: asks "does x equal 5?" -> True
```

Confusing the two is not a minor stylistic slip — it changes what your code *does*. Writing `if x = 5:` is not valid Python at all and will raise a `SyntaxError`, because you cannot assign a value inside a condition. Python forces this error precisely so this exact mistake gets caught immediately rather than silently producing the wrong behavior, which is what tends to happen in languages that permit it.

### The `else` clause

An `if` statement on its own only handles one branch: what to do when the condition is true. Most real decisions have at least two sides. The `else` clause supplies the other one — the code that runs when the condition turns out to be `False`.

```python
age = 15

if age >= 18:
    print("You are eligible to vote.")
else:
    print("You are not eligible to vote yet.")
```

Exactly one of these two blocks will run — never both, and never neither. `else` takes no condition of its own; it simply means "in every other case."

```python
temperature = 15

if temperature >= 25:
    print("It's a hot day.")
else:
    print("It's not a hot day.")
```

Notice that `else` is only ever attached to the `if` (or `elif`) directly above it, at the same indentation level. It cannot stand on its own.

---

## 2. Elif Statements

`if` / `else` is enough when a decision has exactly two outcomes. But most real decisions have more than two. Consider grading a test score: it isn't just "pass" or "fail," it's an entire range of letter grades. This is what `elif` — short for "else if" — is for.

```python
score = 82

if score >= 90:
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

A few things about how this actually executes matter a great deal:

- Python checks the conditions **in order, from top to bottom**.
- The moment one of them evaluates to `True`, that block runs, and **every remaining `elif` and `else` in the chain is skipped** — even if a later condition would also have been true.
- If none of the `if`/`elif` conditions are true, the final `else` block runs, if one is present. `else` is always optional.
- There is no limit to how many `elif` blocks you can chain together, but a chain that grows very long is often a sign the logic could be restructured more clearly (this is a design concern we'll return to later in the book).

Trace through the example above with `score = 82`: Python checks `score >= 90` — false. It checks `score >= 80` — true, so `"Grade: B"` is printed, and Python does not bother checking `score >= 70` or anything after it, even though that condition is also technically true. Order matters, and it matters specifically because the first true condition wins.

A second example, classifying a number:

```python
number = -4

if number > 0:
    print("Positive")
elif number < 0:
    print("Negative")
else:
    print("Zero")
```

Here, exactly one of the three conditions will always be true for any real number, so this chain has effectively no gaps — every possible input lands somewhere.

---

## 3. The `and` / `or` Operators

Sometimes a decision doesn't depend on a single condition — it depends on two or more conditions at once. Guideline 3 introduced `and`, `or`, and `not` as logical operators in the abstract; here is where they earn their keep inside real `if` statements.

**`and`** requires *every* condition joined by it to be `True` for the whole expression to be `True`. If even one of them is `False`, the whole thing is `False`.

```python
age = 25
has_ticket = True

if age >= 18 and has_ticket:
    print("Entry allowed.")
else:
    print("Entry denied.")
```

Both `age >= 18` and `has_ticket` must be true here. Change either one to false, and the whole condition becomes false.

**`or`** requires *at least one* condition joined by it to be `True` for the whole expression to be `True`. It only fails if every single condition is false.

```python
day = "Saturday"

if day == "Saturday" or day == "Sunday":
    print("It's the weekend.")
else:
    print("It's a weekday.")
```

A particularly common and useful pattern combines a comparison operator on both sides of a value with `and`, to check whether something falls inside a range:

```python
age = 30

if age >= 18 and age <= 65:
    print("Working-age adult.")
```

Python actually allows a shorter, more readable form of exactly this pattern, called a **chained comparison**:

```python
if 18 <= age <= 65:
    print("Working-age adult.")
```

This line reads almost like plain English — "18 is less than or equal to age, which is less than or equal to 65" — and it behaves identically to the `and` version above. Both are correct; the chained form is simply the more idiomatic way to write a range check in Python.

One behavior worth knowing about, even if you won't rely on it heavily yet: Python evaluates `and` and `or` with **short-circuiting**. In an `and` expression, if the first condition is already `False`, Python does not bother checking the second one at all — the answer is already decided. In an `or` expression, if the first condition is already `True`, the second is never checked either. This isn't just an efficiency detail; it means the order in which you write your conditions can occasionally matter, particularly once conditions involve more complex operations later in the book.

---

## 4. Multiple If-Else Statements

There are two distinct ways to have more than one `if` in the neighborhood of each other, and they behave very differently. Confusing the two is one of the more consequential mistakes a beginner can make, because the code often still runs — it just runs the *wrong* logic.

### Non-nested: multiple independent `if` statements

If you write several separate `if` statements back to back, at the same indentation level, each one is evaluated **completely independently** of the others. Python checks every single one of them, regardless of what happened with the ones before it.

```python
number = 7

if number > 0:
    print("The number is positive.")
if number % 2 != 0:
    print("The number is odd.")
if number < 100:
    print("The number is under 100.")
```

With `number = 7`, all three conditions are true, so all three messages print. These are not connected to each other at all — they are three separate questions being asked of the same variable, one after another. This is the right tool when the conditions are genuinely unrelated to each other and more than one of them might legitimately need to fire.

Compare that to what happens if these were written as an `if` / `elif` / `elif` chain instead — only the *first* true one would run, and the other two would be silently skipped, which would produce a very different (and, here, wrong) result. The choice between separate `if` statements and an `elif` chain is really a choice about whether your conditions are independent questions or mutually exclusive alternatives.

### Nested: `if`-`elif`-`elif`-`else` inside another branch

A **nested** conditional is an entire `if`/`elif`/`else` structure placed *inside* one of the blocks of another one. This lets you ask a follow-up question, but only once a first-level question has already been answered a particular way.

```python
age = 25
has_id = True

if age >= 18:
    if has_id:
        print("Entry allowed.")
    else:
        print("Entry denied: ID required.")
else:
    print("Entry denied: must be 18 or older.")
```

Notice the indentation carefully — the inner `if`/`else` is indented one full level further than the outer `if`/`else`, marking it as belonging entirely inside the `age >= 18` branch. It is never even evaluated if `age >= 18` is false; there is no way to reach the `has_id` check at all unless the outer condition already passed.

Nesting can go deeper than one level, and can mix `elif` chains at each level:

```python
score = 74
attendance = 90

if score >= 60:
    if attendance >= 80:
        print("Pass with good standing.")
    elif attendance >= 50:
        print("Pass, but attendance is a concern.")
    else:
        print("Pass denied: attendance too low.")
else:
    if attendance >= 80:
        print("Fail, but eligible for a resit.")
    else:
        print("Fail: no resit eligibility.")
```

Here, the *meaning* of the attendance check genuinely depends on which side of the score check you're already on — that dependency is exactly what nesting is for. If the two questions had been independent of each other, separate `if` statements (or a single combined condition using `and`) would have been the more appropriate, and more readable, choice.

As a general guide: reach for nested conditionals when a later question only makes sense in the context of an earlier answer. Reach for separate `if` statements when the questions don't depend on each other at all. And reach for an `elif` chain when the conditions are mutually exclusive alternatives to the *same* underlying question. Part 2 of this guideline will build directly on these three shapes.
