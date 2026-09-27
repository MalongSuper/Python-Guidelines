# Guideline 11: Python Lists (Part 2)

Part 1 covered how to build a list and reach into it directly, element by element or range by range, using indexing and slicing. Those two tools work, but they are manual — you have to already know exactly which position you want to touch. Part 2 introduces the list's **built-in methods**: operations that come attached to every list and handle the common tasks — adding, removing, reordering, counting, and inspecting — without you having to work out the indices yourself.

A quick reminder of the distinction from Guideline 6: a **function** is called on its own (`len(my_list)`), while a **method** is called *on* an object, using dot notation (`my_list.append(...)`). Everything in this section except `len()`, `sum()`, `max()`, and `min()` is a method — you will see the pattern `list_name.method_name()` throughout.

---

## 11.6 Operations on Lists: Concatenation and Repetition

Before getting into methods, two operators you already know from earlier guidelines — `+` and `*` — work on lists too, and behave in a way that is worth seeing explicitly before anything else in this part.

### Concatenation with `+`

Using `+` between two lists **joins them together**, producing a new list that contains every element of the first list followed by every element of the second.

```python
list_a = [1, 2, 3]
list_b = [4, 5, 6]

combined = list_a + list_b
print(combined)
# [1, 2, 3, 4, 5, 6]
```

This is the same `+` used for numeric addition (Guideline 3) and string concatenation (Guideline 5) — its meaning simply adapts to the type of its operands. As with string concatenation, `+` does **not** modify either original list; it builds a brand-new one and leaves `list_a` and `list_b` exactly as they were.

```python
print(list_a)   # [1, 2, 3]  <- unchanged
print(list_b)   # [4, 5, 6]  <- unchanged
```

Note that `+` requires both sides to be lists — it cannot be used to add a single loose value onto the end of a list (that is exactly what `.append()`, coming up next, is for):

```python
numbers = [1, 2, 3]
numbers + 4
# TypeError: can only concatenate list (not "int") to list
```

To add one value with `+`, it has to be wrapped in a list of its own, single-element or otherwise:

```python
numbers = [1, 2, 3] + [4]
print(numbers)
# [1, 2, 3, 4]
```

### Repetition with `*`

Using `*` between a list and an integer **repeats** the list's contents that many times, again producing a new list.

```python
pattern = [0, 1] * 3
print(pattern)
# [0, 1, 0, 1, 0, 1]
```

This mirrors string repetition from Guideline 5 (`"ab" * 3` producing `"ababab"`) — the same idea, applied to a sequence of elements instead of a sequence of characters. It is a quick way to build a list pre-filled with a repeated value, which comes up often when a list needs to start at a known size before being filled in properly by a loop:

```python
scoreboard = [0] * 5
print(scoreboard)
# [0, 0, 0, 0, 0]
```

`[0] * 5` is a much shorter way to write "five zeroes" than typing them out by hand, and it scales the same way regardless of how large the number gets.

**A caution with nested lists:** repetition does not create independent copies of a *mutable* element — it repeats the same reference. This rarely matters for simple values like numbers or strings, but it matters a great deal for a list *of* lists:

```python
grid = [[0, 0]] * 3
grid[0][0] = 9
print(grid)
# [[9, 0], [9, 0], [9, 0]]
```

Changing just `grid[0][0]` appears to change every row, because all three inner lists are actually the *same* list, repeated three times over rather than copied three times. Guideline 11 Part 1's distinction between mutable and immutable elements is exactly what is at play here — this pitfall is worth remembering once nested lists become more common in your own code.

---

## 11.7 Adding Elements: `append()`, `extend()`, and `insert()`

### `.append()` — Add One Element to the End

`.append()` adds a single value to the **end** of a list, growing it by exactly one element.

```python
fruits = ["apple", "banana"]
fruits.append("cherry")
print(fruits)
# ['apple', 'banana', 'cherry']
```

This is the most common way to build a list gradually — starting from `[]` and appending one value at a time, usually inside a loop (Guideline 12). Note that `.append()` always adds **one** item, even if that item is itself a list:

```python
numbers = [1, 2, 3]
numbers.append([4, 5])
print(numbers)
# [1, 2, 3, [4, 5]]
```

The list `[4, 5]` was added as a single element, producing a list *inside* a list — not the four separate numbers you might have expected. That distinction is exactly why `.extend()` exists.

### `.extend()` — Add Multiple Elements Individually

`.extend()` takes another list (or any sequence of values) and adds **each of its elements** to the end of the original list, one at a time — rather than adding the whole thing as one nested item.

```python
numbers = [1, 2, 3]
numbers.extend([4, 5])
print(numbers)
# [1, 2, 3, 4, 5]
```

Compare this directly with the `.append()` example above: same input, `[4, 5]`, but a completely different result. **`.append()` adds the list as one item; `.extend()` unpacks the list and adds its contents.** This is one of the most common points of confusion for beginners, so it is worth testing both in practice until the difference feels automatic.

### `.insert()` — Add an Element at a Specific Position

`.append()` and `.extend()` only add to the end. `.insert()` adds a single element at a chosen **index**, shifting everything after it one position to the right.

```python
colors = ["red", "green", "blue"]
colors.insert(1, "yellow")
print(colors)
# ['red', 'yellow', 'green', 'blue']
```

`"yellow"` was inserted *at* index `1`, pushing `"green"` and `"blue"` back by one position each. `.insert()` takes two arguments — the index to insert at, and the value to insert — in that order: `list_name.insert(index, value)`.

---

## 11.8 Removing Elements: `.pop()` and `.remove()`

These two methods both remove an element, but they answer two different questions: "remove whatever is *at this position*" versus "remove *this specific value*, wherever it is."

### `.pop()` — Remove by Position (and Return It)

`.pop()` removes the element at a given index and **returns it**, so it can be stored or used immediately.

```python
fruits = ["apple", "banana", "cherry"]
removed = fruits.pop(1)
print(removed)   # banana
print(fruits)    # ['apple', 'cherry']
```

Called with **no argument**, `.pop()` removes and returns the **last** element — a common and convenient default:

```python
fruits = ["apple", "banana", "cherry"]
last_item = fruits.pop()
print(last_item)   # cherry
print(fruits)       # ['apple', 'banana']
```

This return-a-value behavior is what sets `.pop()` apart from every other removal method in this section: it does not just delete — it hands you back what it deleted.

### `.remove()` — Remove by Value

`.remove()` searches the list for the **first occurrence** of a given value and deletes that element — no index required, and nothing is returned.

```python
fruits = ["apple", "banana", "cherry", "banana"]
fruits.remove("banana")
print(fruits)
# ['apple', 'cherry', 'banana']
```

Notice that only the **first** `"banana"` was removed; the second one remains untouched. If the value does not exist in the list at all, `.remove()` raises an error:

```python
fruits.remove("mango")
# ValueError: list.remove(x): x not in list
```

**Choosing between them:** use `.pop()` when you know *where* the element is (or want the last one); use `.remove()` when you know *what* the element is but not necessarily where it sits.

---

## 11.9 Reversing: `.reverse()` — Same Result as `[::-1]`

`.reverse()` flips the order of a list's elements **in place** — modifying the original list directly rather than producing a new one.

```python
numbers = [1, 2, 3, 4, 5]
numbers.reverse()
print(numbers)
# [5, 4, 3, 2, 1]
```

Recall from Section 11.4 that `numbers[::-1]` accomplishes the same *visual* result. The two are not identical in mechanism, though, and the difference matters:

```python
numbers = [1, 2, 3, 4, 5]

reversed_copy = numbers[::-1]   # creates a brand-new reversed list
print(reversed_copy)             # [5, 4, 3, 2, 1]
print(numbers)                   # [1, 2, 3, 4, 5]  <- unchanged!

numbers.reverse()                # reverses numbers itself
print(numbers)                   # [5, 4, 3, 2, 1]  <- now changed
```

`list_name[::-1]` **returns a new, separate list**, leaving the original alone unless you reassign it (`numbers = numbers[::-1]`). `.reverse()` **changes the original list directly** and returns nothing at all (technically, it returns `None`). Which one you want depends entirely on whether you still need the original order somewhere else in your program.

---

## 11.10 `.sort()` and `.count()`

### `.sort()` — Arrange Elements in Order

`.sort()` rearranges a list's elements **in place**, in ascending order by default.

```python
numbers = [4, 1, 3, 5, 2]
numbers.sort()
print(numbers)
# [1, 2, 3, 4, 5]
```

Strings sort alphabetically the same way:

```python
names = ["Charlie", "Alice", "Bob"]
names.sort()
print(names)
# ['Alice', 'Bob', 'Charlie']
```

For descending order, pass the keyword argument `reverse=True`:

```python
numbers = [4, 1, 3, 5, 2]
numbers.sort(reverse=True)
print(numbers)
# [5, 4, 3, 2, 1]
```

Just like `.reverse()`, `.sort()` modifies the original list and returns `None`. This is the same in-place pattern from Guideline 6, where `sorted()` (the **function**) was contrasted with `.sort()` (the **method**): `sorted(numbers)` returns a new sorted list and leaves `numbers` untouched, while `numbers.sort()` sorts `numbers` itself, permanently.

### `.count()` — Count Occurrences of a Value

`.count()` returns how many times a specific value appears in the list.

```python
grades = ["A", "B", "A", "C", "A", "B"]
print(grades.count("A"))
# 3
```

If the value does not appear at all, `.count()` simply returns `0` — it does not raise an error the way `.remove()` does for a missing value.

```python
print(grades.count("F"))
# 0
```

---

## 11.11 `.clear()` — A Risky One

`.clear()` removes **every** element from a list, leaving it empty — permanently.

```python
fruits = ["apple", "banana", "cherry"]
fruits.clear()
print(fruits)
# []
```

This is, functionally, one of the simplest methods in this guideline. It is also one of the easiest to misuse, for a reason that has nothing to do with syntax: `.clear()` deletes the entire contents of a list **irreversibly**, with no confirmation and no built-in way to undo it. There is no error, no warning — the list is simply empty on the next line.

Two habits are worth building around this method specifically:

- **Never call `.clear()` on a list you have not deliberately decided to empty.** It is easy to call it by mistake — for instance, confusing it with a method that only *inspects* the list — and lose data that took real effort to build.
- **If a list needs to be preserved before clearing another one, copy it first.** Recall `list_name[:]` from Section 11.4: `backup = fruits[:]` creates an independent copy, so `fruits.clear()` afterward does not affect `backup`.

```python
fruits = ["apple", "banana", "cherry"]
backup = fruits[:]      # independent copy, made first
fruits.clear()
print(fruits)   # []
print(backup)   # ['apple', 'banana', 'cherry']  <- safe
```

The lesson here is not "avoid `.clear()`" — it is a completely legitimate tool when a list genuinely needs to be reset (for example, restarting a game's score list between rounds). The lesson is: because it is irreversible and silent, treat it with the same caution you would treat deleting a file.

---

## 11.12 Applying Built-in Functions: `len()`, `sum()`, `max()`, and `min()`

The four operations in this section are **functions**, not methods — called as `function(list_name)` rather than `list_name.function()` — but they are used on lists constantly enough that they belong in this guideline alongside the methods above.

### `len()` — The Length of a List

`len()` returns how many elements a list contains.

```python
fruits = ["apple", "banana", "cherry"]
print(len(fruits))
# 3
```

On its own this looks simple, but `len()` becomes one of the most-used functions in this entire book once loops are introduced, because it enables **traversal** — visiting every element of a list by index, without knowing in advance how long the list is:

```python
fruits = ["apple", "banana", "cherry"]

for i in range(len(fruits)):
    print(i, fruits[i])
# 0 apple
# 1 banana
# 2 cherry
```

`len(fruits)` tells `range()` exactly how far to count, which means this loop works correctly whether `fruits` has 3 elements or 3,000 — the code does not change. `len()` also has a second common role: bounds-checking, to avoid the `IndexError` from Section 11.3 before it happens.

```python
if len(fruits) > 5:
    print(fruits[5])
else:
    print("Not enough fruits in the list.")
```

### `sum()`, `max()`, and `min()` — Aggregating Numeric Lists

These three functions operate on a list of numbers and return one summary value:

```python
scores = [88, 92, 75, 100, 60]

print(sum(scores))   # 415  -> total of all elements
print(max(scores))   # 100  -> largest value
print(min(scores))   # 60   -> smallest value
```

A common and genuinely useful combination is computing an **average**, which Python has no single built-in function for — it is built from `sum()` and `len()` together:

```python
scores = [88, 92, 75, 100, 60]
average = sum(scores) / len(scores)
print(average)
# 83.0
```

`max()` and `min()` also work on lists of strings, comparing them alphabetically (technically, by their underlying character codes — recall `ord()` from Guideline 6):

```python
names = ["Charlie", "Alice", "Bob"]
print(max(names))   # Charlie
print(min(names))   # Alice
```

---

Between Part 1 and Part 2, you now have the complete toolkit for working with a single list: creating one, reaching into it precisely with indexing and slicing, and reshaping it with the built-in methods and functions covered here. Part 3 of this guideline moves from operating on one list at a time to more advanced list structures and patterns that build on everything established so far.

*End of Guideline 11, Part 2.*
