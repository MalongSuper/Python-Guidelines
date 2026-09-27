# Guideline 16: Python Multi-Dimensional Lists

By now, you've worked with lists as a single row of values — a shopping list, a set of scores, a sequence of names. But real-world data is rarely that flat. A spreadsheet has rows *and* columns. A tic-tac-toe board has a grid. A student record has a name *and* an age *and* a grade, all bundled together. Python handles all of this the same way it handles everything else: by putting lists inside lists.

This guideline is about that idea — multi-dimensional lists — and the handful of related structures (tuples, sets, dictionaries) that show up once your data stops being a single flat line.

---

## 1. What is a Multi-Dimensional List?

A multi-dimensional list is a list whose elements are themselves lists (or list-like collections). Instead of a single row of values, you get a list of rows — and each of *those* rows can itself contain a list of rows, and so on. The "dimension" refers to how many levels of nesting you need to reach an individual value.

```python
flat = [1, 2, 3, 4]          # 1-dimensional — one index needed
grid = [[1, 2], [3, 4]]      # 2-dimensional — two indices needed
cube = [[[1], [2]], [[3], [4]]]  # 3-dimensional — three indices needed
```

Nothing new is happening mechanically — a multi-dimensional list is just a list where `list[i]` gives you *another list* instead of a single value. Everything you already know about indexing, slicing, and looping still applies; you just apply it more than once.

---

## 2. 1-Dimensional List

A quick recap, since it's the baseline everything else builds on. A 1-dimensional list is a flat sequence of values, accessed with a single index:

```python
temperatures = [21.5, 23.0, 19.8, 25.1]

print(temperatures[0])   # 21.5
print(temperatures[-1])  # 25.1
```

One index in, one value out. This is the shape you've been using since Guideline 11. Every "dimension" you add beyond this is just another index required to reach a single value.

---

## 3. 2-Dimensional List

A 2-dimensional list — often called a **matrix** or a **grid** — is a list of lists. Think of it as rows stacked on top of each other, where each row is itself a list.

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
```

To reach a single value, you need **two** indices: the first selects the row, the second selects the position within that row.

```python
print(matrix[0])       # [1, 2, 3]  -> the whole first row
print(matrix[0][0])    # 1          -> first element of first row
print(matrix[1][2])    # 6          -> third element of second row
print(matrix[-1][-1])  # 9          -> last element of last row
```

### Creating a 2D list dynamically

Writing out `[[1, 2, 3], [4, 5, 6], [7, 8, 9]]` by hand works for small, fixed data, but often you need to *build* a grid of a given size — for example, an empty 3×3 board. A natural-looking but broken way to do this is:

```python
board = [[0] * 3] * 3
```

This looks reasonable, but it's a trap. `[0] * 3` creates one list `[0, 0, 0]`, and `* 3` at the outer level doesn't copy that list three times — it repeats the *same reference* three times. Change one row, and all three "rows" change together:

```python
board[0][0] = "X"
print(board)
# [['X', 0, 0], ['X', 0, 0], ['X', 0, 0]]  -> every row changed!
```

The correct way is a nested list comprehension, which builds each inner list *independently*:

```python
board = [[0 for _ in range(3)] for _ in range(3)]
board[0][0] = "X"
print(board)
# [['X', 0, 0], [0, 0, 0], [0, 0, 0]]  -> only the first row changed
```

Rule of thumb: whenever you need a fresh 2D structure, build it with a nested comprehension, not with `*` repetition on the outer list.

---

## 4. 3-Dimensional and Beyond

A 3-dimensional list is a list of 2-dimensional lists — a "stack of grids." A practical way to picture this is a set of matrices, one per "layer": for example, three separate 2×2 boards stacked together, or the red/green/blue channels of an image, each represented as its own grid of pixel values.

```python
cube = [
    [[1, 2], [3, 4]],   # layer 0
    [[5, 6], [7, 8]],   # layer 1
    [[9, 10], [11, 12]] # layer 2
]

print(cube[0])         # [[1, 2], [3, 4]]        -> entire first layer
print(cube[0][1])      # [3, 4]                  -> second row of first layer
print(cube[0][1][0])   # 3                       -> single value
print(cube[2][1][1])   # 12
```

Each additional dimension is just another list wrapped around the structure, and each additional index just walks one level deeper. There's no hard limit in Python — you could nest a 4D or 5D list the same way — but in practice, pure Python lists become awkward and hard to reason about much past 3 dimensions. This is exactly the problem that libraries like **NumPy** were built to solve: they represent multi-dimensional data as a single specialized array object instead of nested Python lists, which is both faster and easier to work with at higher dimensions. That's a topic for later in this book — for now, know that nested lists are the general-purpose tool, and specialized array libraries exist for when the dimensions and the data get large.

---

## 5. Other Examples of Multi Lists

"Multi-dimensional" doesn't only mean lists nested inside lists. Once you're comfortable with the idea of a collection whose elements are themselves structured, several other combinations become useful for representing real data.

### List of tuples

A common way to represent a set of records, where each record has a fixed, ordered set of fields:

```python
students = [("Amara", 90), ("Ben", 85), ("Cho", 78)]

for name, score in students:
    print(f"{name}: {score}")
```

Because tuples are immutable, this pattern is often used when the *shape* of each record (name, then score) should not accidentally change, even if the list itself grows or shrinks.

### Set of tuples — relations

A set of tuples is commonly used to represent a **relation** — an unordered collection of unique pairings, with no duplicates allowed. This shows up naturally when representing connections, such as edges in a graph or pairings between two sets:

```python
friendships = {("Amara", "Ben"), ("Ben", "Cho"), ("Amara", "Cho")}

print(("Amara", "Ben") in friendships)  # True
```

Because it's a set, adding `("Amara", "Ben")` a second time changes nothing — relations like this are exactly the same concept as a mathematical relation: a set of ordered pairs.

### List of dictionaries

Perhaps the most common "multi-dimensional" structure in real-world code — each element is a full record with named fields, rather than a positional tuple:

```python
students = [
    {"name": "Amara", "score": 90},
    {"name": "Ben", "score": 85},
    {"name": "Cho", "score": 78}
]

for student in students:
    print(f"{student['name']} scored {student['score']}")
```

This is the shape you'll see constantly once you start working with real data — it's how JSON data from web APIs is typically structured once loaded into Python.

### Dict of dict

A dictionary whose values are themselves dictionaries — useful when each key needs its own bundle of named fields, rather than a single value:

```python
students = {
    "Amara": {"age": 20, "grade": "A"},
    "Ben":   {"age": 21, "grade": "B"},
    "Cho":   {"age": 19, "grade": "A"}
}

print(students["Ben"]["grade"])  # B
```

Compare this with the list of dictionaries above: a list of dicts is best when you mainly care about *iterating over everyone*; a dict of dicts is best when you mainly care about *looking someone up by name*.

---

## 6. Traversing in Multi-List (Nested Loop)

Since a 2D list is a list of lists, visiting every value requires a loop *inside* a loop: the outer loop walks through the rows, and the inner loop walks through the values within the current row.

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

for row in matrix:
    for value in row:
        print(value, end=" ")
    print()  # newline after each row

# Output:
# 1 2 3
# 4 5 6
# 7 8 9
```

The same principle extends to 3D: a nested loop inside a nested loop.

```python
cube = [
    [[1, 2], [3, 4]],
    [[5, 6], [7, 8]]
]

for layer in cube:
    for row in layer:
        for value in row:
            print(value, end=" ")
    print()

# Output:
# 1 2 3 4
# 5 6 7 8
```

Each dimension you add to the data structure adds exactly one more level of `for` loop needed to reach the individual values.

---

## 7. Indexing and Slicing in Multi-List — Retrieving Rows, Columns, and Calculating Sums with Enumerate

### Retrieving a row

A single index on a 2D list retrieves an entire row, since `matrix[i]` is itself a list:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

print(matrix[1])  # [4, 5, 6]
```

### Retrieving a column

There's no built-in shortcut for a column — a column is one value taken from *every* row, so you build it with a list comprehension:

```python
column_1 = [row[1] for row in matrix]
print(column_1)  # [2, 5, 8]
```

### Slicing rows

Slicing the outer list works exactly like slicing any list — it returns a subset of the rows:

```python
print(matrix[0:2])
# [[1, 2, 3], [4, 5, 6]]  -> first two rows
```

### Slicing within a row

Since `matrix[i]` is a list, it can be sliced too:

```python
print(matrix[0][0:2])  # [1, 2]  -> first two values of the first row
```

### Calculating sums with `enumerate()`

`enumerate()` becomes especially useful in multi-dimensional lists because you often need to know *which* row or column you're looking at while you total values — for example, to report a per-row sum, not just a single grand total.

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

# Sum of each row, with the row's index
for i, row in enumerate(matrix):
    print(f"Row {i} sum: {sum(row)}")

# Row 0 sum: 6
# Row 1 sum: 15
# Row 2 sum: 24
```

The same idea works for a running total across the whole grid, while still tracking position:

```python
total = 0
for i, row in enumerate(matrix):
    for j, value in enumerate(row):
        total += value
        print(f"matrix[{i}][{j}] = {value}, running total = {total}")

print("Grand total:", total)  # 45
```

Without `enumerate()`, you'd have no clean way to report *where* each value came from while traversing — you'd be reduced to manually tracking a counter variable yourself.

---

## 8. Adding New Elements in Multi-List

### Adding a new row

`.append()` on the outer list adds an entirely new row to the end:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6]
]

matrix.append([7, 8, 9])
print(matrix)
# [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
```

### Inserting a row at a specific position

`.insert()` works the same way it does for a normal list — the first argument is the position, the second is the new row:

```python
matrix.insert(1, [10, 11, 12])
print(matrix)
# [[1, 2, 3], [10, 11, 12], [4, 5, 6], [7, 8, 9]]
```

### Adding a new element to an existing row

Since each row is its own list, you `.append()` directly onto it by indexing into the outer list first:

```python
matrix[0].append(99)
print(matrix[0])  # [1, 2, 3, 99]
```

### Adding a new column to every row

There's no single method for this — since a "column" isn't a real structure in Python, you add to it by looping through every row and appending one new value to each:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

new_column = [100, 200, 300]

for row, value in zip(matrix, new_column):
    row.append(value)

print(matrix)
# [[1, 2, 3, 100], [4, 5, 6, 200], [7, 8, 9, 300]]
```

---

## 9. Replacing Elements in Multi-List

### Replacing an entire row

Assign a new list directly to the outer index:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

matrix[1] = [40, 50, 60]
print(matrix)
# [[1, 2, 3], [40, 50, 60], [7, 8, 9]]
```

### Replacing a single element

Use both indices — row, then position within the row:

```python
matrix[0][2] = 999
print(matrix[0])  # [1, 2, 999]
```

### Replacing a slice within a row

Just like a flat list, a slice on an inner row can be replaced with another sequence:

```python
matrix[2][0:2] = [70, 80]
print(matrix[2])  # [70, 80, 9]
```

### Replacing an entire column

As with adding a column, there's no shortcut — loop through the rows and overwrite the same position in each:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

for row in matrix:
    row[1] = 0

print(matrix)
# [[1, 0, 3], [4, 0, 6], [7, 0, 9]]
```

---

Multi-dimensional lists don't introduce a single new rule — they're the same indexing, slicing, and looping you already know, applied once per level of nesting. The real skill is recognizing *which shape* fits the data you're working with: a flat list for a single sequence, a 2D list for a grid, a list of dictionaries for a set of named records, a dict of dicts for records you need to look up by key. Once that shape is chosen correctly, everything else in this guideline follows from it directly.
