# Guideline 11: Python Lists (Part 3)

Parts 1 and 2 covered how to build a list, reach into it with indexing and slicing, and reshape it with methods like `.append()` and `.sort()`. Every one of those techniques, at some point, involves writing out a loop by hand whenever you need to build a *new* list out of an *existing* one. Part 3 introduces a more compact way to do exactly that: **list comprehension** — a single-line construct that replaces the most common loop pattern in this entire book: "make a new list by doing something to every item of another iterable."

---

## 11.12 List Comprehension: Replacing the Traditional Loop

The general form of a list comprehension is:

```python
[expression for variable in iterable]
```

Read left to right, this says: *for each `variable` in `iterable`, compute `expression`, and collect all of those results into a new list.* It is easiest to see by comparing it directly against the traditional loop it replaces.

**Traditional loop:**

```python
numbers = []
for i in range(1, 12):
    numbers.append(i)
print(numbers)
```

**List comprehension — same result, one line:**

```python
numbers = [i for i in range(1, 12)]
print(numbers)
# [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
```

Both versions build an empty list, walk through `range(1, 12)`, and add each value in — but the comprehension does it without an explicit `.append()` call or a separate line to initialize `numbers = []` first. The whole "loop that builds a list" pattern is folded into a single expression.

It is worth being precise about what each part of `[i for i in range(1, 12)]` means:

- `i` (the first one, right after `[`) is the **expression** — what gets added to the new list each time.
- `for i in range(1, 12)` is the **loop** — it walks through the iterable exactly the way a normal `for` loop does (Guideline 8).

Because the expression and the loop variable are both just `i` here, nothing is transformed — the comprehension simply reproduces the values from `range(1, 12)` as-is. The real value of a comprehension appears once the expression *does* something to each value, which is the next example.

---

## 11.13 Creating a List of Squares

Changing only the expression — not the loop — transforms every value on its way into the new list. Squaring each number from 1 to 10 looks like this:

```python
squares = [i ** 2 for i in range(1, 11)]
print(squares)
# [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

The loop, `for i in range(1, 11)`, still walks through 1 through 10, unchanged. But instead of collecting `i` itself, the comprehension collects `i ** 2` — the expression on the left of `for` is evaluated fresh for every value of `i`. This is the core idea behind every comprehension in this section: **the loop decides what to iterate over; the expression decides what to store.**

Compare with the traditional loop equivalent, to see exactly what has been compressed into one line:

```python
squares = []
for i in range(1, 11):
    squares.append(i ** 2)
```

Four lines become one, and — just as importantly — the intent ("build a list of squares") is visible immediately, rather than being assembled mentally from an initialization line, a loop line, and an append line.

---

## 11.14 List Comprehension with an `if`/`else` Statement

Comprehensions support conditions in two distinct ways, and it is important not to confuse them: one **filters** which items make it into the new list at all; the other **chooses between two expressions** for every item, keeping all of them.

### Filtering with a Trailing `if`

Adding `if condition` to the **end** of a comprehension keeps only the items that satisfy it — anything that fails the condition is skipped entirely.

```python
evens = [x for x in range(1, 21) if x % 2 == 0]
print(evens)
# [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
```

Here, `range(1, 21)` produces every number from 1 to 20, but only the ones where `x % 2 == 0` (Guideline 3's modulo operator, testing evenness) are actually added to `evens`. The odd numbers are checked and discarded, not stored.

### Choosing Between Two Expressions with `if`/`else`

Placing `if`/`else` **before** the `for`, as part of the expression itself, does something different: it does not filter anything out — every item from the iterable is still included, but the *value* stored for each one depends on the condition.

```python
labels = ["even" if x % 2 == 0 else "odd" for x in range(1, 11)]
print(labels)
# ['odd', 'even', 'odd', 'even', 'odd', 'even', 'odd', 'even', 'odd', 'even']
```

Every number from 1 to 10 produces exactly one label — nothing is dropped, unlike the `evens` example. The distinction matters: **a trailing `if` decides *whether* an item appears; a leading `if`/`else` decides *what value* appears for an item that is guaranteed to appear.** Mixing up the two positions is a common source of subtly wrong output, so it is worth testing both forms side by side until the difference is second nature.

The two forms can also be combined — filtering *and* transforming in the same comprehension:

```python
result = ["even" if x % 2 == 0 else "odd" for x in range(1, 21) if x % 5 == 0]
print(result)
# ['odd', 'even', 'odd', 'even']
```

This keeps only multiples of 5 from 1 to 20 (`5, 10, 15, 20`), and labels each of those as even or odd.

---

## 11.15 Creating Lists from Another List

So far, every iterable has been a `range()`. A comprehension can iterate over **any** existing list, which makes it a direct way to transform one list into a new, related one — without modifying the original.

```python
names = ["alice", "bob", "cody"]
upper_names = [name.upper() for name in names]
print(upper_names)
# ['ALICE', 'BOB', 'CODY']
print(names)
# ['alice', 'bob', 'cody']  <- unchanged
```

`upper_names` is a brand-new list; `names` is untouched — the comprehension reads from an existing list but always produces a separate one, the same way `sorted()` (Guideline 6) does not modify its input. This makes comprehensions a safe way to derive a new view of data whenever the original still needs to be preserved elsewhere in the program.

Filtering works here exactly as it did with `range()`:

```python
scores = [55, 90, 62, 78, 40, 85]
passing = [s for s in scores if s >= 60]
print(passing)
# [90, 62, 78, 85]
```

And transformation and filtering can combine on a real list just as they did in Section 11.14:

```python
scores = [55, 90, 62, 78, 40, 85]
results = ["Pass" if s >= 60 else "Fail" for s in scores]
print(results)
# ['Fail', 'Pass', 'Pass', 'Pass', 'Fail', 'Pass']
```

---

## 11.16 Nested List Comprehension

A comprehension can contain **more than one** `for` clause, which behaves like a nested loop (Guideline 8) collapsed into one line. This is most useful for generating combinations — pairs, coordinates, or grids — from two iterables at once.

```python
pairs = [(x, y) for x in range(1, 4) for y in range(1, 3)]
print(pairs)
# [(1, 1), (1, 2), (2, 1), (2, 2), (3, 1), (3, 2)]
```

Reading the `for` clauses **left to right** tells you which loop is "outer" and which is "inner" — exactly as if they were written as nested loops:

```python
pairs = []
for x in range(1, 4):        # outer loop
    for y in range(1, 3):    # inner loop
        pairs.append((x, y))
```

The first `for` in the comprehension (`for x in range(1, 4)`) corresponds to the **outer** loop, and the second (`for y in range(1, 3)`) corresponds to the **inner** one — which is why, in the output, `x` stays at `1` while `y` cycles through both of its values before `x` advances to `2`.

Conditions can be added to nested comprehensions the same way as before:

```python
pairs = [(x, y) for x in range(1, 4) for y in range(1, 4) if x != y]
print(pairs)
# [(1, 2), (1, 3), (2, 1), (2, 3), (3, 1), (3, 2)]
```

This produces every pair from 1 to 3 **except** where `x` and `y` are the same value — a pattern you will recognize later if you build anything that compares distinct items against each other, such as checking every player in a game against every *other* player.

**A note on readability:** a comprehension with one `for` clause is almost always easier to read than the loop it replaces. A comprehension with two `for` clauses, like the examples above, is usually still fine. Beyond that — three or more nested loops, or heavy conditional logic packed into one line — a comprehension often becomes *harder* to read than a plain, explicit loop, even though it is technically shorter. Compact code is not automatically better code. When a comprehension starts requiring a second read just to figure out what it does, that is the signal to write it out as a traditional loop instead.

---

Across all three parts, Guideline 11 has covered a list's full working vocabulary: creating one, reading and updating it precisely with indexing and slicing, reshaping it with built-in methods, and generating new lists compactly with comprehensions. This is the foundation on which the next few data-structure guidelines — tuples, dictionaries, and sets — will build, each one adapting these same core ideas (ordering, mutability, iteration) to a different kind of collection.

*End of Guideline 11, Part 3.*
