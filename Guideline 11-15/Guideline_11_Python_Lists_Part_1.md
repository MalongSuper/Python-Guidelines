# Guideline 11: Python Lists (Part 1)

Every data type covered so far — `str`, `int`, `float`, `bool` — holds exactly one value at a time. A single variable can store one name, one age, one price. But real programs rarely deal with one of anything. A gradebook holds dozens of scores. A to-do app holds an unknown number of tasks. A game holds a changing roster of players. For all of these, a single variable is not enough — you need a way to hold *many* values under *one* name, and to work with them as a collection.

This is what a **list** does, and it is one of the most important tools in this entire book. Lists appear constantly from this point forward: in loops, in functions, in data processing, and eventually in almost every program you write. Because of how central it is, this guideline is split into three parts. Part 1 builds the foundation: what a list actually is, the properties that define its behavior, and the two techniques — indexing and slicing — that let you reach into a list and pull out or change exactly what you need.

---

## 11.1 What is a List? Use Square Brackets

A **list** is an ordered collection of values, written with **square brackets** `[ ]`, with each value (called an **element** or **item**) separated by a comma.

```python
fruits = ["apple", "banana", "cherry"]
scores = [88, 92, 75, 100]
mixed = ["Alice", 25, True, 3.14]
empty_list = []
```

A few things to notice right away:

- The values inside the brackets can be **any data type** — strings, integers, floats, booleans — and a single list is allowed to **mix types**, as shown in `mixed` above. Python does not require every element to be the same type, unlike some other languages.
- A list can be **empty** (`[]`), holding zero elements. This is common when you plan to fill the list later, usually inside a loop.
- A list is itself a value, and it can be stored in a variable just like a string or a number. This means a list can be printed, passed into a function, or stored inside *another* list — a **list of lists**, which will come up again in a later guideline.

```python
print(fruits)
# ['apple', 'banana', 'cherry']

print(type(fruits))
# <class 'list'>
```

The `type()` function confirms it: `fruits` is not a string or a collection of separate variables — it is one object, of type `list`, that happens to contain three strings.

**Why not just use separate variables?** Compare these two approaches for storing five test scores:

```python
# Without a list — unmanageable
score1 = 88
score2 = 92
score3 = 75
score4 = 100
score5 = 60

# With a list — one name, five values
scores = [88, 92, 75, 100, 60]
```

The second version scales. If there are 500 students instead of 5, you cannot reasonably create `score1` through `score500`. A list holds all 500 under a single name, and — as you will see in Guideline 12 (Loops) — a list can be processed automatically, one element at a time, without writing 500 lines of code.

---

## 11.2 Properties of a List

Two properties define how a list behaves, and both are worth understanding precisely, because they distinguish a list from data types you have already learned.

### Mutable

A list is **mutable**, meaning its contents can be changed *after* it has been created — elements can be modified, added, or removed without creating a brand-new list.

```python
colors = ["red", "green", "blue"]
colors[0] = "yellow"
print(colors)
# ['yellow', 'green', 'blue']
```

This is a meaningful contrast with strings, which are **immutable** (Guideline 4). A string cannot have one of its characters changed in place:

```python
word = "cat"
word[0] = "b"
# TypeError: 'str' object does not support item assignment
```

To "change" a string, you must build an entirely new string. A list has no such restriction — the same list object in memory is updated directly. This matters more than it might seem: it means a list can grow, shrink, and be rearranged over the life of a program, which is exactly the kind of flexibility a collection of many values usually needs.

### Preserves Order

A list **preserves order** — the position each element was placed in is remembered, and that position does not change unless you change it yourself. The first element you wrote stays first; the second stays second, and so on, for as long as the list exists (or until you explicitly rearrange it).

```python
letters = ["a", "b", "c"]
print(letters)
# ['a', 'b', 'c']
```

`letters` will never silently reorder itself to `['c', 'a', 'b']` on its own. Order is part of the list's identity, and it is what makes the next two topics — indexing and slicing — possible in the first place. If a list's elements had no fixed position, there would be no meaningful way to ask for "the third item" or "everything after the second item." Because order *is* preserved, both of those questions have exact, reliable answers.

*(Note: not all Python collections behave this way. Guideline 13 introduces sets and dictionaries, and you will see that sets do not preserve order the way lists do. Keep that contrast in mind — it is precisely why lists are the right tool when position matters.)*

---

## 11.3 Indexing

**Indexing** is how you access a single element of a list by its position. Every element has a numbered position called an **index**, and Python lists are **zero-indexed** — counting starts at `0`, not `1`.

```python
fruits = ["apple", "banana", "cherry", "date"]
#            0         1         2        3
```

To retrieve an element, write the list's name followed by the index in square brackets:

```python
print(fruits[0])   # apple
print(fruits[2])   # cherry
```

This is the single most common mistake for beginners: `fruits[1]` does **not** return the first item — it returns the *second* item, `"banana"`, because the first item lives at index `0`.

### Negative Indexing

Python also supports **negative indices**, which count from the *end* of the list backward, starting at `-1` for the last element.

```python
fruits = ["apple", "banana", "cherry", "date"]
#            0         1         2        3
#           -4        -3        -2       -1

print(fruits[-1])   # date
print(fruits[-2])   # cherry
```

Negative indexing is genuinely useful, not just a shortcut. Getting "the last item" of a list without knowing how many items it has (`fruits[-1]`) is far more reliable than trying to calculate the final positive index yourself.

### Index Errors

Every list has a finite length, and asking for an index that does not exist raises an error:

```python
fruits = ["apple", "banana", "cherry"]
print(fruits[5])
# IndexError: list index out of range
```

The valid positive indices for a list of length `n` run from `0` to `n - 1`. Trying to access index `n` or beyond — or an out-of-range negative index — will always raise `IndexError`. This is one of the most common runtime errors you will encounter once loops enter the picture, so it is worth remembering now: **the last valid index is always one less than the list's length.**

### Indexing Also Works for Assignment

Because lists are mutable, an index is not just for *reading* a value — it can also be used to *replace* one:

```python
fruits = ["apple", "banana", "cherry"]
fruits[1] = "blueberry"
print(fruits)
# ['apple', 'blueberry', 'cherry']
```

This single line — index on the left of `=`, new value on the right — is the most direct way to update one specific element of a list, and it will reappear in Section 11.5.

---

## 11.4 Slicing

Indexing retrieves **one** element. **Slicing** retrieves a **range** of elements at once, producing a new list. The syntax adds a colon inside the brackets:

```python
list_name[start:stop]
```

- `start` is the index where the slice **begins** (inclusive — this element *is* included).
- `stop` is the index where the slice **ends** (exclusive — this element is **not** included).

```python
numbers = [10, 20, 30, 40, 50, 60]
#            0   1   2   3   4   5

print(numbers[1:4])
# [20, 30, 40]
```

Notice that index `4` (`50`) is *not* in the result — the stop index marks where the slice stops taking elements, without including that final position. This "stop is exclusive" rule is consistent throughout Python (it will show up again with `range()` and string slicing), so it is worth memorizing now rather than re-deriving it every time.

### Omitting Start or Stop

Either side of the colon can be left out, and Python fills in a sensible default:

```python
numbers = [10, 20, 30, 40, 50, 60]

print(numbers[:3])    # [10, 20, 30]   -> start defaults to 0
print(numbers[3:])    # [40, 50, 60]   -> stop defaults to the end of the list
print(numbers[:])     # [10, 20, 30, 40, 50, 60]  -> a full copy of the list
```

`numbers[:]` — omitting both — is a common and deliberate idiom: it produces a **new list** containing the same elements, which is useful when you want a copy to modify independently rather than a second name pointing at the same original list.

### Adding a Step

A slice can take a third value: `list_name[start:stop:step]`, where `step` controls how many positions to advance between each included element.

```python
numbers = [10, 20, 30, 40, 50, 60]

print(numbers[0:6:2])   # [10, 30, 50]   -> every 2nd element
print(numbers[::3])     # [10, 40]       -> every 3rd element, full range
```

A negative step reverses the direction entirely, and combining it with omitted start/stop produces one of the most useful one-liners in Python:

```python
print(numbers[::-1])
# [60, 50, 40, 30, 20, 10]
```

`list_name[::-1]` reverses a list without a loop, a function, or a single line of extra logic. It works because a step of `-1` walks backward through the whole list by default.

### Negative Indices in Slices

Negative indices work inside slices exactly as they do for regular indexing:

```python
letters = ["a", "b", "c", "d", "e"]

print(letters[-3:])     # ['c', 'd', 'e']  -> last three elements
print(letters[:-2])     # ['a', 'b', 'c']  -> everything except the last two
```

### Slicing Never Raises an IndexError

One notable difference from plain indexing: a slice with an out-of-range boundary does **not** crash. It simply returns as much as it can.

```python
letters = ["a", "b", "c"]
print(letters[0:10])
# ['a', 'b', 'c']
```

Where `letters[10]` would raise `IndexError`, `letters[0:10]` quietly stops at the end of the list. This forgiving behavior is intentional — slicing is meant to describe a *range*, and a range that extends past the list's end is simply capped, not treated as invalid.

---

## 11.5 Creating and Updating Lists: Applying Indexing and Slicing Techniques

With indexing and slicing established, this section brings them together with the practical task of building and modifying lists.

### Creating a List

There are two common ways to create a list:

```python
# 1. A list literal — values written directly inside square brackets
grades = [85, 90, 78]

# 2. The list() function — often used to convert another type into a list
letters = list("abc")
print(letters)
# ['a', 'b', 'c']
```

An **empty list**, created with `[]`, is just as valid and is frequently the starting point before values are added later (a pattern that becomes essential once loops are introduced in the next guideline):

```python
results = []
```

### Updating a Single Element with Indexing

As shown in Section 11.3, an index can appear on the left-hand side of `=` to replace one element in place:

```python
inventory = ["sword", "shield", "potion"]
inventory[2] = "elixir"
print(inventory)
# ['sword', 'shield', 'elixir']
```

This works with negative indices too:

```python
inventory[-1] = "healing potion"
print(inventory)
# ['sword', 'shield', 'healing potion']
```

### Updating Multiple Elements with Slicing

A slice on the left-hand side of `=` replaces an entire *range* of elements at once — and the replacement does not need to be the same length as the range it replaces.

```python
numbers = [1, 2, 3, 4, 5]
numbers[1:3] = [20, 30]
print(numbers)
# [1, 20, 30, 4, 5]
```

```python
numbers = [1, 2, 3, 4, 5]
numbers[1:3] = [100, 200, 300, 400]
print(numbers)
# [1, 100, 200, 300, 400, 4, 5]
```

In the second example, a two-element range (`[2, 3]`) is replaced by four values — and the list simply grows to make room. This is a direct consequence of lists being mutable: the list is not a fixed-size container being partially overwritten, it is a flexible sequence being reshaped.

Slice assignment can also **remove** elements outright, by assigning an empty list to a range:

```python
numbers = [1, 2, 3, 4, 5]
numbers[1:3] = []
print(numbers)
# [1, 4, 5]
```

### Combining Indexing and Slicing in Practice

A short worked example ties this section together — updating a class roster's scores after a retest:

```python
students = ["Amy", "Ben", "Cody", "Dina", "Evan"]
scores   = [55,     90,    62,     78,     40]

# Amy and Evan retook the test — update their scores using indexing
scores[0] = 81
scores[-1] = 70
print(scores)
# [81, 90, 62, 78, 70]

# The middle three students are moving to a new group — replace them using slicing
students[1:4] = ["Ben", "Priya"]
print(students)
# ['Amy', 'Ben', 'Priya', 'Evan']
```

Notice that `students` shrank from five names to four in that last step — Cody and Dina were removed and replaced by a single new name, Priya, all in one line of slice assignment.

---

Indexing and slicing are the two techniques you will use, constantly, for the rest of this book whenever a list is involved — everything from pulling a single record out of a dataset to reversing a sequence to trimming unwanted entries. Part 2 of this guideline builds on this foundation with the list's built-in **methods**: the operations like adding, removing, and searching for elements that go beyond what indexing and slicing alone can do.

*End of Guideline 11, Part 1.*
