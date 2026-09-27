# Guideline 14: Python Dictionaries

Every collection covered so far organizes data around **position** (lists, tuples) or around **uniqueness alone** (sets). None of them let you look something up by a meaningful label — you cannot ask a list "what is Alice's score?" without first knowing which index Alice happens to occupy. A **dictionary** solves exactly this problem: it stores data as **pairs**, so that one value can be looked up directly by another, rather than by position.

---

## 14.1 What is a Dictionary? Keys and Values

A **dictionary** is an unordered-by-design (though, as you'll see in 14.2, actually ordered-in-practice) collection of **key-value pairs**, written with curly braces `{ }`, where each **key** is separated from its **value** by a colon `:`.

```python
student = {"name": "Alice", "age": 25, "major": "Computer Science"}
```

Here, `"name"`, `"age"`, and `"major"` are **keys** — the labels used to look something up — and `"Alice"`, `25`, and `"Computer Science"` are their corresponding **values** — the data actually being stored. A dictionary is best understood through the analogy in its name: a real dictionary maps a *word* (the key) to its *definition* (the value); you never scan through every definition looking for the one you want — you go directly to the word.

Retrieving a value works the same way, using square brackets with the **key** instead of a numeric index:

```python
print(student["name"])
# Alice

print(student["age"])
# 25
```

This is the central difference from every collection in Guidelines 11 through 13: `student[0]` means nothing here — there is no position `0` to speak of. `student["name"]` means everything, because `"name"` is the actual label attached to the value you want.

---

## 14.2 Properties of Dictionaries

Three properties define what is — and is not — allowed inside a dictionary, and each one connects directly back to something already covered in earlier guidelines.

### Keys Must Be Immutable

A key must be a type that cannot change after creation — strings, numbers, and tuples are all valid keys; lists and sets are not, for the exact same reason a list or set could not be an element of a set in Guideline 13 (both rely on the same underlying hashing mechanism).

```python
valid = {("x", "y"): "coordinate label"}   # a tuple key works fine

invalid = {["x", "y"]: "coordinate label"}
# TypeError: unhashable type: 'list'
```

### Keys Cannot Be Duplicated

Just as a set cannot contain the same value twice, a dictionary cannot contain the same key twice. If a key is written more than once in a dictionary literal, the **last** value assigned to it silently wins — earlier ones are simply overwritten, with no error:

```python
settings = {"volume": 50, "brightness": 80, "volume": 100}
print(settings)
# {'volume': 100, 'brightness': 80}
```

The first `"volume": 50` is gone entirely — this is a common, quiet source of bugs when a dictionary is built from data that unintentionally contains a repeated key, so it is worth watching for.

### Values Can Be Any Data Type — Including Mutable Ones

Unlike keys, values face no such restriction. A value can be a string, a number, a boolean, or even another list, dictionary, or any object at all:

```python
company = {
    "name": "Startorch Academy",
    "departments": ["Engineering", "Design", "Research"],
    "address": {"city": "Solaris City", "zip": "00100"}
}
```

Here, `"departments"` maps to a **list**, and `"address"` maps to **another dictionary**. This nesting is entirely normal and is how real-world structured data — a JSON API response, a configuration file — is almost always represented in Python.

### A Note on Order

Sets (Guideline 13) make no order guarantee at all. Dictionaries, since Python 3.7, are different: a dictionary **preserves the order elements were inserted in**, and iterating over it will always visit keys in that same order. This is a genuine, language-guaranteed behavior — not a coincidence of one particular run — and it is one more way a dictionary sits between a list (ordered, but positional) and a set (unique, but unordered): a dictionary is both **unique by key** and **ordered by insertion**.

---

## 14.3 Creating Dictionaries

There is more than one way to build a dictionary, and each fits a different situation.

### With `{}` or `dict()` — Distinguishing from `set()`

The most direct way is the literal syntax already shown above, or the equivalent `dict()` function using keyword arguments:

```python
student = {"name": "Alice", "age": 25}

student2 = dict(name="Alice", age=25)
print(student == student2)
# True
```

Recall from Guideline 13 that `{}` on its own creates an **empty dictionary**, not an empty set — `set()` is required for that. The colon is what makes a curly-brace collection a dictionary rather than a set: `{1, 2, 3}` is a set of three values; `{1: "one", 2: "two"}` is a dictionary of two key-value pairs. Visually similar, semantically completely different collections.

```python
empty_dict = {}
print(type(empty_dict))
# <class 'dict'>

empty_set = set()
print(type(empty_set))
# <class 'set'>
```

### From Two Lists Using `zip()`

`zip()`, introduced in Guideline 8 as a way to loop over two lists together, has a second, very natural use: pairing up a list of keys with a list of values to build a dictionary in one line.

```python
keys = ["name", "age", "major"]
values = ["Alice", 25, "Computer Science"]

student = dict(zip(keys, values))
print(student)
# {'name': 'Alice', 'age': 25, 'major': 'Computer Science'}
```

`zip(keys, values)` pairs each key with the value at the same position, and `dict()` turns that sequence of pairs directly into key-value entries. This is the standard approach whenever data naturally arrives as two separate, parallel lists — for instance, column headers and a row of data read from a file.

### From a Set of Keys Using `.fromkeys()`

`dict.fromkeys()` builds a dictionary from any iterable — commonly a set of unique labels — assigning the **same** default value to every one of them.

```python
positions = {"striker", "goalkeeper", "midfielder"}
lineup = dict.fromkeys(positions, None)
print(lineup)
# {'striker': None, 'goalkeeper': None, 'midfielder': None}
```

This is a convenient way to initialize a dictionary's structure — every key you will need, each starting from the same blank value — before filling in the real values afterward. Passing a set specifically (rather than a list) is a natural fit here, since a set already guarantees the keys you are about to create are unique, which a dictionary requires anyway.

### From `enumerate()` — Index-Value Pairs from a List

`enumerate()`, also introduced in Guideline 8 as a way to loop with both an index and a value at once, can build a dictionary directly, mapping each position in a list to its value:

```python
fruits = ["apple", "banana", "cherry"]
indexed_fruits = dict(enumerate(fruits))
print(indexed_fruits)
# {0: 'apple', 1: 'banana', 2: 'cherry'}
```

`enumerate(fruits)` produces `(0, "apple")`, `(1, "banana")`, `(2, "cherry")` — and `dict()` treats each of those pairs exactly the way it treated the `zip()` pairs above: first element as key, second as value. This is a genuinely useful technique whenever a list's original **position** needs to be preserved and looked up later, even after the data has been reorganized in ways that would otherwise lose track of where each value came from.

---

## 14.4 Methods in Dictionaries

### `.keys()`, `.values()`, and `.items()` — for Iteration

These three methods are the standard way to loop over a dictionary, and each gives you a different piece of it:

```python
student = {"name": "Alice", "age": 25, "major": "Computer Science"}

for key in student.keys():
    print(key)
# name
# age
# major

for value in student.values():
    print(value)
# Alice
# 25
# Computer Science

for key, value in student.items():
    print(key, "->", value)
# name -> Alice
# age -> 25
# major -> Computer Science
```

`.items()` is by far the most commonly used of the three in practice, because most real tasks need both the key and the value together — it unpacks each pair directly into two loop variables, the same way `enumerate()` unpacks an index and a value. Note also that simply looping over a dictionary directly (`for key in student:`) behaves identically to looping over `.keys()` — `.keys()` is there mainly for clarity and for the set-like operations mentioned in Guideline 13's closing note about dictionary keys behaving like a set of unique labels.

### `.get()` — Safe Lookup

Looking up a missing key with square brackets raises an error:

```python
student = {"name": "Alice", "age": 25}
print(student["email"])
# KeyError: 'email'
```

`.get()` looks up a key the same way, but returns `None` instead of crashing if the key does not exist — and optionally, a specific fallback value of your choosing:

```python
print(student.get("email"))
# None

print(student.get("email", "Not provided"))
# Not provided
```

`.get()` is the safer default whenever a key's presence cannot be guaranteed in advance, which in real data is most of the time.

### `.fromkeys()` — Revisited

As shown in Section 14.3, `.fromkeys(iterable, value)` builds a new dictionary from any iterable of keys, all sharing one starting value. It is listed here again because it is, technically, a dictionary **method** (called on the `dict` type itself, `dict.fromkeys(...)`, rather than on an existing dictionary instance) — one of the few methods in Python called this way.

### `.pop()` — Remove a Key and Return Its Value

`.pop(key)` removes a key-value pair and returns the value that was removed — directly parallel to `list.pop()` from Guideline 11, except it removes by **key** instead of by index.

```python
student = {"name": "Alice", "age": 25, "major": "Computer Science"}
removed_value = student.pop("age")
print(removed_value)   # 25
print(student)           # {'name': 'Alice', 'major': 'Computer Science'}
```

An optional second argument provides a fallback instead of raising an error if the key is missing — the same safety pattern as `.get()`:

```python
print(student.pop("email", "No such key"))
# No such key
```

### `.popitem()` — Remove the Most Recently Added Pair

`.popitem()` removes and returns an entire key-value pair, as a tuple — with no key specified at all. Since Python 3.7's guaranteed insertion order (Section 14.2), `.popitem()` specifically removes the **last** pair that was added, making it the dictionary equivalent of `list.pop()` with no argument, rather than the arbitrary removal of `set.pop()` from Guideline 13.

```python
student = {"name": "Alice", "age": 25, "major": "Computer Science"}
last_pair = student.popitem()
print(last_pair)   # ('major', 'Computer Science')
print(student)      # {'name': 'Alice', 'age': 25}
```

### `.setdefault()` — Get a Value, Inserting It if Missing

`.setdefault(key, value)` looks up a key exactly like `.get()` — but if the key does not already exist, it also **inserts** it into the dictionary with the given value, in the same step.

```python
student = {"name": "Alice", "age": 25}

major = student.setdefault("major", "Undeclared")
print(major)        # Undeclared
print(student)       # {'name': 'Alice', 'age': 25, 'major': 'Undeclared'}
```

If the key already exists, `.setdefault()` does not touch it — it simply returns the existing value, exactly like `.get()` would:

```python
name = student.setdefault("name", "Unknown")
print(name)          # Alice  (unchanged — "name" already existed)
```

### `.update()` — Merge Another Dictionary In

`.update()` merges the key-value pairs of another dictionary into the current one, directly paralleling `set.update()` from Guideline 13. Matching keys are overwritten with the new values; new keys are simply added.

```python
student = {"name": "Alice", "age": 25}
student.update({"age": 26, "major": "Computer Science"})
print(student)
# {'name': 'Alice', 'age': 26, 'major': 'Computer Science'}
```

`"age"` was overwritten from `25` to `26`; `"major"` was added fresh — both happened in a single call.

### `len()` and `sum()` on Dictionary Views

`len()` on a dictionary directly returns the number of key-value pairs — equivalent to, but more direct than, calling it on `.keys()` or `.values()`, since all three are always the same length:

```python
student = {"name": "Alice", "age": 25, "major": "Computer Science"}
print(len(student))          # 3
print(len(student.keys()))    # 3
print(len(student.values()))  # 3
```

`sum()` does not work directly on a dictionary itself (a dictionary's default iteration produces keys, and summing a mix of strings would fail) — but it works perfectly well on `.values()` specifically, whenever those values are numeric:

```python
scores = {"quiz1": 88, "quiz2": 92, "quiz3": 75}
print(sum(scores.values()))
# 255
```

---

## 14.5 Retrieving Keys Based on Values, and Vice Versa

Looking up a **value** from a **key** is the entire point of a dictionary, and it is a single, direct operation: `student["name"]` or `student.get("name")`. Looking up a **key** from a **value** is the reverse question, and it is meaningfully harder — because, unlike keys, **values are allowed to repeat**, so there may be more than one correct answer, or none at all.

### Finding One Key for a Given Value

The direct approach loops through `.items()` and checks each value, stopping at the first match:

```python
grades = {"Alice": "A", "Ben": "B", "Cody": "A"}

def find_key_by_value(dictionary, target_value):
    for key, value in dictionary.items():
        if value == target_value:
            return key
    return None

print(find_key_by_value(grades, "B"))
# Ben
```

Note that this returns only the **first** matching key it finds — for `grades`, both `"Alice"` and `"Cody"` map to `"A"`, but only `"Alice"` would be returned by this function, since it stops at the first match and never continues searching.

### Finding All Keys for a Given Value

If every match matters, a list comprehension (Guideline 11) collects all of them instead of stopping at one:

```python
grades = {"Alice": "A", "Ben": "B", "Cody": "A"}

a_students = [key for key, value in grades.items() if value == "A"]
print(a_students)
# ['Alice', 'Cody']
```

### Building a Reverse Lookup Dictionary

If reverse lookups (value → key) will be needed **repeatedly**, it is far more efficient to build the reversed dictionary once, up front, than to loop through `.items()` every single time a lookup is needed:

```python
grades = {"Alice": "A", "Ben": "B", "Cody": "A"}
reversed_grades = {value: key for key, value in grades.items()}
print(reversed_grades)
# {'A': 'Cody', 'B': 'Ben'}
```

This uses a **dictionary comprehension** — the same expression-based idea from Guideline 11's list comprehensions, adapted to produce key-value pairs instead of single values. Look closely at the result, though: `reversed_grades["A"]` is `"Cody"`, not `"Alice"` — because `"Alice"` and `"Cody"` originally shared the same value `"A"`, and a dictionary cannot have two identical keys (Section 14.2), so the later one silently overwrote the earlier one during the reversal, exactly the way the duplicate-key literal did back in Section 14.2.

This is the fundamental asymmetry between the two directions of lookup, and it is worth carrying forward as a rule of thumb: **a reverse lookup dictionary is only fully reliable when the original values are themselves unique.** When they are not, a list comprehension collecting *every* matching key — as shown above — is the honest, complete answer; a reversed dictionary will quietly discard the very duplicates that made the reverse lookup interesting in the first place.

*End of Guideline 14.*
